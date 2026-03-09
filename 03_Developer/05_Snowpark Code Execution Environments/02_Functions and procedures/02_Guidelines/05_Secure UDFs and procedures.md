# [Protecting Sensitive Information with Secure UDFs and Stored Procedures](https://docs.snowflake.com/en/developer-guide/secure-udf-procedure)

# 1. ドキュメント概要（初学者向け）

このドキュメントは、**Snowflake における「セキュア UDF / セキュアプロシージャ（Secure User-Defined Function / Secure Stored Procedure）」**について解説しています。  
これは、**ユーザー定義関数（UDF）やストアドプロシージャ内のコードを「秘密化」し、他ユーザーに関して見えないようにする仕組み**の説明です。  
つまり、**ソースコードを隠しつつ実行可能にする安全なカスタムロジックの構築方法**が学べます。

Snowflake では通常、UDF もプロシージャもユーザーが定義した SQL / JavaScript コードが内部に見える状態で保持されますが、**「SECURE」キーワードを付けることで、コードの中身が他者から見えなくなり、知的財産やロジックを秘匿できる**ようになります。

**Snowflake全体での位置づけ**

- Snowflake の基本機能である UDF / プロシージャに対する **セキュリティ強化機能**
- データベースアプリケーションの再利用性・保守性の向上
- 他者へ提供する API 風関数・処理ロジックの秘匿化

---

# 2. 原文に沿った内容整理（翻訳ベース）

## セキュア UDF/プロシージャとは（What are Secure UDFs/Procedures）

Snowflake では通常、UDF やストアドプロシージャを作成すると、**定義されたコードがオブジェクトのメタデータとしてそのまま保存され、他ユーザーが表示可能**です。  
これに対して **「SECURE」** を付与して定義された UDF/プロシージャは、**コード定義が表示不可（秘匿）**となります。  
これにより、**知的財産（IP）やビジネスロジックを隠蔽**することが可能になります。

```sql
-- 例：セキュア UDF の定義
CREATE SECURE FUNCTION my_secure_udf(x INT)
RETURNS INT
AS $$
  return x * 2;
$$;
```

【補足】  
通常の UDF/プロシージャは `SHOW FUNCTIONS/PROCEDURES` などでコードが見える場合がありますが、セキュア版は見えません。

---

## セキュアオブジェクトの機能と動作（How Secure Objects Work）

セキュア UDF/プロシージャは、以下の特徴を持ちます：

- オブジェクト定義の **コードが表示不可**
- 呼び出し（実行）は通常通り可能
- **依存オブジェクトの定義/コードが漏れないよう保護**
- クエリプロファイルや `DESCRIBE` でもコードは表示されない

【補足】  
「実行は可能 → コードは不可視」という特性により、提供側はロジックを隠しつつ機能だけ公開できます。

---

## 使用可能な言語（Supported Languages）

Snowflake のセキュア UDF/プロシージャは、**Snowflake がサポートする UDF/プロシージャ言語全般**で利用できます。  
具体的には：

- SQL UDF
- JavaScript UDF / プロシージャ
- Java UDF など

セキュア指定は、**対応言語と機能すべてに対して適用可能**です。

---

## セキュア定義の例（Examples）

### セキュア SQL UDF の例

```sql
CREATE SECURE FUNCTION add_one(v INT)
RETURNS INT
AS
  v + 1;
```

### セキュア JavaScript UDF の例

```sql
CREATE SECURE FUNCTION js_add(v NUMBER)
RETURNS NUMBER
LANGUAGE JAVASCRIPT
AS $$
  return v + 10;
$$;
```

### セキュアストアドプロシージャの例

```sql
CREATE SECURE PROCEDURE my_secure_proc()
RETURNS STRING
LANGUAGE JAVASCRIPT
AS $$
  return "Hello Secure";
$$;
```

---

## 権限とアクセス制御（Permissions and Access Control）

セキュア UDF/プロシージャは、**通常の UDF/プロシージャと同様にロールベースアクセス制御（RBAC）**が適用されます。  
ただし、**コードの可視性に関しては追加の制約が働き、権限保有者でも SQL/SHOW コマンドでソースコードが見えません**。

```sql
GRANT USAGE ON FUNCTION my_secure_udf TO ROLE some_role;
```

この例では、ロールへの使用権限の付与は可能ですが、**コードの表示権限は付与されません**。

---

## どのような場合に使うか（Use Cases）

セキュア UDF/プロシージャは、主に次のようなケースで使用されます：

- **サードパーティ提供のロジックを秘匿したい場合**
- **組織内でも特定コードを隠したい場合**
- **Snowflake 上で知的財産（IP）を保護しつつ機能を提供する場合**

【補足】  
単なる社内ユーティリティよりも、外部提供やライブラリ化を意識する場合に役立ちます。

---

# 3. 注意点・制約（明記がある場合）

- セキュア UDF/プロシージャは **コードが表示不可**であり、  
  この性質は意図された動作です。  
- ロールベースアクセスとは別に、**コード非表示というセキュリティ属性が付与される**ため、  
  明示的にコードを確認・検証することができません。  
- セキュアオブジェクトは通常のオブジェクトと同じく管理できますが、  
  **コードの保守は定義元でしか行えません**。

→  
**ドキュメント上で注意点・制約が明記されています**

---

# 4.メモ
- [プッシュダウン](https://docs.snowflake.com/ja/developer-guide/pushdown-optimization.html#label-secure-objects-pushdown)など、一部のUDFsの内部最適化では、ベーステーブルの元になるデータへのアクセスが必要
  - セキュアUDFsはこれらの最適化を使用しない
    ⇒**ユーザが基になるデータに間接的にアクセスすることはない**
  
