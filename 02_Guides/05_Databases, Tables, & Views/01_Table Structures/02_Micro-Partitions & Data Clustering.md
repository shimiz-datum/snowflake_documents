# []()

# 1. ドキュメント概要（初学者向け）

このドキュメントは、Snowflake がテーブルデータを内部で管理する仕組みである **マイクロパーティション（Micro-partition）** と、クエリ性能に関係する **データクラスタリング（Data Clustering）** について説明しています。 :contentReference[oaicite:0]{index=0}

Snowflake では、テーブルに取り込まれたデータが自動的に小さな単位に分割され、その単位ごとにメタデータ（列の値の範囲など）が管理されます。  
この仕組みにより、クエリ実行時に不要なデータを読み飛ばす（プルーニングする）ことができ、パフォーマンス最適化につながります。 :contentReference[oaicite:1]{index=1}

## Snowflake 全体の中での位置づけ
マイクロパーティションは Snowflake の **ストレージレイヤー（データの保存・管理）** における基盤技術で、クエリ実行時の効率化（特にスキャン量削減）に深く関係します。 :contentReference[oaicite:2]{index=2}  
また、クラスタリング（並び順の偏り）に関する情報を活用してクエリ性能を高める考え方は、テーブル設計・運用最適化の重要なテーマです。 :contentReference[oaicite:3]{index=3}

---

# 2. 原文に沿った内容整理（翻訳ベース）

# マイクロパーティションとデータクラスタリング（Micro-partitions & Data Clustering）

従来のデータウェアハウスは、許容可能なパフォーマンスを実現し、より良いスケーリングを可能にするために、大きなテーブルを **静的パーティション（static partitions）** に依存していることが一般的です。 :contentReference[oaicite:4]{index=4}  
これらのシステムでは、パーティションは特別な DDL と構文を使って個別に操作される **管理単位** です。 :contentReference[oaicite:5]{index=5}  
しかし、静的パーティションには、メンテナンスオーバーヘッドやデータスキュー（偏り）など、既知の制限がいくつかあり、パーティションサイズが不均衡になる可能性があります。 :contentReference[oaicite:6]{index=6}

それに対して Snowflake Data Platform は **マイクロパーティショニング（micro-partitioning）** と呼ばれる独自の強力なパーティショニング形式を実装しています。 :contentReference[oaicite:7]{index=7}  
この仕組みにより、静的パーティショニングの利点を維持しつつ、既知の制限を回避し、さらに追加の有用なメリットを提供します。 :contentReference[oaicite:8]{index=8}

**注意（Note）**  
ハイブリッドテーブル（Hybrid tables）は、標準的な Snowflake テーブルで利用できるクラスタリングキーなど、一部機能をサポートしていないアーキテクチャに基づいています。 :contentReference[oaicite:9]{index=9}

【補足】  
このページで解説される「マイクロパーティション」「クラスタリング」は、主に **標準的な Snowflake テーブル**を対象としています。

---

## マイクロパーティションとは何ですか？（What are micro-partitions?）

Snowflake テーブル内のすべてのデータは、自動的に **マイクロパーティション** と呼ばれる連続したストレージ単位に分割されます。 :contentReference[oaicite:10]{index=10}

各マイクロパーティションには、**非圧縮データで 50MB〜500MB** が含まれます  
（ただし、データは常に圧縮して保存されるため、Snowflake での実際のサイズは小さくなります）。 :contentReference[oaicite:11]{index=11}

テーブル内の行のグループが個々のマイクロパーティションに対応付けられ、データは **列指向の状態（columnar）** に整理されます。 :contentReference[oaicite:12]{index=12}  
このサイズと構造により、非常に大きなテーブルに対しても **非常に細かい粒度のプルーニング** が可能になります。 :contentReference[oaicite:13]{index=13}  
テーブルは数百万、または数億のマイクロパーティションで構成される場合があります。 :contentReference[oaicite:14]{index=14}

Snowflake は、マイクロパーティション内のすべての行に関して次のようなメタデータを保存します： :contentReference[oaicite:15]{index=15}

- 各マイクロパーティションの各列の **値の範囲**
- **個別の値の数**
- 最適化や効率的なクエリ処理のために使われる追加プロパティ

**注記（注釈）**  
マイクロパーティションは **すべての Snowflake テーブルで自動的に行われます**。 :contentReference[oaicite:16]{index=16}  
また、テーブルは挿入／ロードされたデータの順序に基づいて透過的にパーティション分割されます。 :contentReference[oaicite:17]{index=17}

【補足】  
ユーザーが「パーティションを作る DDL」を書かなくても、Snowflake が内部で自動的に分割・管理します。

---

## マイクロパーティション分割の利点（Benefits of micro-partitioning）

Snowflake のテーブルデータ分割アプローチには、次の利点があります： :contentReference[oaicite:18]{index=18}

- 静的パーティションとは異なり、Snowflake のマイクロパーティションは **自動的に導出** される  
  → 事前定義やユーザー保守が不要 :contentReference[oaicite:19]{index=19}
- マイクロパーティションは小さく（圧縮前 50〜500MB）、DML の効率と細かいプルーニングにより **クエリ高速化** が期待できる :contentReference[oaicite:20]{index=20}
- マイクロパーティションは値の範囲が互いに重複することがあり、均一に小さいサイズと組み合わさって **スキューの抑制** に役立つ :contentReference[oaicite:21]{index=21}
- 列はマイクロパーティション内に独立して保存され（列指向ストレージ）、**個々の列を効率的にスキャン** できる :contentReference[oaicite:22]{index=22}  
  → クエリで参照される列のみがスキャンされる :contentReference[oaicite:23]{index=23}
- 列はマイクロパーティション内で個別に圧縮され、Snowflake が各列に最適な圧縮アルゴリズムを自動選択する :contentReference[oaicite:24]{index=24}

また、テーブルに **クラスタリングキー** を指定することで、特定テーブルでクラスタリングを有効化できます。 :contentReference[oaicite:25]{index=25}  
（クラスタリングキーの指定は `CREATE TABLE` / `ALTER TABLE` を参照、とされています） :contentReference[oaicite:26]{index=26}

【補足】  
列指向＋メタデータ保持により「必要な列」「必要な範囲」だけ読みやすくなる、というのが Snowflake の効率化の中心です。

---

## マイクロパーティションの影響（Impact of micro-partitions）

### DML
すべての DML 操作（例：`DELETE`、`UPDATE`、`MERGE`）は、基礎となるマイクロパーティションメタデータを利用して、テーブルメンテナンスを容易にし、簡素化します。 :contentReference[oaicite:27]{index=27}  
例えば、テーブルからすべての行を削除するなどの一部操作は **メタデータのみの操作** です。 :contentReference[oaicite:28]{index=28}

【補足】  
「物理的に全行を消す」のではなく、内部的にメタデータ処理として済むケースがある、という説明です。

---

### テーブル列の削除
テーブルの列がドロップされると、ドロップされた列のデータが含まれるマイクロパーティションは、`DROP` 実行時に **再書き込みされません**。 :contentReference[oaicite:29]{index=29}  
ドロップされた列のデータはストレージにそのまま残ります。 :contentReference[oaicite:30]{index=30}  
詳細は `ALTER TABLE` の使用上の注意を参照するよう記載されています。 :contentReference[oaicite:31]{index=31}

【補足】  
列を削除しても「即座に物理削除される」とは限らない、という点を示しています。

---

### クエリプルーニング（Query pruning）
Snowflake が保持するマイクロパーティションメタデータにより、クエリ実行時にマイクロパーティション内の列（半構造化データの列を含む）を正確にプルーニングできます。 :contentReference[oaicite:32]{index=32}

言い換えると、値の範囲の 10% にアクセスするフィルター述語（predicate）を指定するクエリは、理想的には **マイクロパーティションの 10% のみをスキャン**するべきです。 :contentReference[oaicite:33]{index=33}

例として、日付と時間の列を持つ大きなテーブルに 1 年分の履歴データがあるとします。  
データが均一に分布していると仮定すると、特定時間を対象とするクエリは、理想的にはテーブル内の **1/8760** のマイクロパーティションをスキャンし、時間列のデータを含むマイクロパーティションの部分のみをスキャンします。 :contentReference[oaicite:34]{index=34}

また Snowflake は列指向スキャンを用いるため、クエリが 1 列でのみフィルターする場合、パーティション全体がスキャンされるわけではありません。 :contentReference[oaicite:35]{index=35}  
スキャンされたマイクロパーティションと列指向データの比率が、実際に選択されたデータ比率に近いほど、プルーニングはより効率的です。 :contentReference[oaicite:36]{index=36}

時系列データでは、このレベルのプルーニングにより、範囲クエリ（スライス）に対して 1 時間以下の細粒度の潜在的応答時間が可能になります。 :contentReference[oaicite:37]{index=37}

ただし、すべての述語式をプルーニングに利用できるわけではありません。  
例えば、Snowflake は、サブクエリの結果が定数になる場合でも、サブクエリの述語に基づいてマイクロパーティションをプルーニングしません。 :contentReference[oaicite:38]{index=38}

【補足】  
「述語の書き方」によって、どれだけ読み飛ばせるかが変わる可能性がある、という注意です。

---

## データクラスタリングとは（What is data clustering?）

通常、テーブルに格納されるデータは、日付や地域などの自然な次元に沿って並べ替えられます。  
この「クラスタリング」はクエリの重要要素であり、とくに非常に大きなテーブルでは、未ソートまたは部分的にしかソートされていないデータがクエリ性能へ影響する場合があります。 :contentReference[oaicite:39]{index=39}

Snowflake では、データがテーブルに挿入／ロードされると、その過程で作成された各マイクロパーティションの **クラスタリングメタデータ** が収集・記録されます。 :contentReference[oaicite:40]{index=40}  
Snowflake はこのクラスタリング情報を活用して、クエリ中の不要なマイクロパーティションスキャンを回避し、これら列を参照するクエリのパフォーマンスを大幅に向上させます。 :contentReference[oaicite:41]{index=41}

（原文には、日付でソートされた 4 列テーブルの図があり、Snowflake が「不要なマイクロパーティション→残りの中の列」へ段階的にプルーニングする概念が説明されています。） :contentReference[oaicite:42]{index=42}

【補足】  
ここでの「クラスタリング」は、ユーザーがソート命令を書くというより、データの偏り・まとまり方（値の範囲がどの程度重なるか）を指しています。

---

## マイクロパーティション用に維持されるクラスタリング情報（Clustering information maintained）

Snowflake はテーブルに対して、次のようなクラスタリングメタデータを保持します： :contentReference[oaicite:43]{index=43}

- テーブルを構成するマイクロパーティションの総数
- （指定した列のサブセット内で）互いに重複する値を含むマイクロパーティションの数
- 重なり合うマイクロパーティションの深さ

---

### クラスタリングの深さ（Clustering depth）
取り込まれたテーブルのクラスタリングの深さは、テーブル内の指定列における **重複するマイクロパーティションの平均深度（1 以上）** を測定します。 :contentReference[oaicite:44]{index=44}  
平均深度が小さいほど、その列に関してテーブルはよりクラスタ化されています。 :contentReference[oaicite:45]{index=45}

クラスタリングの深さは、次のような目的で利用できます： :contentReference[oaicite:46]{index=46}

- 大きなテーブルのクラスタリングの「健全性」を監視（特に DML が実行される場合）
- 大きなテーブルで、クラスタリングキーを明示定義することが有益か判断

また、マイクロパーティションがないテーブル（取り込まれていない／空のテーブル）のクラスタリングの深さは **0** です。 :contentReference[oaicite:47]{index=47}

**注記（注釈）**  
クラスタリングの深さは、テーブルが適切にクラスタ化されているかどうかの **絶対的または正確な尺度ではありません**。 :contentReference[oaicite:48]{index=48}  
最終的に、クエリパフォーマンスが「どれだけ適切にクラスタ化されているか」を示す最適な指標です： :contentReference[oaicite:49]{index=49}

- クエリが必要または期待どおりに動作しているなら、適切にクラスタ化されている可能性がある
- クエリ性能が時間とともに低下するなら、十分にクラスタ化されておらず、クラスタリングの恩恵を受ける可能性がある

【補足】  
数値（深さ）は参考値で、最終判断は「実際のクエリが速いかどうか」を見る、という考え方です。

---

### クラスタリングの深さの図解（Illustration）
原文では、A〜Z の範囲を持つ 5 つのマイクロパーティションの概念例により、「値の範囲の重複」がクラスタリングの深さへどう影響するかが説明されています。 :contentReference[oaicite:50]{index=50}

示されているポイントは次のとおりです： :contentReference[oaicite:51]{index=51}

- 当初はすべてのマイクロパーティションの値範囲が重複している
- 重複するマイクロパーティション数が減るほど、重なりの深さが減少する
- 値範囲が完全に重複しない場合、マイクロパーティションは「一定の状態（constant state）」とみなされ、クラスタリングで改善できない

また、これは実テーブルではなく概念図であり、実際の大規模テーブルでは全マイクロパーティションが一定状態に達することはほとんどなく、またその必要もない、と説明されています。 :contentReference[oaicite:52]{index=52}

---

## テーブルのクラスタリング情報の監視（Monitoring clustering information）
テーブルのクラスタリングメタデータを表示／監視するために、Snowflake は次のシステム関数を提供します： :contentReference[oaicite:53]{index=53}

- `SYSTEM$CLUSTERING_DEPTH`
- `SYSTEM$CLUSTERING_INFORMATION`（クラスタリング深さを含む）

これらの関数がクラスタリングメタデータをどのように使用するかの詳細は、「クラスタリングの深さの図解」を参照するよう記載されています。 :contentReference[oaicite:54]{index=54}

---

# 3. 注意点・制約（明記がある場合）

ドキュメント上で明記されている注意点・制約は次のとおりです。 :contentReference[oaicite:55]{index=55}

- **ハイブリッドテーブル**は、標準テーブルで利用可能なクラスタリングキーなど、一部機能をサポートしない。 :contentReference[oaicite:56]{index=56}
- マイクロパーティションは **非圧縮で 50MB〜500MB** を含み、データは常に圧縮して保存される。 :contentReference[oaicite:57]{index=57}
- すべての Snowflake テーブルで **マイクロパーティションは自動的に作成**され、ユーザーが明示定義・保守する必要はない。 :contentReference[oaicite:58]{index=58}
- すべての述語式がプルーニングに利用できるわけではない（例：サブクエリ述語によるプルーニングは行わない）。 :contentReference[oaicite:59]{index=59}
- 列をドロップしても、該当データを含むマイクロパーティションは **即時に再書き込みされず**、ドロップ列のデータはストレージに残る。 :contentReference[oaicite:60]{index=60}
- **クラスタリングの深さは絶対的・正確な尺度ではない**。最終的な指標はクエリパフォーマンスである。 :contentReference[oaicite:61]{index=61}
