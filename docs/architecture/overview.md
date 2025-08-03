# アーキテクチャ概要

## システム構成図

```mermaid
graph TB
    %% UI層
    subgraph "UI Layer"
        Slack[Slack Workspace]
        WebUI[Web Interface]
        VoiceUI[Voice Interface]
    end

    %% データ収集層
    subgraph "Data Collection Layer"
        subgraph "Devices"
            Android[Android Device]
            PC[PC Activity]
            Wearable[Galaxy Fit]
            Camera[Posture Camera]
            Scale[Smart Scale]
            UPRIGHT[UPRIGHT GO 2]
        end
        
        subgraph "IoT Devices"
            Alarm[Smart Alarm]
            AC[Air Conditioner]
            Light[Smart Light]
            Speaker[Smart Speaker]
            Sensors[Environment Sensors]
        end
    end

    %% 外部サービス層
    subgraph "External Services"
        Jira[Jira]
        GitHub[GitHub]
        GCal[Google Calendar]
        Notion[Notion]
        Chrome[Chrome History]
        XGrok[X/Grok]
    end

    %% API Gateway層
    subgraph "API Gateway Layer"
        Gateway[API Gateway / MCP]
        AuthManager[Auth Manager]
        RateLimiter[Rate Limiter]
    end

    %% コア処理層
    subgraph "Core Processing Layer"
        EventBus[Event Bus]
        TaskManager[Task Manager]
        WorkflowEngine[Workflow Engine]
        HealthManager[Health Manager]
        IoTController[IoT Controller]
        MonitoringService[Activity Monitor]
    end

    %% AI/ML層
    subgraph "AI/ML Layer"
        ChatGPT[ChatGPT/Claude]
        VLM[Vision Language Model]
        TaskDecomposer[Task Decomposer AI]
        HealthAdvisor[Health Advisor AI]
        BehaviorAnalyzer[Behavior Analyzer]
    end

    %% データ層
    subgraph "Data Layer"
        TimeSeries[(Time Series DB)]
        TaskDB[(Task DB)]
        UserProfile[(User Profile)]
        HealthData[(Health Data)]
        ActivityLogs[(Activity Logs)]
    end

    %% 接続関係
    Slack --> Gateway
    WebUI --> Gateway
    VoiceUI --> Gateway
    
    Android --> Gateway
    PC --> Gateway
    Wearable --> Gateway
    Camera --> Gateway
    Scale --> Gateway
    UPRIGHT --> Gateway
    
    Alarm --> IoTController
    AC --> IoTController
    Light --> IoTController
    Speaker --> IoTController
    Sensors --> IoTController
    
    Gateway --> AuthManager
    Gateway --> RateLimiter
    AuthManager --> EventBus
    RateLimiter --> EventBus
    
    EventBus --> TaskManager
    EventBus --> WorkflowEngine
    EventBus --> HealthManager
    EventBus --> IoTController
    EventBus --> MonitoringService
    
    TaskManager --> TaskDecomposer
    TaskManager --> ChatGPT
    TaskManager --> TaskDB
    TaskManager --> Jira
    TaskManager --> GitHub
    
    WorkflowEngine --> GCal
    WorkflowEngine --> TaskDB
    WorkflowEngine --> UserProfile
    
    HealthManager --> VLM
    HealthManager --> HealthAdvisor
    HealthManager --> HealthData
    
    MonitoringService --> ActivityLogs
    MonitoringService --> BehaviorAnalyzer
    MonitoringService --> Chrome
    
    BehaviorAnalyzer --> XGrok
    BehaviorAnalyzer --> Notion
    
    IoTController --> GCal
    IoTController --> TimeSeries
```

## アーキテクチャ設計の指針

### 1. マイクロサービス・アーキテクチャ
- 各機能を独立したサービスとして実装
- サービス間はEvent Bus経由で疎結合に連携
- 段階的な機能追加が容易

### 2. データ収集の一元化
- すべてのデバイス・サービスからのデータはAPI Gateway経由
- 認証・レート制限を統一的に管理
- MCP (Model Context Protocol) を活用した標準化

### 3. AI/ML層の抽象化
- 複数のAIモデルを用途別に使い分け
- モデルの入れ替えが容易な設計
- プロンプトやモデル設定の一元管理

### 4. データストレージの最適化
- 時系列データ専用DB（センサーデータ、活動ログ）
- リレーショナルDB（タスク、ユーザー情報）
- ドキュメントDB（ヘルスケアデータ、分析結果）

### 5. セキュリティ・プライバシー考慮
- 読み取り専用権限の原則
- 個人情報の暗号化
- プライベートサーバーへの選択的データ送信

## 実装技術スタック

### バックエンド
- **言語**: Python (FastAPI) / Node.js (Express)
- **メッセージング**: Redis Pub/Sub, Apache Kafka
- **API Gateway**: Kong, Traefik
- **コンテナ**: Docker, Kubernetes

### データストレージ
- **時系列DB**: InfluxDB, TimescaleDB
- **リレーショナルDB**: PostgreSQL
- **ドキュメントDB**: MongoDB
- **キャッシュ**: Redis

### AI/ML
- **フレームワーク**: LangChain, LlamaIndex
- **モデル管理**: MLflow
- **ベクトルDB**: Pinecone, Weaviate

### モニタリング・可観測性
- **メトリクス**: Prometheus + Grafana
- **ログ**: ELK Stack
- **トレーシング**: OpenTelemetry

### フロントエンド
- **Web UI**: React/Next.js
- **Slack App**: Bolt for Python/JavaScript
- **音声**: Web Speech API, Google Assistant SDK

## 要件と優先順位

### Phase 1: 基盤構築（優先度：高）

#### 1. タスク自動分解・スケジューリング機能
- **目的**: Jiraから大きなタスクを取得し、30分単位の実行可能なタスクに分解
- **実装内容**:
  - Jira APIとの完全統合
  - 対話的なタスク分解（段階的実装）:
    - Phase 1a: 専用Webインターフェースでの実装
    - Phase 1b: Slack統合への移行
  - コーディングタスクの設計/実装フェーズ分離
  - 分解されたタスクのJiraへの自動登録

#### 2. AIレビュー・品質管理システム
- **目的**: タスク完了基準の自動チェックと人間レビューの効率化
- **実装内容**:
  - タスクの受け入れ条件の自動検証
  - AIによる初期レビュー → 人間による最終確認フロー
  - 読み取り専用権限でのAI実行環境

#### 3. Slack統合チャットインターフェース
- **目的**: ChatGPTとのやり取りをSlack上で完結させ、履歴管理を改善
- **実装内容**:
  - Slack Block Kitを活用したリッチUI（表のコピーボタン等）
  - チャット履歴の管理・複製機能
  - Jira等とのシームレス連携
  - **Slack入力の自動タスク化**: メッセージからタスクを自動抽出・登録

#### 4. 日次タスク管理・繰り越しシステム
- **目的**: 未完了タスクの自動繰り越しと翌日計画の最適化
- **実装内容**:
  - 残タスクの自動次日繰り越し
  - バッファ時間を考慮した計画調整
  - 仕事（Jira）の行動結果の自動収集
  - プライベートサーバーへの活動ログ送信

### Phase 2: 拡張機能（優先度：中）

#### 5. 活動モニタリング統合システム
- **目的**: 多様なデバイスからタスク取り組み状況を自動追跡
- **実装内容**:
  - Android（アプリ使用状況、位置情報）
  - PC活動ログ
  - ウェアラブル端末（Galaxy Fit）データ統合
  - ポモドーロタイマー連携
  - 体重計・環境センサーデータ収集
  - 姿勢監視（UPRIGHT GO 2、カメラシステム）

#### 6. 振り返り自動化
- **目的**: Jira/カレンダーから情報を収集し、定期的な振り返りを支援
- **実装内容**:
  - 実績データの自動収集
  - Notion APIを使った定期レポート作成
  - 改善提案の自動生成

#### 7. IoT連携・自動化システム
- **目的**: Googleカレンダーと連動した生活環境の自動最適化
- **実装内容**:
  - アラーム自動設定
  - エアコン・照明の自動制御
  - スピーカーでのリマインダー
  - 体重計の自動記録

#### 8. GitHub対話型レビュー
- **目的**: PRレビューをAIとの対話形式で効率化
- **実装内容**:
  - GitHub APIとの連携
  - コメント/提案の対話的な生成
  - レビュー履歴の管理

#### 9. 長期目標管理システム
- **目的**: 3年単位の長期ロードマップと日々のタスクを連携
- **実装内容**:
  - JPD（Jira Product Discovery）連携
  - Epic/Story/Taskの階層管理
  - 四半期・半期単位でのマイルストーン追跡

### Phase 3: 高度な機能（優先度：低）

#### 10. ヘルスケア統合管理
- **目的**: 食事・薬・健康データの一元管理
- **実装内容**:
  - 食事写真からのVLM栄養分析
  - 服薬リマインダー・記録
  - 健康データの統合ダッシュボード
  - AIによる健康アドバイス

#### 11. 行動分析・改善提案
- **目的**: ブラウザ履歴等から行動パターンを分析し、改善を提案
- **実装内容**:
  - Chrome履歴の収集（暗号化対応）
  - X(Twitter)リンクのGrok連携
  - 新サービスアイデアの自動生成

#### 12. Notion MCP統合
- **目的**: Notionに蓄積された知識をAIが活用
- **実装内容**:
  - ドキュメント・要件定義書の参照
  - アイデアごとのリポジトリ管理
  - 資料のバージョン管理

#### 13. 統合ボットインターフェース
- **目的**: 全情報への単一アクセスポイント提供
- **実装内容**:
  - 全システムデータへの統合アクセス
  - コンテキストを考慮した相談機能
  - マルチモーダル対話（音声・テキスト・画像）

#### 14. 対話型UI最適化
- **目的**: ユーザーエクスペリエンスの向上
- **実装内容**:
  - パーソナライズされた対話インターフェース
  - o3等の高度なAIモデルとの履歴複製
  - カスタマイズ可能なアバター/キャラクター
