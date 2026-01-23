# サンプルデータ（スプレッドシート設定例）

GAS実装時にスプレッドシートへ入力するサンプルデータです。

---

## SETTINGSシート

| A列（項目名） | B列（値） |
|--------------|----------|
| Dashboard Settings | |
| | |
| 必須設定 | |
| ダッシュボード表示名 | LINEWORKS 事務所ダッシュボード |
| 社内ポータルURL | https://sites.google.com/your-domain/portal |
| 既定の開き方 | new_tab |
| APP_MASTER シート名 | APP_MASTER |
| データ開始行 | 4 |

---

## APP_MASTERシート

### ヘッダー（3行目）

```
enabled | sort | key | category | label | url | layout | icon_source | icon_value | open_mode | note | status
```

### サンプルデータ（4行目以降）

| enabled | sort | key | category | label | url | layout | icon_source | icon_value | open_mode | note | status |
|---------|------|-----|----------|-------|-----|--------|-------------|------------|-----------|------|--------|
| TRUE | 10 | meeting_room | 予約・設備 | 会議室予約 | https://example.com/meeting | small | drive_file_id | (DriveファイルID) | new_tab | 空き状況確認 | OK |
| TRUE | 20 | business_trip | 外出・出張 | 出張看板 | https://example.com/trip | small | drive_file_id | (DriveファイルID) | new_tab | 出張・外出状況 | OK |
| TRUE | 30 | vehicle | 予約・設備 | 車両予約 | https://example.com/vehicle | small | drive_file_id | (DriveファイルID) | new_tab | 社用車予約 | OK |
| TRUE | 40 | production_board | 生産・工程 | 生産工程表 | https://example.com/production | wide | drive_file_id | (DriveファイルID) | new_tab | 工程・進捗確認 | OK |
| TRUE | 50 | ceo_schedule | 役員・重要予定 | 社長予定 | https://example.com/ceo | tall | drive_file_id | (DriveファイルID) | new_tab | 社長スケジュール | OK |

---

## CATEGORIESシート

| sort | category | note |
|------|----------|------|
| 10 | 予約・設備 | 会議室/車両など |
| 20 | 外出・出張 | 出張看板/在席管理など |
| 30 | 生産・工程 | 工程表/進捗など |
| 40 | 役員・重要予定 | 社長予定など |
| 90 | その他 | |

---

## VALIDATIONシート（入力規則用リスト）

| 項目 | 選択肢 |
|------|--------|
| Enabled | TRUE, FALSE |
| Layout | small, wide, tall |
| OpenMode | new_tab, same_tab |
| IconSource | drive_file_id, image_url, none |

---

## 注意事項

1. **status列**はシート側で自動計算する想定
   - URL未入力 → `Missing URL`
   - label未入力 → `Missing label`
   - すべてOK → `OK`

2. **icon_value**にはGoogle DriveのファイルIDを入力
   - ファイルIDの取得方法：Driveで画像を右クリック → 共有 → リンクをコピー
   - URL内の `id=XXXXX` 部分がファイルID

3. **enabled=FALSE**にすると非表示（削除せずに一時的に隠せる）
