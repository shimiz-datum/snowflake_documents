# [COPY_HISTORY view](https://docs.snowflake.com/en/sql-reference/account-usage/copy_history)

# 1. ドキュメント概要（初学者向け）

このドキュメントは、Snowflake の **ACCOUNT_USAGE スキーマ**にある **`COPY_HISTORY` ビュー**について説明しています。  
`COPY_HISTORY` を使うと、過去 1 年間（365 日）の **データロード履歴**を確認できます。

具体的には、以下のロードが対象です：

- `COPY INTO <table>` による **一括データロード（Bulk load）**
- Snowpipe による **連続データロード（Continuous load）**

ロードしたファイル名、ステージの場所、ロード結果（成功/失敗/スキップなど）、行数、エラー情報などを確認でき、データロードの監査やトラブルシュートに役立ちます。

## Snowflake全体の中での位置づけ
Snowflake には、利用状況や処理履歴を確認するための **Account Usage ビュー（SNOWFLAKE.ACCOUNT_USAGE）** が用意されています。  
`COPY_HISTORY` はその中でも、「データロード（COPY/Snowpipe）」に関する履歴を長期間（365 日）追跡するためのビューです。 :contentReference[oaicite:0]{index=0}

【補足】  
- **ACCOUNT_USAGE**：アカウント全体の履歴や利用状況を参照するビュー群（反映に遅延がある場合があります）
- **Snowpipe**：ステージ上の新しいファイルを継続的に取り込む仕組み

---

# 2. 原文に沿った内容整理（翻訳ベース）

# COPY_HISTORY view

## 概要
この Account Usage ビューを使用して、過去 **365 日（1 年）** の Snowflake データロード履歴をクエリできます。 :contentReference[oaicite:1]{index=1}

このビューには、次の両方のロードアクティビティが表示されます： :contentReference[oaicite:2]{index=2}

- `COPY INTO <table>` ステートメントによるロード
- Snowpipe を使用した連続データロード

またこのビューは、`LOAD_HISTORY` ビューにある **10,000 行の制限を回避**します。 :contentReference[oaicite:3]{index=3}

Snowsight でもデータロードの詳細を確認できます（関連トピック参照）。 :contentReference[oaicite:4]{index=4}

【補足】  
大量のロード履歴を確認したい場合に `COPY_HISTORY` が役立ちます。

---

## 列（Columns）
`COPY_HISTORY` ビューには以下の列があります。 :contentReference[oaicite:5]{index=5}

### FILE_NAME
- ソースファイルの名前と、ファイルへの相対パス。 :contentReference[oaicite:6]{index=6}

### STAGE_LOCATION
- ソースファイルが配置されているステージの名前。 :contentReference[oaicite:7]{index=7}

【補足】  
**ステージ（stage）** は、クラウドストレージや Snowflake 管理領域など、ロード元ファイルを置く場所です。

### LAST_LOAD_TIME
- ファイルのロードが終了した日時。 :contentReference[oaicite:8]{index=8}

### ROW_COUNT
- ソースファイルからロードされた行の数。 :contentReference[oaicite:9]{index=9}

### ROW_PARSED
- ソースファイルから解析された行数。  
  `STATUS` が `Load in progress` の場合は `NULL`。 :contentReference[oaicite:10]{index=10}

### FILE_SIZE
- ロードされたソースファイルのサイズ。 :contentReference[oaicite:11]{index=11}

### FIRST_ERROR_MESSAGE
- ソースファイルの最初のエラー。 :contentReference[oaicite:12]{index=12}

### FIRST_ERROR_LINE_NUMBER
- 最初のエラーの行番号。 :contentReference[oaicite:13]{index=13}

### FIRST_ERROR_CHARACTER_POS
- 最初のエラー文字の位置。 :contentReference[oaicite:14]{index=14}

### FIRST_ERROR_COLUMN_NAME
- 最初のエラーの列名。 :contentReference[oaicite:15]{index=15}

### ERROR_COUNT
- ソースファイル内のエラー行の数。 :contentReference[oaicite:16]{index=16}

### ERROR_LIMIT
- エラーの数がこの制限に達した場合、中止します。 :contentReference[oaicite:17]{index=17}

### STATUS
- ステータス：`Loaded`、`Load failed`、`Partially loaded`、または `Load skipped`。 :contentReference[oaicite:18]{index=18}

### TABLE_ID
- ターゲットテーブルの内部/システム生成識別子。 :contentReference[oaicite:19]{index=19}

### TABLE_NAME
- ターゲットテーブルの名前。 :contentReference[oaicite:20]{index=20}

### TABLE_SCHEMA_ID
- Snowflake が生成する、テーブルのスキーマの内部識別子。 :contentReference[oaicite:21]{index=21}

### TABLE_SCHEMA_NAME
- ターゲットテーブルが存在するスキーマの名前。 :contentReference[oaicite:22]{index=22}

### TABLE_CATALOG_ID
- テーブルのデータベースの内部/システム生成識別子。 :contentReference[oaicite:23]{index=23}

### TABLE_CATALOG_NAME
- ターゲットテーブルが存在するデータベースの名前。 :contentReference[oaicite:24]{index=24}

### PIPE_CATALOG_NAME
- パイプが存在するデータベースの名前。 :contentReference[oaicite:25]{index=25}

### PIPE_SCHEMA_NAME
- パイプが存在するスキーマの名前。 :contentReference[oaicite:26]{index=26}

### PIPE_NAME
- ロードパラメーターを定義するパイプの名前。  
  `COPY` ステートメントのロードの場合は `NULL`。 :contentReference[oaicite:27]{index=27}

【補足】  
**パイプ（pipe）** は Snowpipe が参照するオブジェクトで、内部的には「どこからどこへロードするか」を定義した `COPY` 文を持ちます。

### PIPE_RECEIVED_TIME
- パイプ経由でロードされたファイルの INSERT リクエストが受信された日時。  
  `COPY` ステートメントロードの場合は `NULL`。 :contentReference[oaicite:28]{index=28}

### FIRST_COMMIT_TIME
- ファイルの最初のチャンクがコミットされた日時。  
  Snowpipe は、個別にコミットされた複数のチャンクでファイルをロードする場合があります。 :contentReference[oaicite:29]{index=29}

### BYTES_BILLED
- Snowpipe が請求目的で使用したバイト数を表し、これらの履歴ビュー内で直接 Snowpipe のコストへの影響を可視化します。 :contentReference[oaicite:30]{index=30}

---

## 使用上の注意（Usage notes）

### ビューの遅延（Latency）
多くの場合、ビューの遅延は最大 **120 分（2 時間）** です。 :contentReference[oaicite:31]{index=31}

ただし、あるテーブルのコピー履歴の遅延は、次の条件が **両方** 真である場合、最大 **2 日** になることがあります。 :contentReference[oaicite:32]{index=32}

- `COPY_HISTORY` で最後に更新されてから、特定のテーブルに追加されたステートメントが **32 DML 未満**
- `COPY_HISTORY` で最後に更新されてから、特定のテーブルに追加された行が **100 行未満**

【補足】  
履歴ビューは反映が遅れる場合があるため、「直近のロードがすぐ見えない」可能性があります。

---

### 含まれるロードの条件
このビューには、エラーの有無にかかわらず **完了するまで実行された `COPY INTO` コマンドのみ** が含まれます。 :contentReference[oaicite:33]{index=33}

---

### テーブルの削除・再作成の影響
テーブルオブジェクトを **削除または再作成** すると、テーブルへの一括データロード（`COPY INTO <table>` ステートメント）の  
**重複排除のためのロード履歴メタデータが削除**されます。 :contentReference[oaicite:34]{index=34}

【補足】  
ロード履歴は「同じファイルの二重取り込みを防ぐ」用途にも使われるため、テーブルの作り直しで履歴が消える点に注意します。

---

### テーブル名変更の影響
テーブルオブジェクトの名前を変更すると、コピー履歴の対応する `TABLE_NAME` エントリが更新されます。 :contentReference[oaicite:35]{index=35}

---

### パイプの削除・再作成の影響
パイプオブジェクトを削除または再作成しても、パイプのロード履歴メタデータは削除されません。 :contentReference[oaicite:36]{index=36}

---

### 現在のロールによる表示範囲
このビューには、セッションの **現在のロールにアクセス権限が付与されているオブジェクトのみ** が表示されます。 :contentReference[oaicite:37]{index=37}

【補足】  
権限がないオブジェクトの履歴は見えないため、監査・分析の際はロールに注意します。

---

### 複製（Replication）後の履歴表示
コピー履歴の複製後、`COPY_HISTORY` Account Usage ビューには、ターゲットテーブルの **最新の切り捨て（TRUNCATE）操作後** にのみ履歴が表示されます。 :contentReference[oaicite:38]{index=38}  
これは、完全なコピー履歴を表示する **複製なしのビュー** とは異なります。 :contentReference[oaicite:39]{index=39}

---

## 例（Examples）

### 実行された最新の `COPY INTO` コマンド 10 件を取得
```sql
select file_name, error_count, status, last_load_time
from snowflake.account_usage.copy_history
order by last_load_time desc
limit 10;
```
:contentReference[oaicite:40]{index=40}

---

# 3. 注意点・制約（明記がある場合）

ドキュメントに明記されている注意点・制約は次のとおりです。 :contentReference[oaicite:41]{index=41}

- `COPY_HISTORY` ビューで参照できるのは **過去 365 日（1 年）** のロード履歴。 :contentReference[oaicite:42]{index=42}
- このビューは `COPY INTO <table>` と Snowpipe の両方のロードアクティビティを表示する。 :contentReference[oaicite:43]{index=43}
- `LOAD_HISTORY` ビューの **10,000 行制限を回避**できる。 :contentReference[oaicite:44]{index=44}
- 多くの場合の遅延は最大 **120 分** だが、条件によっては最大 **2 日** になることがある。 :contentReference[oaicite:45]{index=45}
- このビューに含まれるのは、エラーの有無にかかわらず **完了まで実行された `COPY INTO` のみ**。 :contentReference[oaicite:46]{index=46}
- テーブルの削除/再作成で、一括ロードの **重複排除のための履歴メタデータが削除**される。 :contentReference[oaicite:47]{index=47}
- パイプの削除/再作成では、パイプのロード履歴メタデータは削除されない。 :contentReference[oaicite:48]{index=48}
- 表示される履歴は、セッションの **現在ロールに権限があるオブジェクトに限られる**。 :contentReference[oaicite:49]{index=49}
- 複製後は、`COPY_HISTORY` に表示される履歴が **最新の TRUNCATE 後のみ** になる場合がある。 :contentReference[oaicite:50]{index=50}
