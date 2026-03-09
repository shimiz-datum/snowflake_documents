# [Introduction to unstructured data](https://docs.snowflake.com/en/user-guide/unstructured-intro)

# 1. ドキュメント概要（初学者向け）

このドキュメントは、Snowflake における **非構造化データ（Unstructured data）** を扱うための「導入（Overview）」です。  
非構造化データとは、表形式のような **事前定義されたスキーマに当てはまらない情報**で、画像・動画・音声・PDF・ドキュメント・ログなどが該当します。 :contentReference[oaicite:0]{index=0}

Snowflake は、非構造化データを **ステージ（Stage）** に保存し、次のような操作をできるようにします。 :contentReference[oaicite:1]{index=1}

- クラウドストレージ上のファイルへ **安全にアクセス**
- 共同作業者やパートナーと **ファイルアクセスURLを共有**
- ファイルアクセスURLやファイルメタデータを **Snowflake テーブルへロード**
- 非構造化データを **処理（Process）**

## Snowflake全体の中での位置づけ
Snowflake は通常、テーブル形式のデータを扱う基盤として使われますが、非構造化データ機能により **「ファイルをSnowflakeの管理・分析フローに乗せる」**ことができます。  
特に、ファイルの安全な共有や、ファイルメタデータをテーブル化して分析する、といった用途の入り口になります。 :contentReference[oaicite:2]{index=2}

【補足】  
- **ステージ（Stage）**：Snowflake がファイルを置いたり参照したりする場所（内部ステージ／外部ステージ）  
- **非構造化データ**：行・列の表に整形されていないファイル形式のデータ

---

# 2. 原文に沿った内容整理（翻訳ベース）

# 非構造化データの概要（Unstructured data）

## 非構造化データとは
非構造化データは、事前定義されたデータモデルまたはスキーマに適合しない情報です。 :contentReference[oaicite:3]{index=3}  
通常、フォームの応答やソーシャルメディアの会話など、テキストが多い非構造化データには、画像、ビデオ、オーディオも含まれます。 :contentReference[oaicite:4]{index=4}  
また、このカテゴリには、次のような **業界固有のファイル型** も含まれます。 :contentReference[oaicite:5]{index=5}

- VCF（ゲノミクス）
- KDF（半導体）
- HDF5（航空学）

【補足】  
「CSVのような表データ」ではなく、「ファイルのまま扱うデータ」を指しています。

---

## Snowflake がサポートするアクション（できること）
Snowflake は、次のアクションをサポートしています。 :contentReference[oaicite:6]{index=6}

- クラウドストレージにあるデータファイルへの **安全なアクセス**
- 共同編集者およびパートナーとの **ファイルアクセス URLs の共有**
- ファイルアクセス URLs およびその他ファイルメタデータの **Snowflake テーブルへのロード**
- 非構造化データを **処理**

さらに、Document AI を使って非構造化データをロードするトピックも案内されています。 :contentReference[oaicite:7]{index=7}

【補足】  
「ファイルを置いて終わり」ではなく、URL化・共有・メタデータのテーブル化・処理までの流れが意識されています。

---

## このトピックの内容（目次）
このページでは、次の項目が扱われます。 :contentReference[oaicite:8]{index=8}

- クラウドストレージサービスのサポート
- ファイルにアクセスできる URLs の型
- 非構造化データアクセスのサーバー側の暗号化
- ディレクトリテーブル
- SQL 関数
- Snowsightでステージングされたファイルのダウンロード
  - 生成されたスコープ、署名済み、またはファイル URL のダウンロード
  - 内部ステージからのダウンロード
- 非構造化データの処理
- FILE データ型

---

## クラウドストレージサービスのサポート
Snowflake は、外部（外部クラウドストレージ）ステージと内部（つまり Snowflake）ステージの両方で非構造化データをサポートします。 :contentReference[oaicite:9]{index=9}

### 外部ステージ（External stage）
外部クラウドストレージへのファイル保存として、以下が挙げられています。 :contentReference[oaicite:10]{index=10}

- Amazon S3
- Google Cloud Storage
- サポートされている Microsoft Azure クラウドストレージサービスのいずれか：
  - BLOB ストレージ
  - Data Lake Storage Gen2
  - 汎用 v1
  - 汎用 v2

【補足】  
外部ステージは「Snowflakeの外にあるクラウドストレージを参照する設定」です。

---

## ファイルにアクセスできる URLs の型
クラウドストレージ内のファイルにアクセスするには、次の型の URLs を使用できます。 :contentReference[oaicite:11]{index=11}

- スコープ URL（Scoped URL）
- ファイル URL（File URL）
- 事前署名付き URL（Presigned URL）

以下、原文の表で示されている特性を、見出しごとに翻訳して整理します。

---

### スコープ URL（Scoped URL）
- ステージに権限を付与せずに、ステージングされたファイルへの **一時的なアクセス** を許可するエンコードされた URL。 :contentReference[oaicite:12]{index=12}
- URL は **永続クエリ結果期間** が終了すると（結果キャッシュが期限切れになると）期限切れになります。  
  これは現在 **24時間** です。 :contentReference[oaicite:13]{index=13}

**ユースケース** :contentReference[oaicite:14]{index=14}  
ファイル管理者が同じアカウント内の特定ロールに対してデータファイルへのスコープアクセスを許可することを推奨します。  
スコープ付き URLs を取得するビューを使用してファイルアクセスを提供し、ビューに対する権限を持つロールのみがファイルにアクセスできます。

Snowflake は、誰がいつスコープ付き URL を使用してファイルにアクセスしたのかを、クエリ履歴に情報として記録します。 :contentReference[oaicite:15]{index=15}  
また、次の用途に最適です。 :contentReference[oaicite:16]{index=16}

- カスタムアプリケーションでの使用
- 共有を介した他のアカウントへの非構造化データ提供
- Snowsight での非構造化データのダウンロードとアドホック分析

**生成方法**：`BUILD_SCOPED_FILE_URL` 関数をクエリします。 :contentReference[oaicite:17]{index=17}

**使用状況（アクセス方法）** :contentReference[oaicite:18]{index=18}  
次のオプションを使用できます：

- Snowsight のクエリ結果テーブルにあるスコープ URL をクリックする  
  - Snowsight は、スコープ URL を生成したユーザーに対してのみファイルを取得
- GET リクエストでスコープ URL を **ファイルサポート REST API エンドポイント** に送信する  
  - 詳細は「非構造化データサポート用の REST API」を参照

**認証** :contentReference[oaicite:19]{index=19}  
スコープ付き URL を生成したユーザーのみが、その URL を使って参照ファイルにアクセスできます。

【補足】  
スコープURLは「一時的」「発行ユーザーに紐づく」性質があり、アクセス範囲を狭く制御したいときに向きます。

---

### ファイル URL（File URL）
- ステージ上のファイルに対する **永続的 URL**。 :contentReference[oaicite:20]{index=20}
- ファイルをダウンロードまたはアクセスするには、ユーザーは GET リクエストでファイル URL を認証トークンとともに REST API エンドポイントに送信します。 :contentReference[oaicite:21]{index=21}
- 非構造化データファイルへのアクセスを必要とするカスタムアプリケーションに最適です。 :contentReference[oaicite:22]{index=22}

**生成方法** :contentReference[oaicite:23]{index=23}  
次のいずれかです：

- ステージングされたファイルを参照するステージのディレクトリテーブルをクエリする
- `BUILD_STAGE_FILE_URL` 関数を呼び出す

**使用状況（アクセス方法）** :contentReference[oaicite:24]{index=24}  
次のオプションを使用できます：

- Snowsight でクエリ結果テーブルのファイル URL をクリックする  
  - Snowsight は、アクティブなロールに十分な権限がある場合にのみファイルを取得
- GET リクエストでファイル URL をファイルサポート REST API エンドポイントに送信する

**認証** :contentReference[oaicite:25]{index=25}  
GET REST API 呼び出しで指定されたロールには、ステージに対する十分な権限が必要です。

- 外部ステージ：`USAGE`
- 内部ステージ：`READ`

【補足】  
ファイルURLは「永続」ですが、アクセスする側がステージ権限を持っている必要があります。

---

### 事前署名付き URL（Presigned URL）
- ウェブブラウザーを介してファイルにアクセスするために使用される単純な HTTPS URL。 :contentReference[oaicite:26]{index=26}
- ユーザーは、事前署名付きのアクセストークンを使用し、この URL 経由でファイルに一時的にアクセスできます。 :contentReference[oaicite:27]{index=27}
- アクセストークンの有効期限は構成可能です。 :contentReference[oaicite:28]{index=28}

**ユースケース** :contentReference[oaicite:29]{index=29}  
<span style= "color: red; ">Snowflake への認証や認証トークンの受け渡しなしで、ファイルをダウンロードまたはアクセス</span>するために使用されます。  
事前署名付き URLs は開いており、すべてのユーザーまたはアプリケーションはファイルに直接アクセスまたはダウンロードできます。  
非構造化ファイルの内容を表示する必要がある BI アプリケーションまたはレポートツールに最適です。

**生成方法**： **<span style= "color: red; ">`GET_PRESIGNED_URL` 関数をクエリ</span>** します。 :contentReference[oaicite:30]{index=30}

**使用状況（アクセス方法）** :contentReference[oaicite:31]{index=31}  
次のオプションを使用できます：

- Snowsight のクエリ結果テーブルの事前署名付き URL をクリックする
- ウェブブラウザーで直接、事前署名付き URL に移動する

**認証** :contentReference[oaicite:32]{index=32}  
事前署名付き URL を持っている人は誰でも、トークンの存続期間中、参照されたファイルにアクセスできます。

**有効期限** :contentReference[oaicite:33]{index=33}  
`expiration_time` 引数で指定された時間の長さ。

【補足】  
事前署名付きURLは「URLを知っていればアクセスできる」ため、共有時の取り扱いに注意が必要です。

---

### URL の型ごとの Secure Data Sharing 可否
Snowflake Secure Data Sharing において、データコンシューマーがプロバイダーのセキュアビュー経由でアクセスできるかどうかが整理されています。 :contentReference[oaicite:34]{index=34}

- **スコープ URL**：アクセスできます  
- **ファイル URL**：アクセスできません  
- **事前署名付き URL**：アクセスできます

【補足】  
データ共有（Share）経由でファイルアクセスを提供したい場合、URLの型によって可能/不可能が分かれます。

---

## 非構造化データアクセスのサーバー側の暗号化
内部ステージで非構造化データにアクセスできるようにするために、ステージ作成時に **サーバー側の暗号化** を使用することを検討できます。 :contentReference[oaicite:35]{index=35}  
そうしない場合、ステージングされたファイルはデフォルトで **クライアント側で暗号化** されます。 :contentReference[oaicite:36]{index=36}

暗号化キーは Snowflake が所有し、クライアント側で暗号化されたファイルは、事前署名付き／ファイル／スコープ付き URLs を使用するユーザーや外部ツールによって読み取ることはできません。 :contentReference[oaicite:37]{index=37}

内部ステージにサーバー側の暗号化を構成するには、`CREATE STAGE` コマンドで `SNOWFLAKE_SSE` 暗号化タイプを指定します。 :contentReference[oaicite:38]{index=38}  
例として、サーバー側暗号化とディレクトリテーブル付きで内部ステージを作成する SQL が示されています：

```sql
CREATE STAGE my_int_stage
  ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE')
  DIRECTORY = ( ENABLE = true );
```
:contentReference[oaicite:39]{index=39}

### 重要（Important）
セキュリティコンプライアンスのために Tri-Secret Secure が必要な場合は、内部ステージに `SNOWFLAKE_FULL` 暗号化タイプを使用します。  
`SNOWFLAKE_SSE` は Tri-Secret Secure をサポートしていません。 :contentReference[oaicite:40]{index=40}

### 注釈（Note）
- ステージ作成後に、内部ステージの暗号化タイプを変更することはできません。 :contentReference[oaicite:41]{index=41}
- 現在、サーバー側の暗号化を使用した内部ステージの作成は、**JDBC ドライバー v3.12.11（またはそれ以上）** の Snowflake クライアントバージョンに限定されています。 :contentReference[oaicite:42]{index=42}

【補足】  
内部ステージの暗号化方式は「後から変更できない」ため、設計段階で決める必要があります。

---

## ディレクトリテーブル（Directory table）
ディレクトリテーブルは、ステージングされたファイルのカタログをクラウドストレージに保存します。 :contentReference[oaicite:43]{index=43}  
十分な権限を持つロールは、ディレクトリテーブルにクエリを実行して、ファイル URLs を取得し、ステージングされたファイルにアクセスできます。 :contentReference[oaicite:44]{index=44}

【補足】  
ファイル一覧やパス情報を SQL で扱えるようになるため、「ファイルの棚卸し」や「URL生成の起点」に使えます。

---

## SQL 関数（ファイル関数）
データファイルにアクセスするために、次の **ファイル関数** が用意されています。 :contentReference[oaicite:45]{index=45}

| SQL 関数 | 説明 |
|---|---|
| GET_STAGE_LOCATION | ステージ名を入力として使用して、外部または内部の名前付きステージ URL を返します。 |
| GET_RELATIVE_PATH | ステージ名と絶対ファイルパスを入力として使用して、ステージ内の場所を基準にしたファイルパスを抽出します。 |
| GET_ABSOLUTE_PATH | ステージ名と相対ファイルパスを入力として使用して、ファイルの絶対パスを返します。 |
| GET_PRESIGNED_URL | ステージ名と相対ファイルパスを入力として使用して、ファイルに事前署名付き URL を生成します（外部ステージのファイルにアクセス）。 |
| BUILD_SCOPED_FILE_URL | ステージ名と相対ファイルパスを入力として使用して、ファイルにスコープ付き Snowflake ファイル URL を生成します。 |
| BUILD_STAGE_FILE_URL | ステージ名と相対ファイルパスを入力として使用して、ファイルに Snowflake ファイル URL を生成します。 |
| TO_FILE | 内部または外部ステージに保存されたファイルを表す FILE オブジェクトを返します。 |
| TRY_TO_FILE | TO_FILE と同様に FILE オブジェクトを返しますが、ファイルが存在しないかアクセスできない場合は NULL を返します。 |

:contentReference[oaicite:46]{index=46}

【補足】  
URL生成系（GET_PRESIGNED_URL / BUILD_*）と、FILE 型に変換する系（TO_FILE / TRY_TO_FILE）が分かれています。

---

## Snowsightでステージングされたファイルのダウンロード

### 生成されたスコープ、署名済み、またはファイル URL のダウンロード
ユーザーは Snowsight ワークシートの結果テーブルで、生成されたスコープ／事前署名／ファイル URL を選択して、参照ファイルをダウンロードできます。 :contentReference[oaicite:47]{index=47}

手順： :contentReference[oaicite:48]{index=48}
1. Snowsight にサインインします。
2. ナビゲーションメニューで Worksheets を開く（My Worksheets、Recent、Folders など）。
3. サポートされているメソッドのいずれかを使用して、スコープ／事前署名／ファイル URL をクエリで返します。
4. 結果テーブルにある URL を選択します。
5. Snowsight が URL の参照ファイルをダウンロードします。

---

### 内部ステージからのダウンロード
ユーザーは内部ステージのファイルを Snowsight から直接ダウンロードできます。 :contentReference[oaicite:49]{index=49}

手順： :contentReference[oaicite:50]{index=50}
1. Snowsight にサインインします。
2. 内部ステージにあるファイルに移動します。
3. Download を選択します。

---

## 非構造化データの処理
Snowflake は、非構造化データの処理を支援するために次の機能をサポートしています。 :contentReference[oaicite:51]{index=51}

### 外部関数（External functions）
外部関数は、Snowflake 外で格納および実行されるユーザー定義関数です。 :contentReference[oaicite:52]{index=52}  
外部関数により、Amazon Textract、Document AI、Azure Computer Vision といった、内部のユーザー定義関数（UDFs）からはアクセスできないライブラリを使用できます。 :contentReference[oaicite:53]{index=53}

---

### ユーザー定義関数（UDF）およびストアドプロシージャ
Snowflake は Java または Python コード内でファイルを読み取る複数の方法をサポートしているため、次の中で非構造化データを処理できます。 :contentReference[oaicite:54]{index=54}

- ユーザー定義関数（UDFs）
- ユーザー定義テーブル関数（UDTFs）
- ストアドプロシージャ

さらに Snowflake で使用している SQL を拡張したり、Snowpark API を使用してアプリケーションを構築したりできます。 :contentReference[oaicite:55]{index=55}  
詳細と例は、関連トピックとして次が示されています： :contentReference[oaicite:56]{index=56}

- UDF およびプロシージャハンドラーを使用した非構造化データの処理
- Python UDF ハンドラーを使用したファイルの読み取り
- Python Stored Procedure Handler を使用したファイルの読み取り
- Java 用 Snowpark API を使用したファイルの読み取り
- Python 用 Snowpark API を使用したファイルの読み取り

【補足】  
SQLだけで完結しない処理（画像解析など）を、UDF/外部関数などでつなげる前提の説明です。

---

## FILE データ型
FILE データタイプは、内部または外部ステージに保存されたファイルを表します。 :contentReference[oaicite:57]{index=57}  
組み込み関数の中には、ステージ名とファイルパスの代わりに FILE オブジェクトを受け付けるものがあります。 :contentReference[oaicite:58]{index=58}

`TO_FILE` または `TRY_TO_FILE` 関数を使用して、ステージングされたファイルの場所を FILE オブジェクトに変換します。 :contentReference[oaicite:59]{index=59}

【補足】  
「ファイルを指す値」を SQL 上で扱えるようにするデータ型です。

---

# 3. 注意点・制約（明記がある場合）

ドキュメントに明記されている注意点・制約は次のとおりです。 :contentReference[oaicite:60]{index=60}

- スコープ URL は **永続クエリ結果期間（現在24時間）** で期限切れになる。 :contentReference[oaicite:61]{index=61}
- 内部ステージでサーバー側暗号化を使わない場合、ファイルはデフォルトで **クライアント側暗号化** され、外部ツール等から URL 経由で読み取れない。 :contentReference[oaicite:62]{index=62}
- Tri-Secret Secure が必要な場合は `SNOWFLAKE_FULL` を使用し、`SNOWFLAKE_SSE` は Tri-Secret Secure をサポートしない。 :contentReference[oaicite:63]{index=63}
- ステージ作成後に、内部ステージの暗号化タイプを **変更できない**。 :contentReference[oaicite:64]{index=64}
- 現在、サーバー側暗号化を使用した内部ステージの作成は **JDBC ドライバー v3.12.11 以上**のクライアントバージョンに限定されている。 :contentReference[oaicite:65]{index=65}
- Secure Data Sharing では URL の型によりアクセス可否が異なる（ファイル URL はコンシューマーがアクセス不可）。 :contentReference[oaicite:66]{index=66}
```
::contentReference[oaicite:67]{index=67}
