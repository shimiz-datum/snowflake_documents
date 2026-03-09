# [Snowflake editions](https://docs.snowflake.com/en/user-guide/intro-editions)

# 1. ドキュメント概要（初学者向け）

このドキュメントは、Snowflake が提供する **エディション（Edition）** の種類と、それぞれの位置づけ・特徴を説明しています。  
Snowflake では、利用できる機能やセキュリティレベル、サポート範囲などが **契約エディションによって異なる**ため、アカウント設計や運用方針を考える上で重要な情報です。 :contentReference[oaicite:0]{index=0}

## Snowflake全体の中での位置づけ
Snowflake のエディションは「Snowflake をどのレベルの機能・保護・運用要件で使うか」を決める枠組みです。  
たとえば、より強いセキュリティ・ガバナンス・BCP（事業継続）要件が必要な場合、上位エディションが選択肢になります。 :contentReference[oaicite:1]{index=1}

---

# 2. 原文に沿った内容整理（翻訳ベース）

# Snowflake Edition（Snowflake エディション）

このトピックでは、Snowflake のエディションの概要と、各エディションに含まれる機能のマトリックスを説明します。 :contentReference[oaicite:2]{index=2}

---

## 容量（Capacity）
Snowflake は、**事前の容量コミットメント** に基づく割引価格を提供します。 :contentReference[oaicite:3]{index=3}  
価格の詳細については、価格設定ページ（Snowflake Web サイト）を参照してください。 :contentReference[oaicite:4]{index=4}

【補足】  
ここでいう容量コミットメントは、一定量の利用を前提とした契約形態を指します。

---

## このトピックの内容（This topic covers）
このページでは、次の内容を扱います。 :contentReference[oaicite:5]{index=5}

- エディションの概要
  - Standard Edition
  - Enterprise Edition
  - Business Critical Edition
  - Virtual Private Snowflake（VPS）
- 機能/エディションのマトリックス

---

## エディションの概要（Edition overview）

### Standard Edition
Standard Edition は、Snowflake のすべての標準機能への **完全かつ無制限のアクセス**を提供する **入門レベルの製品**です。 :contentReference[oaicite:6]{index=6}  
機能、サポートのレベル、およびコストの間のバランスが優れています。 :contentReference[oaicite:7]{index=7}

【補足】  
「最初に Snowflake を使い始めるための基本エディション」という位置づけです。

---

### Enterprise Edition
Enterprise Edition は、Standard Edition のすべての機能とサービスを提供し、大規模な企業や組織のニーズに合わせて特別に設計された **追加機能** を備えています。 :contentReference[oaicite:8]{index=8}

【補足】  
標準機能に加えて、企業向けに必要になりやすい機能が追加されるイメージです。

---

### Business Critical Edition（以前は Enterprise for Sensitive Data）
以前 Enterprise for Sensitive Data（ESD）と呼ばれていた Business Critical Edition は、さらに高いレベルのデータ保護を提供し、**非常に機密性の高いデータ**、特に HIPAA と HITRUST CSF 規制に準拠する必要がある **PHI データ** を持つ組織のニーズをサポートします。 :contentReference[oaicite:9]{index=9}

Enterprise Edition のすべての機能とサービスに加えて、**セキュリティとデータ保護が強化**されています。 :contentReference[oaicite:10]{index=10}  
さらに、アカウントの **フェールオーバー/フェールバック** により、事業継続とディザスタリカバリをサポートします。 :contentReference[oaicite:11]{index=11}

#### 注記（HIPAA/HITRUSTとBAA）
HIPAA と HITRUST CSF 規制の要件に従い、PHI データを Snowflake に保存する前に、代理店/組織と Snowflake Inc の間で署名済みの **ビジネスアソシエイト契約（BAA）** を確立する必要があります。 :contentReference[oaicite:12]{index=12}

【補足】  
- PHI は医療関連の高機微情報（患者情報など）を含む概念です。  
- BAA は、規制要件に沿った契約の枠組みです。

---

### Virtual Private Snowflake（VPS）
Virtual Private Snowflake は、非常に機密性の高いデータを収集、分析、共有する金融機関やその他の大企業など、最も要件が厳しい組織に **最高レベルのセキュリティ** を提供します。 :contentReference[oaicite:13]{index=13}

これには Business Critical Edition のすべての機能とサービスが含まれていますが、**完全に別の Snowflake 環境に置かれており**、他のすべての Snowflake アカウントから分離されています。 :contentReference[oaicite:14]{index=14}  
つまり VPS アカウントは、VPS 外部のアカウントと **ハードウェアリソースを共有しません**。 :contentReference[oaicite:15]{index=15}

#### 注記（アカウント識別子とアカウントロケーター形式）
アカウントにアクセスするには、組織名とアカウント名を指定する **アカウント識別子** を使用できます。 :contentReference[oaicite:16]{index=16}  
代わりに、アカウント識別子として **アカウントロケーター** を使用することを選択した場合、VPS アカウントのアカウントロケーターは、他の Snowflake エディションのアカウントとは異なる形式を使用することに注意してください。 :contentReference[oaicite:17]{index=17}  
詳細は「VPSアカウントのアカウントロケーター形式の検出」を参照してください。 :contentReference[oaicite:18]{index=18}

【補足】  
VPS は「他の利用者と物理リソースを共有しない」ことが重要な点として説明されています。

---

## 機能/エディションのマトリックス（Feature/edition matrix）
次のテーブルに、各エディションに含まれる主要な機能とサービスのリストを示します。 :contentReference[oaicite:19]{index=19}

注記：これは機能の **部分的なリスト** にすぎません。 :contentReference[oaicite:20]{index=20}  
より完全で詳細なリストについては、「主な機能の概要」を参照してください。 :contentReference[oaicite:21]{index=21}

（原文にはカテゴリ別に機能一覧が表形式で掲載されています）

### 例：リリース管理（Release management）
- 新しい週次リリースへの **24時間前の早期アクセス**は、各リリースが実稼働環境アカウントにデプロイされる前に、追加のテストや検証に使用できます。 :contentReference[oaicite:22]{index=22}

### 例：セキュリティ、ガバナンス、およびデータ保護（Security, governance, and data protection）
原文の表では、例えば次のような機能が並びます（カテゴリの一部例）： :contentReference[oaicite:23]{index=23}

- SOC 2 Type II 認証
- SSO
- OAuth
- ネットワークポリシー
- すべてのデータの暗号化
- 多要素認証（MFA）
- オブジェクトレベルアクセス制御
- Time Travel（標準：最大1日）
- Fail-safe（Time Travel以後の7日間）
- オブジェクトタグ（※一部のタグ機能は Enterprise 以上が必要）
- Time Travel 延長（最大90日）
- 暗号化キーの定期的な更新
- マスキングポリシー（列レベルセキュリティ）
- 行アクセスポリシー（行レベルセキュリティ）
- 集計ポリシー
- 投影ポリシー
- 差分プライバシー
- 分類（classification）
- ACCESS_HISTORY ビュー（Account Usage）
- イベントテーブル
- Tri-Secret Secure（顧客管理暗号鍵） など :contentReference[oaicite:24]{index=24}

【補足】  
ここは「どのエディションでどの機能が使えるか」を表で確認するセクションです。実際の運用要件（監査、暗号、復旧要件など）に合わせて参照します。

---

# 3. 注意点・制約（明記がある場合）

ドキュメント上で明記されている注意点・制約は次のとおりです。 :contentReference[oaicite:25]{index=25}

- 機能/エディションのマトリックスは **主要機能の部分的なリスト** であり、完全版は別ページ（主な機能の概要）を参照する必要がある。 :contentReference[oaicite:26]{index=26}
- Business Critical Edition で PHI データを保存する前に、HIPAA/HITRUST CSF 要件に従い Snowflake との **BAA（ビジネスアソシエイト契約）** を確立する必要がある。 :contentReference[oaicite:27]{index=27}
- VPS アカウントは他エディションと **アカウントロケーター形式が異なる**点に注意が必要。 :contentReference[oaicite:28]{index=28}
- <span style= "color: red; ">Snowflake MarketplaceはSnowflakeユーザー以外も閲覧できる</span>が、マーケットプレイスのデータを利用するにはSnowflakeエディションへのサインアップが必要。