# [Introduction to Streams](https://docs.snowflake.com/en/user-guide/streams-intro)

# 1. ドキュメント概要（初学者向け）

このドキュメントは、**Snowflake における「ストリーム（Streams）」の基本概念**を説明しています。  
「ストリーム」は、テーブルやビューなどのソースオブジェクトに対して行われた **DML（データ変更操作：INSERT・UPDATE・DELETE）** の変更を追跡し、**変更データキャプチャ（CDC：Change Data Capture）** を実現するための仕組みです。:contentReference[oaicite:0]{index=0}

Snowflake 内では、変更データキャプチャを行うための公式なオブジェクトとしてストリームが提供され、**増分データ処理やデータパイプライン**の構築などで利用されます。:contentReference[oaicite:1]{index=1}

---

# 2. 原文に沿った内容整理（翻訳ベース）

## ストリームとは（What is a stream）

ストリーム・オブジェクトは、テーブルへの DML 操作（INSERT／UPDATE／DELETE／`COPY INTO` を含む）による変更と、**各変更に関するメタデータを記録するオブジェクト**です。  
この仕組みによって、**変更されたデータを把握し、処理のトリガーやパイプラインに利用できます**。:contentReference[oaicite:2]{index=2}

個々のテーブルストリーム（単に「ストリーム」）は、**ソーステーブルの行に加えられた変更を追跡**します。ストリームは、テーブル内の 2 つのトランザクションポイント（オフセット）の間で行レベルの変更内容を表す「変更テーブル」を提供し、この一連の変更記録を **トランザクションとしてクエリおよび利用可能**にします。:contentReference[oaicite:3]{index=3}

【補足】  
ストリームは変更の詳細を管理する「差分ビュー」のように働き、データの増分だけを扱えるメリットがあります。

---

## ストリームで追跡できるオブジェクト（Streams can be created on）

ストリームは、次の種類のオブジェクトの変更データをクエリできます：:contentReference[oaicite:4]{index=4}

- 標準テーブル（共有テーブルを含む）  
- ビュー（セキュアビューを含む）  
- ディレクトリテーブル  
- ダイナミックテーブル  
- Apache Iceberg™ テーブル（制限あり）  
- イベントテーブル  
- 外部テーブル

---

## オフセットストレージ（Offset Storage）

ストリームを作成すると、**現時点のソースオブジェクトのトランザクション版（オフセット）**として初期スナップショットが取得されます。  
その後にコミットされた DML の変更は、ストリームの変更追跡システムによって記録されます。:contentReference[oaicite:5]{index=5}

変更レコードは、**変更前後の行の状態**を提供し、追跡対象オブジェクトの列構造を反映すると同時に、各変更イベントを説明する追加のメタデータ列を含みます。:contentReference[oaicite:6]{index=6}

ストリーム自体はソースオブジェクトのデータを含まず、**オフセット（時点情報）だけを保存**し、Snowflake のバージョン管理履歴を活用して変更レコードを返します。:contentReference[oaicite:7]{index=7}

【補足】  
オフセットは「変更を読み取る開始位置」を表すポイントであり、それ以降の変更を差分として扱います。

---

## ブックマークとしての比喩（Thinking of streams as bookmarks）

ストリームを **ブックのページのブックマーク**に例えると分かりやすいです。  
ブックマークはページの位置を示し、別の位置に入れ替えることができます。  
同様に、ストリームはドロップして再作成することで、**異なるオフセット（時点）から変更を読み取れる**ようになります。:contentReference[oaicite:8]{index=8}

---

## テーブルのバージョン管理（Table Versioning）

トランザクション内の DML ステートメントがテーブルにコミットされるたびに、**新しいテーブルバージョン**が作成されます。  
ストリームのオフセットはこうした **2 つのバージョンの間に位置**します。ストリームをクエリすると、オフセットより後で現在時刻以前にコミットされた変更が返されます。:contentReference[oaicite:9]{index=9}

例えば、テーブルに 10 のバージョンがあるとき、オフセット位置が 3 と 4 の間であれば、`v4` 〜 `v10` までの **全ての変更が返されます**。:contentReference[oaicite:10]{index=10}

![alt text](../../../../image/Introduction%20to%20Streams.png)

---

## 複数のクエリとオフセット（Multiple queries and offset）

複数のクエリが同じストリームを参照しても、**オフセットは変わりません**。  
ストリームのオフセットは、**DML トランザクションでストリームを消費した場合のみ進みます**。  
これには、`CREATE TABLE AS SELECT`（CTAS）や `COPY INTO` も含まれます。:contentReference[oaicite:11]{index=11}

【補足】  
単に SELECT でストリームを参照しただけでは、オフセットは進まない点が重要です。

---

## ストリームカラム（Stream Columns）

ストリームはソースオブジェクトの列やデータ自体を保存しませんが、**以下の追加列付きで変更データを返します**：

- `METADATA$ACTION`: 実行された DML 操作（INSERT／DELETE）を示す  
- `METADATA$ISUPDATE`: UPDATE 操作の一部かを示す（TRUE/FALSE）  
- `METADATA$ROW_ID`: 変更された行の一意で不変の ID（履歴追跡用）  

ストリームは **2 つのオフセット間の差分**を記録します。:contentReference[oaicite:12]{index=12}

---

## ストリームのタイプ（Types of Streams）

Snowflake では、用途に応じて次のストリームタイプが利用可能です：:contentReference[oaicite:13]{index=13}

- **Standard（標準）**: 全ての DML 変更（INSERT／UPDATE／DELETE／TRUNCATE）を追跡  
- **Append-only**: 行の挿入のみを追跡  
- **Insert-only**: 外部テーブル等で挿入のみを追跡（DELETE を追わない）

【補足】  
用途に応じて変更検知の粒度を選べる点が柔軟性を提供します。

---

## データ保持期間と劣化（Data Retention and Staleness）

ストリームは、**ソースオブジェクトのデータ保持期間（Time Travel）を超えると stale（劣化）状態**になります。  
劣化すると、変更履歴や未消費のデータがアクセス不能になり、再作成が必要です。:contentReference[oaicite:14]{index=14}

未消費データを定期的に消費することで、**ストリームの stale リスクを防げます**。  
また、`SYSTEM$STREAM_HAS_DATA` を使うと、ストリームが空かどうかを判定できます。:contentReference[oaicite:15]{index=15}

---

## 複数消費者（Multiple Consumers）

1 つのストリームに対して複数のコンシューマー（例：タスク・スクリプト）がある場合、  
ストリームのオフセットは **DML で消費した時点で進む**ため、同じデータを複数回消費したい場合は、**別々のストリームを作成**する必要があります。:contentReference[oaicite:16]{index=16}

---

## ビュー上のストリーム（Streams on Views）

ビューにもストリームを作成できますが、**基になるテーブルの変更追跡を有効化する必要あり**です。  
また、マテリアライズドビュー上ではストリームはサポートされません。:contentReference[oaicite:17]{index=17}

---

# 3. 注意点・制約（明記がある場合）

- ストリーム自体は **データを直接持たず、オフセットと変更履歴を返します**。:contentReference[oaicite:18]{index=18}  
- **単なる SELECT ではオフセットは進まない**ため、DML 内で消費する必要があります。:contentReference[oaicite:19]{index=19}  
- ソースオブジェクトの **スキーマ互換性のない変更**がオフセットと進行点の間にあると、クエリが失敗する可能性があります。:contentReference[oaicite:20]{index=20}

---

**ドキュメント上で注意点・制約の明記あり**
