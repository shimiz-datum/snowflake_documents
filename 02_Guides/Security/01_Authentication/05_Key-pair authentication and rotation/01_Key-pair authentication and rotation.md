# [Key-pair authentication and rotation](https://docs.snowflake.com/en/user-guide/key-pair-auth)

# 1. ドキュメント概要（初学者向け）

このドキュメントは、Snowflake で **キーペア認証（Key-pair authentication）** を使ってログイン（接続）する方法と、運用上重要な **キーペアローテーション（鍵の入れ替え）** について説明しています。

キーペア認証は、ユーザー名＋パスワードのような「基本認証」の代わりに、**公開鍵・秘密鍵のペア**（暗号鍵）を使って認証する方式です。  
パスワードを送信・保存しない設計にできるため、より強固な認証を目指すときに使います。

## Snowflake全体の中での位置づけ
Snowflake は複数の認証方法（パスワード、SSO、OAuth、キーペア認証など）をサポートしています。  
その中で **キーペア認証** は、特に **クライアントアプリケーション / ドライバ / CLI** などから Snowflake に安全に接続したいケースで使われる方法です。 :contentReference[oaicite:0]{index=0}

【補足】  
- **公開鍵（public key）**：Snowflake のユーザーに登録する鍵  
- **秘密鍵（private key）**：クライアント（アプリ側）が安全に保管する鍵  
- 認証では「秘密鍵を持っていること」を証明し、Snowflake が登録済み公開鍵で検証します :contentReference[oaicite:1]{index=1}

---

# 2. 原文に沿った内容整理（翻訳ベース）

# Key-pair authentication and key-pair rotation（キーペア認証とキーペアローテーション）

このトピックでは、Snowflake で **キーペア認証** と **キーペアローテーション** を使用する方法を説明します。 :contentReference[oaicite:2]{index=2}

---

## Overview（概要）
Snowflake は、ユーザー名とパスワードなどの基本認証の代替として、認証セキュリティを強化するための **キーペア認証** の使用をサポートしています。 :contentReference[oaicite:3]{index=3}

この認証方式には、最低でも **2048 ビットの RSA キーペア** が必要です。 :contentReference[oaicite:4]{index=4}

OpenSSL を使用して、PEM 形式の公開鍵/秘密鍵ペアを生成できます。 :contentReference[oaicite:5]{index=5}

一部のサポート対象 Snowflake クライアントでは、**暗号化された秘密鍵** を使用して Snowflake に接続できます。 :contentReference[oaicite:6]{index=6}

公開鍵は、Snowflake クライアントを使って Snowflake に接続し認証する **Snowflake ユーザーに割り当てられます**。 :contentReference[oaicite:7]{index=7}

また Snowflake は、より堅牢なセキュリティおよびガバナンスの姿勢に準拠できるよう、**公開鍵のローテーション** もサポートしています。 :contentReference[oaicite:8]{index=8}

【補足】  
- 「ローテーション」とは、定期的に鍵を入れ替える運用のことです（秘密鍵漏えいなどのリスクを下げる目的）。  
- ここでは **公開鍵をユーザーに登録する** ことが前提になります。

---

## Supported Snowflake clients（サポートされている Snowflake クライアント）
このセクションでは、Snowflake の各クライアントが **キーペア認証** および **キーペアローテーション** をサポートしているか、また **暗号化/非暗号化の秘密鍵** を使えるかが表で整理されています。 :contentReference[oaicite:9]{index=9}

例として、次のようなクライアントが挙げられています（表内に掲載）：

- Snowflake CLI
- SnowSQL（CLI クライアント）
- Python 用 Snowflake Connector
- JDBC / ODBC / Node.js / .NET など各種ドライバー
- ほか（Spark Connector、Kafka Connector など） :contentReference[oaicite:10]{index=10}

【補足】  
クライアントによって **暗号化された秘密鍵が使えるか** が異なるため、導入前に確認が必要です。

---

## Configure key pair authentication（キーペア認証の構成）
サポートされるすべての Snowflake クライアントでキーペア認証を構成するには、**いくつかのステップ**を実行します。 :contentReference[oaicite:11]{index=11}  
（原文では、そのステップに沿って説明が進みます）

---

### Generate a private key（秘密鍵の生成）
Snowflake への接続に使用するクライアントに応じて、**暗号化された秘密鍵** または **暗号化されていない秘密鍵** を生成するオプションがあります。 :contentReference[oaicite:12]{index=12}

一般に、暗号化されたキーを生成する方が安全です。 :contentReference[oaicite:13]{index=13}

Snowflake は、このステップを完了する前に、内部のセキュリティおよびガバナンス担当者と連携して、どのキータイプを生成するか決定することを推奨しています。 :contentReference[oaicite:14]{index=14}

Snowflake は、次のアルゴリズムで生成された暗号化キーをサポートしています： :contentReference[oaicite:15]{index=15}

- RSA デジタル署名アルゴリズム  
  - RS256 / RS384 / RS512
- 楕円曲線デジタル署名アルゴリズム（ECDSA）  
  - ES256（P-256）/ ES384（P-384）/ ES512（P-512）

これらの署名はそれぞれ、SHA-256 / SHA-384 / SHA-512 のハッシュアルゴリズムを使用します。 :contentReference[oaicite:16]{index=16}

#### Tip（原文のヒント：暗号化キーとパスフレーズ）
暗号化されたキーを生成するコマンドは、キーへのアクセスを規制するための **パスフレーズ入力** を求めます。 :contentReference[oaicite:17]{index=17}

Snowflake は、ローカルで生成された秘密鍵を保護するために **PCI DSS 標準** に準拠したパスフレーズを使用することを推奨しています。 :contentReference[oaicite:18]{index=18}  
さらに、パスフレーズは安全な場所に保管することを推奨しています。 :contentReference[oaicite:19]{index=19}

暗号化されたキーを使用して Snowflake に接続する場合は、最初の接続時にパスフレーズを入力します。 :contentReference[oaicite:20]{index=20}  
パスフレーズは秘密鍵の保護のみに使用され、Snowflake には送信されません。 :contentReference[oaicite:21]{index=21}

#### OpenSSL で秘密鍵を生成する
ターミナルを開いて秘密鍵を生成します。秘密鍵は、暗号化版または非暗号化版のいずれかを生成できます。 :contentReference[oaicite:22]{index=22}

**非暗号化版（-nocrypt を使用）**：

```bash
openssl genrsa 2048 | openssl pkcs8 -topk8 -inform PEM -out rsa_key.p8 -nocrypt
```

**暗号化版（-nocrypt を省略、例として -v2 des3 を使用）**：

```bash
openssl genrsa 2048 | openssl pkcs8 -topk8 -v2 des3 -inform PEM -out rsa_key.p8
```

上記コマンドは PEM 形式の秘密鍵を生成します。 :contentReference[oaicite:23]{index=23}

---

### Assign the public key to a Snowflake user（公開鍵を Snowflake ユーザーに割り当てる）
公開鍵は、Snowflake に接続して認証する **ユーザー** に割り当てます。 :contentReference[oaicite:24]{index=24}  
（原文では、この後に「公開鍵を取り出す」「ユーザーに設定する」流れが説明されます）

【補足】  
ここでのポイントは「秘密鍵はクライアントが保持」「公開鍵は Snowflake のユーザーに登録」という役割分担です。

---

### Key-pair rotation（キーペアローテーション）
Snowflake は公開鍵のローテーションをサポートしており、より堅牢なセキュリティ／ガバナンス方針に沿った運用を行えるようにしています。 :contentReference[oaicite:25]{index=25}

【補足】  
ローテーション機能があると、運用中に鍵を入れ替える（更新する）ことが可能になります。

---

# 3. 注意点・制約（明記がある場合）

ドキュメント上で明記されている注意点・制約は次のとおりです。 :contentReference[oaicite:26]{index=26}

- キーペア認証には、最低でも **2048 ビットの RSA キーペア** が必要。 :contentReference[oaicite:27]{index=27}
- 一部のサポート対象クライアントでは **暗号化された秘密鍵** を使えるが、クライアントによって対応状況が異なる（表で確認が必要）。 :contentReference[oaicite:28]{index=28}
- 暗号化された秘密鍵を使用する場合、パスフレーズは秘密鍵保護のみに使われ、**Snowflake には送信されない**。 :contentReference[oaicite:29]{index=29}
- 公開鍵は Snowflake の **ユーザーに割り当てる**（登録する）必要がある。 :contentReference[oaicite:30]{index=30}
- Snowflake は、セキュリティ／ガバナンス要件に合わせて **公開鍵ローテーション** をサポートしている。 :contentReference[oaicite:31]{index=31}
