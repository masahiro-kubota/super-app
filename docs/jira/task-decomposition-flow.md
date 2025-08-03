# Jiraタスク分解フロー

## Slackからのタスク登録・分解の詳細フロー

```mermaid
sequenceDiagram
    participant User as ユーザー
    participant Slack as Slack
    participant Gateway as API Gateway
    participant Agent as AI Agent
    participant DataAPI as Data API
    participant Jira as Jira Cloud

    User->>Slack: タスク依頼メッセージ
    Slack->>Gateway: イベント通知
    Gateway->>Agent: メッセージ転送
    
    %% コンテキスト収集フェーズ
    Agent->>DataAPI: 関連情報クエリ
    Agent->>Jira: 既存タスク確認
    DataAPI-->>Agent: ユーザー設定・履歴
    Jira-->>Agent: 既存タスク情報
    
    %% 文章整理・確認フェーズ
    Note over Agent: タスク内容を内部で整理
    Agent->>Slack: 確認メッセージ送信
    Slack->>User: 「以下の内容でよろしいですか？」
    User->>Slack: 確認/修正
    
    %% Jira登録フェーズ
    alt ユーザーが承認
        Agent->>Jira: タスク（Epic/Story）作成
        Jira-->>Agent: タスクID・URL
    else ユーザーが修正
        Note over Agent: 修正内容で再整理
        Note over Agent: 確認プロセスを繰り返し
    end
    
    %% タスク分解フェーズ
    Note over Agent: タスク分解ロジック実行
    Agent->>Slack: 分解案の提示
    Slack->>User: 「以下のサブタスクでよろしいですか？」
    
    alt ユーザーが承認
        Agent->>Jira: サブタスク一括作成
        Agent->>Slack: 完了通知とJiraリンク
    else ユーザーが修正
        User->>Slack: 修正指示
        Note over Agent: 再分解実行
    end
```

### フローの特徴

1. **対話的確認プロセス**
   - AIが整理した内容を必ずユーザーに確認
   - 修正が必要な場合は繰り返し対話

2. **段階的処理**
   - まずメインタスクをJiraに登録
   - その後、サブタスクに分解
   - 各段階でユーザー確認を実施

3. **柔軟な修正対応**
   - どの段階でもユーザーが介入可能
   - AIへのフィードバックループ