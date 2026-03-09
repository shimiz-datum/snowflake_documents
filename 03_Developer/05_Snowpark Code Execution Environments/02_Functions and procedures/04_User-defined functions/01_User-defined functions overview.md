# [User-defined functions overview](https://docs.snowflake.com/developer-guide/udf/udf-overview#scalar-and-tabular-functions)

# 1. ドキュメント概要（初学者向け）

このドキュメントは、Snowflake における **ユーザー定義関数（UDF: User-Defined Function）** の概要を説明しています。  
UDF を使うと、Snowflake が標準で提供している組み込み関数だけでは実現できない処理を、自分で関数として追加できます。作成した UDF は何度でも再利用できます。

## Snowflake全体の中での位置づけ
Snowflake には SQL だけでは難しい処理を拡張する手段が複数あります。UDF はそのうちの 1 つです。

- **UDF（ユーザー定義関数）**：SQL から呼び出せる自作関数
- **ストアドプロシージャ（Stored Procedure）**：処理をまとめて実行する仕組み
- **外部関数（External Function）**：Snowflake 外部のサービスを呼び出す仕組み
- **Snowpark API**：Python/Java/Scala などでデータ処理を書く仕組み

**Note（原文の注記）**  
UDF はストアドプロシージャに似ていますが、重要な違いがあります。どちらを使うべきかは別ページで説明されています。

---

# 2. 原文に沿った内容整理（翻訳ベース）

# User-defined functions overview（ユーザー定義関数の概要）

Snowflake が提供する **システム定義（組み込み）関数** では利用できない処理を実現するために、**ユーザー定義関数（UDF）** を作成してシステムを拡張できます。  
一度 UDF を作成すると、複数回再利用できます。

UDF は Snowflake の拡張方法の 1 つにすぎません。他の方法については次を参照してください：

- Stored procedures overview（ストアドプロシージャ概要）
- Writing external functions（外部関数の作成）
- Snowpark API

**Note（原文の注記）**  
UDF はストアドプロシージャに似ていますが、両者には重要な違いがあります。詳細は「ストアドプロシージャと UDF のどちらを書くべきか」を参照してください。

---

## What is a user-defined function (UDF)?（UDF とは？）

**ユーザー定義関数（UDF）** とは、SQL から呼び出せるようにユーザーが定義する関数です。  
SQL から呼び出せる **組み込み関数** と同様に、UDF のロジックは SQL に足りない、または SQL が得意ではない機能を補う/拡張する目的で使われます。

また UDF を利用すると、ある処理をひとまとまりの機能として **カプセル化** できるため、複数箇所から繰り返し呼び出せます。

UDF のロジック（処理本体）は、サポートされているいずれかの言語で **ハンドラ（handler）** として実装します。  
ハンドラを用意したら、次の手順で利用します：

1. `CREATE FUNCTION` コマンドで **UDF を作成**
2. `SELECT` 文で **UDF を呼び出し**

【補足】  
ここでいう **ハンドラ（handler）** は「関数の中身（処理ロジック）」を指します。SQL 側では `CREATE FUNCTION` でそのハンドラを参照します。

---

### Scalar and tabular functions（スカラー関数と表形式関数）

UDF は、戻り値の形式によって次の 2 種類があります。

- **スカラー関数（scalar UDF）**  
  単一の値を返す UDF です。

- **表形式関数（UDTF: User-Defined Table Function）**  
  表（テーブル）のような形式を返す関数です。

それぞれの動作は次のとおりです：

- **スカラー関数（UDF）**  
  入力の各行に対して **出力は 1 行** になります。  
  返される行は **1 列（1 つの値）** で構成されます。

- **表形式関数（UDTF）**  
  入力の各行に対して **表形式の値（複数列・複数行になり得る）** を返します。  
  UDTF のハンドラでは、Snowflake が要求するインターフェースに従うメソッドを実装します。これらのメソッドは次を行います：
  - パーティション内の各行を処理する（必須）
  - パーティションごとに 1 回ハンドラを初期化する（任意）
  - パーティションごとに処理を終了させる（任意）

これらメソッドの名前は、ハンドラ言語によって異なります。  
サポートされる言語の一覧は **Supported languages** を参照してください。

【補足】  
ここでいう **パーティション（partition）** は、UDTF が処理対象の行を一定のまとまりとして扱う単位のことです（詳細な意味合いは UDTF の仕様に従います）。

---

### Considerations（考慮事項）

- クエリが UDF を呼び出して **ステージ上のファイル** にアクセスする場合、  
  その SQL 文が、（その UDF/UDTF がステージファイルにアクセスするかどうかに関係なく）**何らかの UDF または UDTF を呼び出すビュー** も同時にクエリしていると、操作は **ユーザーエラーとして失敗** します。

- UDTF は複数ファイルを **並列** に処理できますが、UDF は現時点ではファイルを **直列** に処理します。  
  回避策として、サブクエリで `GROUP BY` 句を使って行をグルーピングします。例は「Process a CSV with a UDTF」を参照してください。

- 現在、クエリ実行中にクエリが参照しているステージファイルが **変更または削除** されると、関数呼び出しは **エラーで失敗** します。

- UDF のハンドラコード内で `CURRENT_DATABASE` または `CURRENT_SCHEMA` 関数を指定した場合、  
  その関数が返すのは **セッションで使用中のデータベース/スキーマではなく、UDF が格納されているデータベース/スキーマ** です。

【補足】  
ステージ（stage）は Snowflake 内でファイルを置く領域です。UDF/UDTF がステージファイルを扱う場合は、クエリ構成や実行中のファイル変更に注意が必要です。

---

## Get started（はじめに）

SQL で書かれたハンドラを使って UDTF を作成するチュートリアルとして、次を参照してください：

- Quickstart: Getting Started With User-Defined SQL Functions

---

## UDF example（UDF の例）

次の例では、Python で書かれたハンドラを使って `addone` という UDF を作成します。  
ハンドラ関数は `addone_py` で、この UDF は `int` を返します。

```sql
CREATE OR REPLACE FUNCTION addone(i int)
RETURNS INT
LANGUAGE PYTHON
RUNTIME_VERSION = '3.8'
HANDLER = 'addone_py'
as
$$
def addone_py(i):
  return i+1
$$;
```

次の例では `addone` UDF を実行します。

```sql
SELECT addone(3);
```

---

## Supported languages（サポートされている言語）

関数のハンドラ（ロジック）は、複数のプログラミング言語で記述できます。  
各言語では、言語とそのランタイム環境の制約の中でデータを操作できます。

ただし、ハンドラ言語に関係なく、関数自体は SQL で同じように作成します。  
つまり、SQL でハンドラと言語を指定して作成します。

ハンドラを記述できる言語は次のとおりです：

- Java
  - Java UDFs
  - Java UDTFs
- JavaScript
  - JavaScript UDFs
  - JavaScript UDTFs
- Python
  - Python UDFs
  - Python UDTFs
- Scala
  - Scala UDFs
- SQL
  - SQL UDFs
  - SQL UDTFs

---

### Language choice（言語選択）

UDF のハンドラ（ロジック）は、複数の言語から選んで書くことができます。  
各言語には、それぞれのランタイム環境の範囲内でデータ操作できる特徴があります。

次のような場合、特定の言語を選ぶことがあります：

- すでにその言語のコード資産がある場合  
  例：Java でハンドラとして使えるメソッドがすでにあり、そのオブジェクトが `.jar` ファイルに入っている場合、  
  `.jar` をステージにコピーし、ハンドラをクラスとメソッドとして指定し、言語を Java に指定できます。

- 他の言語にはない機能をその言語が持つ場合

- 必要な処理に役立つライブラリがその言語にある場合

また、言語を選ぶ際には次の点も考慮します：

- **対応しているハンドラ配置（handler location）**  
  すべての言語が「ステージ上のハンドラ参照」に対応しているわけではありません。  
  言語によっては、ハンドラコードを **インライン（CREATE FUNCTION 文の中）** に書く必要があります。  
  詳細は「Keeping handler code in-line or on a stage」を参照してください。

- **共有可能な UDF になるかどうか（sharable）**  
  共有可能な UDF は、Snowflake の **Secure Data Sharing（セキュアデータ共有）** 機能で利用できます。

言語ごとの特性（ハンドラ配置 / 共有可否）は次のとおりです：

| 言語 | Handler Location | Sharable |
|---|---|---|
| Java | In-line または staged | No |
| JavaScript | In-line | Yes |
| Python | In-line または staged | No |
| Scala | In-line または staged | No |
| SQL | In-line | Yes |

注記（原文脚注）：
- Java UDF の共有制限については「General limitations」を参照してください。
- Python UDF の共有制限については「General limitations」を参照してください。
- Scala UDF の共有制限については「Scala UDF limitations」を参照してください。

【補足】  
ここでいう **Sharable（共有可能）** は、Snowflake のセキュアデータ共有機能と一緒に利用できるか、という観点です。

---

## Developer guides（開発ガイド）

### Guidelines and constraints（ガイドラインと制約）

**Snowflake の制約（constraints）**  
Snowflake 環境内で安定性を確保するために、Snowflake の制約の中で開発できます。  
詳細は「Designing Handlers that Stay Within Snowflake-Imposed Constraints」を参照してください。

**命名（Naming）**  
他の関数との衝突を避けるように関数名を付けてください。  
詳細は「Naming and overloading procedures and UDFs」を参照してください。

**引数（Arguments）**  
引数を指定し、どの引数が任意（optional）かを示します。  
詳細は「Defining arguments for UDFs and stored procedures」を参照してください。

**データ型マッピング（Data type mappings）**  
ハンドラ言語ごとに、その言語のデータ型と SQL の引数/戻り値型の対応表（マッピング）が別々に定義されています。  
詳細は「Data Type Mappings Between SQL and Handler Languages」を参照してください。

---

### Handler writing（ハンドラの記述）

**ハンドラ言語（Handler languages）**  
言語ごとのハンドラ記述内容については Supported languages を参照してください。

**外部ネットワークアクセス（External network access）**  
`external network access` を使うことで、外部ネットワーク上の場所にアクセスできます。  
Snowflake 外部の特定ネットワークロケーションへの安全なアクセスを作成し、そのアクセスをハンドラコードから利用できます。

**ロギングとトレース（Logging and tracing）**  
ログメッセージやトレースイベントを記録し、そのデータを後からクエリできるデータベースに保存できます。

---

### Security（セキュリティ）

UDF または UDTF によって特定の SQL 操作を実行させるために必要なオブジェクトに対して、権限（privileges）を付与できます。  
詳細は「Granting privileges for user-defined functions」を参照してください。

関数はストアドプロシージャと共通するセキュリティ上の注意点があります。詳細は次を参照してください：

- 「Security Practices for UDFs and Procedures」に記載されたベストプラクティスに従うことで、ハンドラコードを安全に実行できます。
- 「Protecting Sensitive Information with Secure UDFs and Stored Procedures」を参照し、アクセス権のないユーザーから機密情報が見えないようにします。

---

### Handler code deployment（ハンドラコードの配置・デプロイ）

関数を作成する際、関数ロジックを実装するハンドラは次の形式で指定できます：

- `CREATE FUNCTION` 文の中に **インラインでコードを書く**
- 文の外部にあるコードを指定する（例：コンパイル済みコードをパッケージ化してステージにコピーしたもの）

詳細は「Keeping handler code in-line or on a stage」を参照してください。

---

## Create and call functions（関数の作成と呼び出し）

ユーザー定義関数は **SQL** を使って作成・呼び出しします。

- 関数を作成するには、`CREATE FUNCTION` 文を実行し、関数のハンドラを指定します。  
  詳細は「Creating a UDF」を参照してください。

- 関数を呼び出すには、関数を引数として指定する SQL の `SELECT` 文を実行します。  
  詳細は「Calling a UDF」を参照してください。

---

# 3. 注意点・制約（明記がある場合）

このページで明記されている注意点・制約は以下のとおりです：

- ステージファイルにアクセスする UDF を呼び出すクエリは、同時に **（ステージアクセス有無に関係なく）UDF/UDTF を呼び出すビュー** をクエリしている場合、ユーザーエラーとして失敗する。
- UDTF は複数ファイルを並列処理できるが、UDF は現時点でファイルを直列処理する。
- クエリ実行中に参照しているステージファイルが変更/削除されると、関数呼び出しはエラーで失敗する。
- ハンドラ内で `CURRENT_DATABASE` / `CURRENT_SCHEMA` を使うと、セッションの DB/スキーマではなく **UDF が格納された DB/スキーマ** が返る。
- 言語によって **ハンドラ配置（インライン/ステージ）** や **共有可能（Sharable）かどうか** が異なる。
