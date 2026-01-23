# GAS Code.gs 仕様書

そのままコピペで動く `Code.gs` 雛形。
`SETTINGS` と `APP_MASTER` を読み込み → HTMLテンプレに変数セット。

## 前提

- Excelで作った台帳を **Googleスプレッドシートに置き換え**、同じ列構成で使う
- 公開設定（推奨）:
  - 実行ユーザー：自分
  - アクセス権：ドメイン内全員

---

## Code.gs

```javascript
/**
 * 社内ダッシュボード - Code.gs 雛形
 * - SETTINGS / APP_MASTER を読み込み、Index.html にデータを渡して描画
 *
 * 期待シート:
 *  - SETTINGS: A列=項目名, B列=値（Excel雛形と同じ）
 *  - APP_MASTER: ヘッダー行=3, データ開始=4（Excel雛形と同じ）
 *
 * 公開設定（推奨）:
 *  - 実行ユーザー：自分
 *  - アクセス権：ドメイン内全員
 */

// ===== 設定 =====
const SHEET_SETTINGS = "SETTINGS";
const SHEET_APP_MASTER = "APP_MASTER";
const APP_HEADER_ROW = 3;
const APP_DATA_START_ROW = 4;

// SETTINGSのキー（A列）をこの文字列で探します（Excel雛形に合わせる）
const SETTINGS_KEYS = {
  DASHBOARD_TITLE: "ダッシュボード表示名",
  PORTAL_URL: "社内ポータルURL",
  DEFAULT_OPEN_MODE: "既定の開き方", // new_tab / same_tab
};

// icon_source
const ICON_SOURCE = {
  DRIVE_FILE_ID: "drive_file_id",
  IMAGE_URL: "image_url",
  NONE: "none",
};

// open_mode
const OPEN_MODE = {
  NEW_TAB: "new_tab",
  SAME_TAB: "same_tab",
};

/**
 * Webアプリ入口
 */
function doGet(e) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();

  // 1) SETTINGS取得
  const settings = loadSettings_(ss);

  // 2) APP_MASTER取得
  const apps = loadApps_(ss, settings.defaultOpenMode);

  // 3) 最終表示データ
  const viewModel = {
    portalUrl: settings.portalUrl,
    dashboardTitle: settings.dashboardTitle,
    lastUpdated: formatDateTime_(new Date()),
    apps: apps,
  };

  // 4) HTMLテンプレに渡す
  const tpl = HtmlService.createTemplateFromFile("Index"); // Index.html
  tpl.portalUrl = viewModel.portalUrl;
  tpl.dashboardTitle = viewModel.dashboardTitle;
  tpl.lastUpdated = viewModel.lastUpdated;
  tpl.apps = viewModel.apps;

  return tpl.evaluate()
    .setTitle(viewModel.dashboardTitle)
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL); // Googleサイト埋め込み想定
}

/**
 * SETTINGSシートを読み込み、A列=キー、B列=値 のマップとして返す
 */
function loadSettings_(ss) {
  const sh = ss.getSheetByName(SHEET_SETTINGS);
  if (!sh) throw new Error(`シートが見つかりません: ${SHEET_SETTINGS}`);

  // SETTINGSは上から数行に収まる想定。念のため広めに取得
  const range = sh.getRange(1, 1, Math.min(sh.getLastRow(), 200), 2).getValues();

  const map = {};
  range.forEach(row => {
    const key = (row[0] || "").toString().trim();
    const val = (row[1] || "").toString().trim();
    if (key) map[key] = val;
  });

  const dashboardTitle = map[SETTINGS_KEYS.DASHBOARD_TITLE] || "社内ダッシュボード";
  const portalUrl = map[SETTINGS_KEYS.PORTAL_URL] || "";
  const defaultOpenMode = map[SETTINGS_KEYS.DEFAULT_OPEN_MODE] || OPEN_MODE.NEW_TAB;

  return { dashboardTitle, portalUrl, defaultOpenMode, raw: map };
}

/**
 * APP_MASTERを読み込み、表示対象だけ返す
 * 条件: enabled=TRUE かつ status=OK
 */
function loadApps_(ss, defaultOpenMode) {
  const sh = ss.getSheetByName(SHEET_APP_MASTER);
  if (!sh) throw new Error(`シートが見つかりません: ${SHEET_APP_MASTER}`);

  const lastRow = sh.getLastRow();
  const lastCol = sh.getLastColumn();
  if (lastRow < APP_DATA_START_ROW) return [];

  // ヘッダー取得
  const headers = sh.getRange(APP_HEADER_ROW, 1, 1, lastCol).getValues()[0].map(h => (h || "").toString().trim());
  const col = indexColumns_(headers);

  // データ取得
  const values = sh.getRange(APP_DATA_START_ROW, 1, lastRow - APP_DATA_START_ROW + 1, lastCol).getValues();

  const apps = [];
  values.forEach((row) => {
    const enabled = normalizeBool_(row[col.enabled]);
    const status = (row[col.status] || "").toString().trim();

    if (!enabled) return;
    if (status !== "OK") return;

    const app = {
      sort: toNumber_(row[col.sort], 9999),
      key: (row[col.key] || "").toString().trim(),
      category: (row[col.category] || "").toString().trim(),
      label: (row[col.label] || "").toString().trim(),
      url: (row[col.url] || "").toString().trim(),
      layout: (row[col.layout] || "").toString().trim(), // small|wide|tall
      iconSource: (row[col.icon_source] || "").toString().trim(),
      iconValue: (row[col.icon_value] || "").toString().trim(),
      openMode: ((row[col.open_mode] || "").toString().trim() || defaultOpenMode),
      note: (row[col.note] || "").toString().trim(),
      iconUrl: "",
    };

    // アイコンURL生成
    app.iconUrl = buildIconUrl_(app.iconSource, app.iconValue);

    // 最低限のバリデーション（保険）
    if (!app.label || !app.url || !app.layout) return;

    apps.push(app);
  });

  // sort昇順
  apps.sort((a, b) => (a.sort - b.sort));
  return apps;
}

/**
 * ヘッダー配列から列インデックスを返す（0-based）
 */
function indexColumns_(headers) {
  const required = [
    "enabled", "sort", "key", "category", "label", "url", "layout",
    "icon_source", "icon_value", "open_mode", "note", "status"
  ];

  const idx = {};
  required.forEach(name => {
    const i = headers.indexOf(name);
    if (i === -1) throw new Error(`APP_MASTERのヘッダーに '${name}' が見つかりません`);
    idx[name] = i;
  });

  return idx;
}

/**
 * icon_source と icon_value からブラウザで表示できるURLを作る
 * - drive_file_id: 画像公開権限が必要。推奨はドメイン内閲覧可。
 * - image_url: そのまま使用
 */
function buildIconUrl_(iconSource, iconValue) {
  if (!iconSource || iconSource === ICON_SOURCE.NONE) return "";

  if (iconSource === ICON_SOURCE.IMAGE_URL) {
    return iconValue || "";
  }

  if (iconSource === ICON_SOURCE.DRIVE_FILE_ID) {
    if (!iconValue) return "";

    // DriveファイルID -> 画像表示URL（軽量）
    // ※ファイル共有が必要。表示できない場合は共有設定を見直す。
    return `https://drive.google.com/uc?export=view&id=${encodeURIComponent(iconValue)}`;
  }

  return "";
}

/**
 * TRUE/FALSE 等をbool化
 */
function normalizeBool_(v) {
  const s = (v || "").toString().trim().toLowerCase();
  return (s === "true" || s === "1" || s === "yes" || s === "y");
}

function toNumber_(v, fallback) {
  const n = Number(v);
  return Number.isFinite(n) ? n : fallback;
}

function formatDateTime_(d) {
  return Utilities.formatDate(d, Session.getScriptTimeZone(), "yyyy-MM-dd HH:mm");
}

/**
 * HTMLファイルを分割する場合に利用（任意）
 * <?!= include('style'); ?> のように呼べる
 */
function include(filename) {
  return HtmlService.createHtmlOutputFromFile(filename).getContent();
}
```

---

## 揃ったもの

- `Index.html`（HTML_TEMPLATE_SPEC.md に記載）
- `Code.gs`（本ファイル）

あとは **スプレッドシートのID** と **シート名が合っていること** を確認すれば動作する。

---

## 実装時の注意点（詰まりやすいところ）

### 1. Googleサイトに埋め込む場合

`setXFrameOptionsMode(ALLOWALL)` が必要（上記コードで設定済み）

### 2. アイコンが表示されない場合

- Drive画像の共有設定が足りないことが多い
- まずは `image_url` で直URLテストして切り分け推奨

### 3. APP_MASTER の status 列

- シート側で自動計算する前提
- GAS側でも最低限チェックして落ちないようにしている
