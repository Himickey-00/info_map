# InfoMap システムアーキテクチャ

## 全体構成図

```mermaid
flowchart TB
    subgraph Layer1["🔴 層① 争点・視点の教師なし分類モデル"]
        direction TB

        subgraph TopicExtraction["(a) 争点抽出"]
            A1[ニュース記事] --> A2[SentenceBERT<br/>埋め込み生成]
            A2 --> A3[Parametric UMAP<br/>次元削減]
            A3 --> A4[HDBSCAN<br/>クラスタリング]
            A4 --> A5[c-TF-IDF<br/>代表語抽出]
            A5 --> A6[ナレッジグラフ照合]
            A6 --> A7[トピックラベル + 2D座標]
        end

        subgraph ViewpointExtraction["(b) 視点抽出"]
            B1[同一争点内<br/>記事ペア] --> B2[対照学習<br/>InfoNCE Loss]
            B2 --> B3[視点ベクトル]
            B3 --> B4[ナレッジグラフ<br/>バイアス中和]
            B4 --> B5[視点ラベル]
        end
    end

    subgraph Layer2["🔵 層② リアルタイム可視化エンジン"]
        direction LR
        C1[新規記事] --> C2[API Endpoint]
        C2 --> C3[Parametric UMAP<br/>固定座標射影]
        C3 --> C4[分類処理]
        C4 --> C5[(記事DB)]
        C5 --> C6[D3.js<br/>可視化]
        C6 --> C7[情報空間マップ]
    end

    subgraph Layer3["🟢 層③ Chrome Extension"]
        direction TB
        D1[マップ埋め込み<br/>現在地表示]
        D2[閲覧ログ蓄積<br/>GS-score算出]
        D3[未閲覧記事<br/>レコメンド]
        D4[ダッシュボード]

        D1 --> D4
        D2 --> D4
        D3 --> D4
    end

    Layer1 --> Layer2
    Layer2 --> Layer3
```

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| 層① 分類モデル | Python, BERTopic, SentenceBERT, Parametric UMAP, HDBSCAN, PyTorch |
| 層② 可視化エンジン | FastAPI/Flask, D3.js, PostgreSQL/Redis |
| 層③ Chrome拡張 | TypeScript, Chrome Extension API, D3.js, IndexedDB |

## データフロー

```mermaid
sequenceDiagram
    participant User as ユーザー
    participant Ext as Chrome拡張
    participant API as APIサーバー
    participant ML as 分類モデル
    participant KG as ナレッジグラフ
    participant DB as 記事DB

    User->>Ext: ニュースサイト閲覧
    Ext->>API: 記事URL/コンテンツ送信
    API->>ML: 埋め込み生成リクエスト
    ML->>ML: UMAP次元削減
    ML->>ML: HDBSCANクラスタリング
    ML->>KG: 代表語照合
    KG-->>ML: トピックラベル
    ML-->>API: 2D座標 + ラベル
    API->>DB: 記事情報保存
    API-->>Ext: 可視化データ
    Ext->>Ext: D3.jsでマップ描画
    Ext->>Ext: 閲覧ログ記録
    Ext->>Ext: 多様性スコア算出
    Ext-->>User: 情報空間マップ表示
    Ext-->>User: レコメンド表示
```

## 争点・視点抽出の詳細

```mermaid
flowchart LR
    subgraph Input
        I1[記事タイトル]
        I2[記事本文/要約]
    end

    subgraph Embedding
        E1[SentenceBERT]
    end

    subgraph DimensionReduction
        D1[Parametric UMAP<br/>固定座標空間]
    end

    subgraph Clustering
        C1[HDBSCAN<br/>トピック数自動決定]
    end

    subgraph Labeling
        L1[c-TF-IDF<br/>代表語抽出]
        L2[ナレッジグラフ<br/>上位概念取得]
        L3[コサイン類似度<br/>ラベルマッチング]
    end

    subgraph Output
        O1[トピックラベル]
        O2[2D座標 x,y]
        O3[視点ベクトル]
    end

    I1 --> E1
    I2 --> E1
    E1 --> D1
    D1 --> C1
    C1 --> L1
    L1 --> L2
    L2 --> L3
    L3 --> O1
    D1 --> O2
    C1 --> O3
```
