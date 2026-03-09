# [Multi-factor authentication (MFA)](https://docs.snowflake.com/en/user-guide/security-mfa)

# 1. ドキュメント概要（初学者向け）

このドキュメントは、Snowflake における **多要素認証（MFA: Multi-factor authentication）** について説明しています。  
MFA を有効にすると、パスワード認証を行うユーザーは、Snowflake にサインインする際に **パスワードに加えて「第二の認証要素」** を使う必要があります。これにより、パスワードが漏えいした場合でも不正ログインのリスクを下げられます。 :contentReference[oaicite:0]{index=0}

## Snowflake全体の中での位置づけ
Snowflake では、アカウントやユーザーの安全性を高めるために複数の認証方式（SSO、OAuth、キーペア認証など）を提供しています。  
その中で MFA は、特に **パスワード認証を利用するユーザー** のサインインを強化するための重要なセキュリティ機能です。 :contentReference[oaicite:1]{index=1}

【補足】  
このページの MFA は主に **Web UI（Snowsight）でのサインイン**を想定していますが、Snowflake CLI / SnowSQL / JDBC / Node.js / ODBC でも完全にサポートされています。 :contentReference[oaicite:2]{index=2}

---

# 2. 原文に沿った内容整理（翻訳ベース）

# Multi-factor authentication (MFA)（多要素認証）

## MFAとは（概要）
多要素認証（MFA）は、**パスワード認証に伴うセキュリティリスクを低減**します。 :contentReference[oaicite:3]{index=3}  
パスワードユーザーが MFA に登録されると、Snowflake にサインインする際に **第二の認証要素** を使用する必要があります。 :contentReference[oaicite:4]{index=4}  
これらのユーザーは、まずパスワードを入力し、その後に第二要素を使用します。 :contentReference[oaicite:5]{index=5}

【補足】  
「第二要素」は、スマホの認証アプリ・パスキー・Duo など、パスワード以外の追加認証手段です。

---

## MFAによるSnowflakeへの接続
MFA ログインは主に **Web インターフェイス** で Snowflake に接続するために設計されていますが、次のクライアントでも **完全にサポート**されています。 :contentReference[oaicite:6]{index=6}

- Snowflake CLI
- SnowSQL
- Snowflake JDBC ドライバー
- Snowflake Node.js ドライバー
- Snowflake ODBC ドライバー :contentReference[oaicite:7]{index=7}

【補足】  
「Web だけではなく、CLI/ドライバ接続でも MFA を使える」ことが明記されています。

---

## MFA の方法（第二認証要素の種類）
（このページでは、MFA の第二要素を追加して利用する流れが説明されています）

Snowflake が提示する第二認証要素の例： :contentReference[oaicite:8]{index=8}

- **パスキー（Passkey）** による認証  
  WebAuthn 標準に基づく認証で、公開鍵/秘密鍵暗号方式を使用します。 :contentReference[oaicite:9]{index=9}
- **認証アプリ（Authenticator app）** による認証  
  時間ベースのワンタイムパスコード（TOTP）を第二要素として使用できます。 :contentReference[oaicite:10]{index=10}
- **Duo** による認証 :contentReference[oaicite:11]{index=11}

【補足】  
管理者は、どの MFA 方法を利用可能にするかをコントロールできます（詳細は関連トピック参照）。 :contentReference[oaicite:12]{index=12}

---

## 第二認証の構成（Snowsight での設定）
管理者がユーザーに対して MFA 登録を要求している場合、そのユーザーは **次回 Snowsight にサインインする際**、第二認証要素を追加するよう促されます。 :contentReference[oaicite:13]{index=13}

すでに Snowsight にサインインしていて、第二要素をセットアップしたい場合は次の手順です。 :contentReference[oaicite:14]{index=14}

1. 左側のナビゲーションで **自分の名前** を選択  
2. ユーザーメニューが開くので **Settings** を選択  
3. **Authentication** を選択  
4. **Multi-factor authentication** セクションで **Add new authentication method** を選択  
5. 画面の指示に従って第二認証を構成する :contentReference[oaicite:15]{index=15}

---

### パスキー認証の使用（Use passkey authentication）
パスキーは WebAuthn 標準に基づく認証で、公開鍵/秘密鍵暗号方式を使用します。 :contentReference[oaicite:16]{index=16}  
Snowflake をパスキー認証で構成すると、秘密キーは以下のような個人の場所に安全に保存されます： :contentReference[oaicite:17]{index=17}

- マシン
- ハードウェアセキュリティキー（例：Yubikey）
- パスワードマネージャ

第二認証としてパスキーを設定するには、プロンプトが表示されたら **Passkey** を選択し、他の Web サイトやアプリケーションと同様に保存手順を完了します。 :contentReference[oaicite:18]{index=18}  
その後、Snowflake にサインインする際に識別できるように、認証方法の名前を指定します。 :contentReference[oaicite:19]{index=19}  
パスワード入力後、構成した方法でパスキーを使用するよう求められます。 :contentReference[oaicite:20]{index=20}

【補足】  
パスキーは「スマホ生体認証」「セキュリティキー」などと組み合わせて使うケースがあります。

---

### 認証アプリの使用（Use an authenticator app）
Snowflake では、好みの認証アプリを使用して **時間ベースのワンタイムパスコード（TOTP）** を第二要素として使用できます。 :contentReference[oaicite:21]{index=21}  
一般的な認証アプリには次があります： :contentReference[oaicite:22]{index=22}

- Google Authenticator
- Microsoft Authenticator
- Authy

認証アプリを第二要素として設定するには、プロンプトで **Authenticator** を選択し、他の Web サイト等と同様に手順を完了します。 :contentReference[oaicite:23]{index=23}  
その後、認証方法の名前を指定し、サインイン時にパスワードの後で TOTP を入力します。 :contentReference[oaicite:24]{index=24}

---

### Duoの使用（Use Duo）
Duo を第二要素として設定するには、プロンプトで **DUO** を選択し、他の Web サイト等と同様に手順を完了します。 :contentReference[oaicite:25]{index=25}

**注記（Note）**  
Duo を第二認証として使用するには、管理者が組織のファイアウォールを構成する必要があります。 :contentReference[oaicite:26]{index=26}

---

## 認証方法の表示（View authentication methods）
第二認証は、**Snowsight / SQL** で表示できます。 :contentReference[oaicite:27]{index=27}

### Snowsight で表示する
1. Snowsight にサインインします  
2. 左側のナビゲーションで自分の名前を選択  
3. **Settings** を選択  
4. **Authentication** を選択  
5. **Multi-factor authentication** セクションで MFA 方法を表示します :contentReference[oaicite:28]{index=28}

**注記（Note）**  
管理者として他のユーザーの認証方法を表示したい場合は **`SHOW MFA METHODS`** を参照してください。 :contentReference[oaicite:29]{index=29}

また、アカウント内のすべてのユーザーについて、パスキーおよび TOTP の情報は **`CREDENTIALS` ビュー** をクエリします。 :contentReference[oaicite:30]{index=30}  
このビューには、Duo 認証方式（Duo プッシュおよびパスコード）の情報は含まれない点に注意してください。 :contentReference[oaicite:31]{index=31}

---

## デフォルト認証方法のセット（Set a default sign-in method）
第二要素として **2 つ以上** の MFA 方法を構成している場合、パスワード入力後の認証に使用する方法を選択できます。 :contentReference[oaicite:32]{index=32}

デフォルトの第二要素を設定するには次の手順です： :contentReference[oaicite:33]{index=33}

1. 左側のナビゲーションで自分の名前を選択  
2. **Settings** を選択  
3. **Authentication** を選択  
4. **Multi-factor authentication** セクションで **Default sign-in method** から MFA 方法を選択

---

## 第二要素の認証情報が使用されたログインセッションの識別
第二要素の認証情報（例：特定のパスキー、または特定の TOTP）が認証に使用されたかどうかを判断するには、  
`LOGIN_HISTORY` と `CREDENTIALS` ビューを、認証情報 ID で結合します。 :contentReference[oaicite:34]{index=34}

- `LOGIN_HISTORY` ビュー
  - `second_authentication_factor` が `PASSKEY` または `TOTP` の場合、`second_authentication_factor_id` 列に認証情報 ID が含まれます。 :contentReference[oaicite:35]{index=35}
- `CREDENTIALS` ビュー
  - `credential_id` 列に認証情報 ID が含まれています。 :contentReference[oaicite:36]{index=36}

ドキュメント内の例（そのまま記載）：

```sql
SELECT login.event_timestamp, login.user_name, cred.name
FROM SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY login
JOIN SNOWFLAKE.ACCOUNT_USAGE.CREDENTIALS cred
  ON login.second_authentication_factor_id = cred.credential_id
WHERE login.second_authentication_factor IN ('PASSKEY', 'TOTP');
```

このログインセッション中に実行されたクエリの情報を取得するには、  
`LOGIN_HISTORY` を `SESSIONS` ビュー（`login_event_id` を含む列）に結合し、さらに `QUERY_HISTORY` ビューへ結合します。 :contentReference[oaicite:37]{index=37}

【補足】  
「MFAが使われたログインかどうか」「どの認証情報（どのパスキー/TOTP）だったか」を監査できる、という説明です。

---

## MFA方法の無効化（例：別ユーザーのOTPを無効化）
`REMOVE MFA METHOD` コマンドを使用すると、別のユーザーの特定の OTP を無効化できます。 :contentReference[oaicite:38]{index=38}  
自分自身の OTP を無効化する場合は、Snowsight を使用します。 :contentReference[oaicite:39]{index=39}

例（ユーザー joe の OTP_2 を無効化）：

```sql
ALTER USER joe REMOVE MFA METHOD OTP_2;
```

---

# 3. 注意点・制約（明記がある場合）

ドキュメント上で明記されている注意点・制約は次のとおりです。 :contentReference[oaicite:40]{index=40}

- MFA はパスワード認証のリスクを低減し、MFA 登録ユーザーは **サインイン時に第二要素が必須**になる。 :contentReference[oaicite:41]{index=41}
- MFA ログインは主に Web UI で設計されているが、Snowflake CLI / SnowSQL / JDBC / Node.js / ODBC でも完全にサポートされる。 :contentReference[oaicite:42]{index=42}
- Duo を第二要素として利用するには、管理者による **組織ファイアウォール構成が必要**。 :contentReference[oaicite:43]{index=43}
- `CREDENTIALS` ビューには **Duo 認証方式の情報が含まれない**。 :contentReference[oaicite:44]{index=44}
- 別ユーザーの OTP を無効化するには `ALTER USER ... REMOVE MFA METHOD ...` を使用できるが、自分の OTP を無効化する場合は **Snowsight を使用**する。 :contentReference[oaicite:45]{index=45}
