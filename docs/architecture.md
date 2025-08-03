# アーキテクチャ概要

## システム構成図

```mermaid
graph TB
    %% チャットインターフェース層
    subgraph "Chat Interface Layer"
        ChatInterface[Slack / Web UI]
        ChatGateway[Chat Gateway]
    end

    %% AI Agent層
    subgraph "AI Agent Layer"
        AIAgent[AI Agent]
    end

    %% デバイス・データソース層
    subgraph "Device & Data Source Layer"
        subgraph "Push Devices"
            Android[Android Device]
            PCClient[PC Client]
            SmartScale[Smart Scale]
        end
    
        subgraph "Pull APIs"
            HealthAPI[Samsung Health API]
            UprightAPI[UPRIGHT API]
        end
    
        subgraph "Local Devices"
            Camera[Posture Camera]
            Sensors[Environment Sensors]
        end
    end

    %% IoTデバイス層（双方向）
    subgraph "IoT Devices (Bidirectional)"
        Alarm[Smart Alarm]
        AC[Air Conditioner]
        Light[Smart Light]
        Speaker[Smart Speaker]
    end

    %% 外部サービス層
    subgraph "External Services"
        JiraCloud[Jira Cloud]
        GitHub[GitHub]
        GCal[Google Calendar]
        Notion[Notion]
        Chrome[Chrome History]
        XGrok[X/Grok]
    end


    %% データ収集層
    subgraph "Data Collection Layer"
        DataCollector[Data Collector]
    end
  
    %% 制御層
    subgraph "Control Layer"
        IoTController[IoT Controller]
    end
  
    %% データアクセス層
    subgraph "Data Access Layer"
        DataAPI[Data Query API]
        ControlAPI[Device Control API]
    end


    %% データ層
    subgraph "Data Layer"
        TimeSeries[(Time Series DB)]
        UserProfile[(User Profile)]
    end

    %% ユーザーとAI Agent間の双方向通信
    ChatInterface --> ChatGateway
    ChatGateway --> AIAgent
    AIAgent --> ChatInterface
  
    %% AI AgentからAPIを通じたデータアクセス
    AIAgent --> DataAPI
    AIAgent --> JiraCloud
    AIAgent --> GitHub
    AIAgent --> GCal
    AIAgent --> Notion
    AIAgent --> Chrome
    AIAgent --> XGrok
  
    %% AI Agentからの制御
    AIAgent --> ControlAPI
  
    %% データ収集（AIを介さない自動収集）
    Android --> DataCollector
    PCClient --> DataCollector
    SmartScale --> DataCollector
  
    HealthAPI --> DataCollector
    UprightAPI --> DataCollector
  
    Camera --> DataCollector
    Sensors --> DataCollector
  
    %% CollectorからDBへの直接格納
    DataCollector --> TimeSeries
  
    %% APIからDBへのアクセス
    DataAPI --> TimeSeries
    DataAPI --> UserProfile
  
    %% IoTデバイス制御
    ControlAPI --> IoTController
    IoTController --> Alarm
    IoTController --> AC
    IoTController --> Light
    IoTController --> Speaker
  
    %% IoTデバイスからのデータ収集
    Alarm --> DataCollector
    AC --> DataCollector
    Light --> DataCollector
    Speaker --> DataCollector
```

## アーキテクチャ設計の指針

### 1. AI Agent中心設計

- **ユーザー指示はSlack経由（本番）またはWeb UI（開発用）**でAI Agentが受信
- AI Agent（Gemini CLI/Claude Code）が自律的に判断・実行
- AI Agent内部でコンテキスト収集・アクション決定・実行を一元管理
- データ収集は決められたルールで自動実行（AI介在なし）

### 2. タスク管理の一元化

- **すべてのタスクはJira Cloudで管理**（Task DBは使用しない）
- AI AgentがJira APIを直接操作
- タスクの作成、更新、分解はすべてJira上で実行

### 3. IoTデバイスの双方向制御

- データ収集だけでなく、**AI Agentからの制御も可能**
- カレンダー連携による自動制御
- ユーザーの行動パターンに基づく最適化

### 4. データ収集とアクセスの分離

- **Collector層**：デバイスからのデータを自動収集・格納（AIは介在しない）
  - Push型：認証が必要なデバイスからの能動的送信
  - Pull型：外部APIからの定期取得
  - ローカル型：Gateway不要のローカルネットワーク内収集
- **Data API層**：AI Agentがクエリを投げてデータを取得
  - 時系列データ、ヘルスデータ、行動ログへの統一的アクセス
  - AI Agentが必要なデータを自律的に判断して取得

### 5. セキュリティ・プライバシー考慮

- AI Agentには基本的に読み取り専用権限
- 制御が必要な場合のみ書き込み権限を付与
- 個人情報の暗号化とアクセス制御
