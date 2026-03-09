# [JDBC Driver](https://docs.snowflake.com/en/developer-guide/jdbc/jdbc)

# 1. ドキュメント概要（初学者向け）

このドキュメントは、Snowflake に接続するための **JDBC ドライバー（JDBC Driver）** について説明しています。  
Snowflake は **Type 4 JDBC ドライバー**（Java から直接ネットワーク経由で DB に接続するタイプ）を提供しており、Java アプリケーションや JDBC 対応ツールから Snowflake に接続して SQL を実行できます。:contentReference[oaicite:0]{index=0}

## Snowflake全体の中での位置づけ
Snowflake は、さまざまなクライアント（BIツール、ETL、アプリケーション）から接続して利用されます。  
JDBC ドライバーは、その中でも **Java の標準的な接続方式（JDBC）** を使って Snowflake を利用するための基盤コンポーネントであり、JDBC 対応の多くのツールやアプリケーションで利用できます。:contentReference[oaicite:1]{index=1}

【補足】  
Snowflake が提供していた JDBC ベースのコマンドラインクライアント **sfsql** は、現在 **非推奨（deprecated）** になっています。:contentReference[oaicite:2]{index=2}

---

# 2. 原文に沿った内容整理（翻訳ベース）

# JDBC Driver（JDBC ドライバー）

## 概要（Overview）
Snowflake は、コア JDBC 機能をサポートする **JDBC Type 4 ドライバー**を提供しています。:contentReference[oaicite:3]{index=3}  
JDBC ドライバーは **64-bit 環境にインストール**する必要があり、**Java の LTS（長期サポート）バージョン 1.8 以上**が必要です。:contentReference[oaicite:4]{index=4}

このドライバーは、データベースサーバーへの接続に JDBC をサポートする多くのクライアントツール/アプリケーションで使用できます。:contentReference[oaicite:5]{index=5}  
例として Snowflake が提供していた **sfsql** は JDBC ベースのアプリケーションですが、現在は非推奨です。:contentReference[oaicite:6]{index=6}

【補足】  
「Type 4」は、JDBCドライバー自身がネットワーク越しに直接 Snowflake と通信するタイプを指します（中間層が不要）。

---

## JDBC ドライバーの入手（Downloading / integrating the JDBC Driver）
Snowflake JDBC ドライバー（`snowflake-jdbc`）は **JAR ファイル**として提供されており、次の方法で利用できます。:contentReference[oaicite:7]{index=7}

- Maven の artifact として **ダウンロード**する
- Java プロジェクトへ **直接組み込む（依存関係として追加）** する :contentReference[oaicite:8]{index=8}

また、Maven からは以下のタイプのドライバーもダウンロードできます。:contentReference[oaicite:9]{index=9}

- **FIPS 準拠**の fat jar：`snowflake-jdbc-fips`
- **thin jar**：`snowflake-jdbc-thin`

【補足】  
- **fat jar** は依存関係を含めた形の配布を指すことがあります（利用形態に応じて選択します）。
- **FIPS 準拠**は、特定のセキュリティ要件に沿った暗号運用が必要なケースで選ばれます。

---

## JDBC ドライバーの構成（Configuring the JDBC Driver）
このトピックでは、JDBC ドライバーを構成する方法（Snowflake に接続する方法を含む）を説明します。:contentReference[oaicite:10]{index=10}  
接続パラメーターは **JDBC Driver connection parameter reference** に移動して整理されています。:contentReference[oaicite:11]{index=11}

### JDBC ドライバークラス
JDBC アプリケーションでは、ドライバークラスとして次を使用します。:contentReference[oaicite:12]{index=12}

- `net.snowflake.client.jdbc.SnowflakeDriver`

【補足】  
Snowflake の Java コードから JDBC で接続する場合、「どの Driver class を使うか」は重要な設定項目です。

---

## JDBC ドライバーの利用（Using the JDBC Driver）
Snowflake JDBC ドライバーは、Snowflake 固有のメソッドをサポートしています。:contentReference[oaicite:13]{index=13}  
これらのメソッドは、Snowflake 固有の Java インターフェースで定義されています。:contentReference[oaicite:14]{index=14}

例（原文にあるもの）：

- `SnowflakeConnection`
- `SnowflakeStatement`
- `SnowflakeResultSet` :contentReference[oaicite:15]{index=15}

例えば `SnowflakeStatement` には、JDBC の標準 `Statement` インターフェースには存在しない `getQueryID()` メソッドが含まれます。:contentReference[oaicite:16]{index=16}

【補足】  
Snowflake 固有機能（例：クエリID取得など）を使いたい場合は、標準 JDBC の範囲を超えた API を利用できます。

---

# 3. 注意点・制約（明記がある場合）

ドキュメント上で明記されている注意点・制約は次のとおりです。:contentReference[oaicite:17]{index=17}

- Snowflake JDBC ドライバーは **JDBC Type 4** ドライバーである。:contentReference[oaicite:18]{index=18}
- JDBC ドライバーは **64-bit 環境**で使用する必要がある。:contentReference[oaicite:19]{index=19}
- Java は **LTS バージョン 1.8 以上**が必要。:contentReference[oaicite:20]{index=20}
- JDBC ドライバー（`snowflake-jdbc`）は **JAR ファイル**で提供され、Maven artifact として取得できる。:contentReference[oaicite:21]{index=21}
- FIPS 準拠 fat jar（`snowflake-jdbc-fips`）や thin jar（`snowflake-jdbc-thin`）も利用可能。:contentReference[oaicite:22]{index=22}
- Snowflake JDBC ドライバーには Snowflake 固有の拡張インターフェース／メソッドがある（例：`getQueryID()`）。:contentReference[oaicite:23]{index=23}
````markdown
::contentReference[oaicite:24]{index=24}
