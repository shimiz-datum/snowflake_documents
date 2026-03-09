# [PIPE_USAGE_HISTORY view](https://docs.snowflake.com/en/sql-reference/account-usage/pipe_usage_history)

# 1. ドキュメント概要（初学者向け）

このドキュメントは、Snowflake の **ACCOUNT_USAGE スキーマ**にある **`PIPE_USAGE_HISTORY` ビュー**について説明しています。  
`PIPE_USAGE_HISTORY` を使うと、次の履歴を確認できます。

- **Snowpipe によるデータロード履歴**
- **（Snowpipe または Iceberg の自動更新などにより）請求されたクレジット使用履歴**

つまり「いつ・どのパイプで・どれくらいのデータがロードされ・どれくらいクレジットが請求されたか」を追跡するためのビューです。

## Snowflake全体の中での位置づけ
Snowflake には、利用状況やコストを把握するための **Usageビュー（Account Usage / Organization Usage）** が用意されています。  
`PIPE_USAGE_HISTORY` はその中でも **Snowpipe（連続データロード）** や関連する処理における **利用量・課金状況の監査/可視化** に使うビューです。 :contentReference[oaicite:0]{index=0}

【補足】  
- **Snowpipe**：ステージに到着したファイルを継続的（マイクロバッチ）にテーブルへ取り込む仕組み  
- **ACCOUNT_USAGE**：アカウント全体の履歴・利用状況を参照するための共有ビュー群（遅延がある場合があります）

---

# 2. 原文に沿った内容整理（翻訳ベース）

# PIPE_USAGE_HISTORY view

## 概要
この Account Usage ビューは、次の履歴をクエリするために使用できます。 :contentReference[oaicite:1]{index=1}

- **Snowpipe を使用してテーブルにロードされたデータの履歴**
- **過去 365 日（1 年）以内の Iceberg 自動更新（automated refresh）で使用されたクレジットの履歴**

このビューには、**ロードされたデータの履歴**と、**Snowflake アカウント全体で請求されたクレジット**が表示されます。 :contentReference[oaicite:2]{index=2}

また、`PIPE_NAME` 列を使用して特定のパイプ（または Iceberg テーブル）に絞り込めます。 :contentReference[oaicite:3]{index=3}

【補足】  
「Account Usage ビュー」は、主に運用・監査・課金分析のために利用されます。

---

## 列（Columns）
`PIPE_USAGE_HISTORY` ビューには、以下の列があります。 :contentReference[oaicite:4]{index=4}

### PIPE_ID
- **データロードに使用されるパイプ**の内部（システム生成）識別子。 :contentReference[oaicite:5]{index=5}
- クエリでパイプ名が指定されなかった場合、`NULL` を表示します。 :contentReference[oaicite:6]{index=6}
- 各行には、時間範囲内で使用中のすべてのパイプの合計が含まれます。 :contentReference[oaicite:7]{index=7}

### PIPE_NAME
- 自動更新を行う **パイプ**または **Iceberg テーブル**の名前。 :contentReference[oaicite:8]{index=8}
- 外部テーブルまたは Delta ベースの Iceberg テーブルのメタデータを更新するために使用される内部（非表示）パイプオブジェクトは `NULL` を表示します。 :contentReference[oaicite:9]{index=9}

### START_TIME
- データ取り込み情報が集計される期間の開始時刻。 :contentReference[oaicite:10]{index=10}

### END_TIME
- データ取り込み情報が集計される期間の終了時刻。 :contentReference[oaicite:11]{index=11}

### CREDITS_USED
- `START_TIME`〜`END_TIME` のウィンドウ内で、Snowpipe データロードに対して **請求されたクレジット数**。 :contentReference[oaicite:12]{index=12}

### BYTES_INSERTED
- `START_TIME`〜`END_TIME` のウィンドウ内で **ロードされたバイト数**。 :contentReference[oaicite:13]{index=13}

### FILES_INSERTED
- `START_TIME`〜`END_TIME` のウィンドウ内で **ロードされたファイル数**。 :contentReference[oaicite:14]{index=14}

### BYTES_BILLED
- Snowpipe が **請求目的で使用したバイト数**を表します。  
  この履歴ビュー内で Snowpipe のコストへの影響を直接可視化できます。 :contentReference[oaicite:15]{index=15}

【補足】  
`BYTES_INSERTED`（実際に入った量）と `BYTES_BILLED`（課金対象として扱われる量）は、用途が違う点に注意します。

---

## 使用上の注意（Usage notes）
このビューに関して、ドキュメントでは以下が明記されています。 :contentReference[oaicite:16]{index=16}

### ビューの待機時間（Latency）
- このビューの待機時間は最大 **180 分（3 時間）** です。 :contentReference[oaicite:17]{index=17}

【補足】  
リアルタイムの秒単位監視ではなく、「数時間遅れて集計が反映される可能性がある」ビューです。

---

### Organization Usage との整合（タイムゾーン）
- このビューのデータを `ORGANIZATION USAGE` スキーマの対応するビューと一致させるには、  
  まずセッションのタイムゾーンを **UTC** に設定する必要があります。 :contentReference[oaicite:18]{index=18}
- Account Usage ビューをクエリする前に、次を実行します。

```sql
ALTER SESSION SET TIMEZONE = UTC;
```

---

### データ圧縮とメンテナンスによるクレジット消費
- データ圧縮とメンテナンスの過程で、Snowflake クレジットが消費されることがあります。 :contentReference[oaicite:19]{index=19}
- 例えば結果が **`0 BYTES_INSERTED` と `0 FILES_INSERTED`** でクレジット消費を示す場合があります。 :contentReference[oaicite:20]{index=20}
- これは **データはロードされていない**ものの、**圧縮とメンテナンスのプロセスでクレジットが消費されている**ことを意味します。 :contentReference[oaicite:21]{index=21}

---

### 外部テーブル / ディレクトリテーブルの自動更新通知の課金
- Snowflake は、内部名前付きステージおよび外部ステージの **外部テーブル**および **ディレクトリテーブル**の自動更新通知に対して、  
  **Snowpipe ファイル料金と同等のレート**で請求します。 :contentReference[oaicite:22]{index=22}
- `PIPE_USAGE_HISTORY` ビュー（または `PIPE_USAGE_HISTORY` 関数）をクエリすると、  
  **外部テーブルとディレクトリテーブルの自動更新通知**によって発生する料金を見積もることができます。 :contentReference[oaicite:23]{index=23}
- 自動更新パイプは **`NULL` パイプ名**の下にリストされることに注意してください。 :contentReference[oaicite:24]{index=24}

【補足】  
外部テーブル等の自動更新は「パイプとして見えるが名前が `NULL` になるケースがある」ため、フィルタ時に見落としやすい点です。

---

### 外部テーブルの通知履歴（より細かい粒度）
- `INFORMATION_SCHEMA` のテーブル関数 `AUTO_REFRESH_REGISTRATION_HISTORY` を使用して、  
  **テーブルレベル / ステージレベル**の粒度で外部テーブルの自動更新通知履歴を表示することもできます。 :contentReference[oaicite:25]{index=25}

---

### 自動更新通知の課金を回避するには
- 自動更新通知の課金を回避するには、外部テーブルとディレクトリテーブルの **手動更新**を実行します。 :contentReference[oaicite:26]{index=26}
- 外部テーブルの場合は、次のように実行します（ドキュメントでは `ALTER EXTERNAL TABLE ... REFRESH` が示されています）。 :contentReference[oaicite:27]{index=27}

```sql
ALTER EXTERNAL TABLE <name> REFRESH ... ;
```

（※ `...` の部分は原文の記載に従ってください）

---

# 3. 注意点・制約（明記がある場合）

ドキュメントに明記されている注意点・制約は次のとおりです。 :contentReference[oaicite:28]{index=28}

- 対象期間は **過去 365 日（1 年）**（Snowpipe データロード履歴、または Iceberg 自動更新のクレジット履歴）。 :contentReference[oaicite:29]{index=29}
- このビューには最大 **180 分（3 時間）** の待機時間（遅延）がある。 :contentReference[oaicite:30]{index=30}
- `ORGANIZATION USAGE` と突合する場合は、セッションのタイムゾーンを **UTC** に設定する必要がある。 :contentReference[oaicite:31]{index=31}
- `BYTES_INSERTED=0` / `FILES_INSERTED=0` でも、圧縮やメンテナンスにより **クレジット消費が発生する**ことがある。 :contentReference[oaicite:32]{index=32}
- 外部テーブルやディレクトリテーブルの **自動更新通知は Snowpipe ファイル料金と同等レートで課金**される。 :contentReference[oaicite:33]{index=33}
- 外部テーブル/ディレクトリテーブルの自動更新パイプは **`PIPE_NAME` が `NULL`** として表示される場合がある。 :contentReference[oaicite:34]{index=34}
- 自動更新通知の課金を回避するには **手動更新**を実行する（外部テーブルでは `ALTER EXTERNAL TABLE ... REFRESH`）。 :contentReference[oaicite:35]{index=35}
