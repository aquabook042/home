# CLAUDE.md - プロジェクト管理ガイド

## リポジトリ概要

The Gentlemen ランサムウェアグループを対象とした脅威インテリジェンス・検出ルールリポジトリ。
ADS (Attack Description Sheet) ファイルをもとに Sigma 検出ルールを作成・管理する。

## ディレクトリ構造

| ディレクトリ | 内容 |
|---|---|
| `001_CTI/` | Cyber Threat Intelligence (CTI) データ |
| `002_AttackFlow/` | Attack Flow 定義ファイル |
| `003_ADS/` | Attack Description Sheet (ADS) YAML ファイル |
| `004_Sigma/` | Sigma 検出ルール (1ルール = 1ファイル) |
| `005_Signal/` | Signal インジケーター |

## 命名規則

### 成果物名

#### ADS（Attack Defense Scenario）
```
A(CTIの取り込み連番)-(AttackFlow上の連番)
```
- 例：`A099-001`

#### Sigma ルール
```
SIG_(紐づくADS名)_(連番2桁)
```
- 例：`SIG_A099-001_01`

#### Signal
```
C(連番4桁)_(戦術名)_(任意記述※内容)
```
- 例：`C0004_Execution_Windows Management Instrumentation`

---

### Issue 名

> **原則：1 ADS = 1 Issue。Sigma と Signal は同じ Issue で管理する。**

#### ADS Issue
```
[ADS-(連番)] (technique名) - (technique名)((アクター名))
```
- 例：`[ADS-012] T1486 - Golang Cross-Platform Ransomware Deployment (The Gentlemen)`

#### Sigma / Signal Issue
```
[C(連番)](戦術名)_(任意記述※内容)
```
- 例：`[C0004]Execution_Windows Management Instrumentation`
- ※ Issue 名は対応する Signal ファイル名と同一にすること。

---

## ADS → Sigma 作成ワークフロー

ADS ファイルから Sigma ルールを作成する際の標準手順。

### Step 1: ADS ファイルを読む

`003_ADS/` から対象 ADS YAML を読み込み、以下を抽出する：

- `id`: ADS-ID (例: ADS-012)
- `actor`: 脅威アクター名
- `mitre_technique.id` / `mitre_technique.name`: MITRE技術ID・名称
- `phase`: Kill chain フェーズ
- `detection.expected_events`: 検出対象イベント
- `detection.log_sources`: ログソース
- `ioc`: IoC (ハッシュ、IP、ファイル名)
- `references`: 参照URL

### Step 2: Sigma ルールを個別ファイルで作成

**原則: 1つの検出観点 = 1ファイル**

典型的な検出観点（ADS の内容に応じて判断する）：
- 固有ファイル拡張子・ファイル名の検出
- 身代金ノート・マルウェア設定ファイルの作成
- 異常パス (NETLOGON, Temp等) からの実行
- サービスインストール
- 大量ファイル操作・振る舞い検出
- ネットワーク接続・通信パターン

保存先: `004_Sigma/sigma-{ADS-ID}-{NN}_{actor}_{short-description}.yml`

### Step 3: Sigma ルールの構造

各 Sigma ファイルの必須フィールド：

```yaml
title: "{Actor} - {Detection Target}"
id: "a1b2c3d4-{4-digit-ADS-num}-{4-digit-rule-num}-{4-digit-ADS-num}-{12-zeros-with-ADS-num}"
  # 例 ADS-012 の1番目: a1b2c3d4-0012-0001-0012-000000000012
status: experimental
description: |
  何を検出するか、なぜこれが悪意あるものかの説明。
  ADS ID と MITRE 技術IDを明記する。
references:
  - https://attack.mitre.org/techniques/{TXXXX}
  - {ADS の references URL を含める}
author: "Claude (ADS-{NNN} auto-generated)"
date: {作成日: YYYY-MM-DD}
tags:
  - attack.{phase}        # 例: attack.impact
  - attack.{technique}    # 例: attack.t1486
logsource:
  product: windows
  category: file_event    # process_creation / file_event / network_connection 等
  # または service: system / security (Windows イベントログの場合)
detection:
  selection:
    {フィールド名}|{modifier}:
      - {値1}
      - {値2}
  condition: selection
falsepositives:
  - {想定される誤検知シナリオ}
level: critical           # critical / high / medium / low
```

### レベル基準

| レベル | 基準 |
|---|---|
| `critical` | ユニークな IoC (固有のファイル拡張子、身代金ノートファイル名) |
| `high` | 強力な指標 (異常パスからの実行、不審なサービスインストール) |
| `medium` | 疑わしいが誤検知が発生しうる振る舞い |
| `low` | ベースライン収集・参考情報 |

### Step 4: コミット & PR

コミットメッセージ形式：
```
feat: Add individual Sigma rules for {ADS-ID} {Actor} ({TXXXX})
```

PR は `auto-merge-claude-pr.yml` により自動マージされる (ブランチ名が `claude/` で始まる場合)。

### Step 5: サブ Issue 作成

マージ完了後、各 Sigma ファイルに対応する GitHub Issue を作成し、親 ADS Issue のサブ Issue として登録する。

**Issue タイトル**: `[{ADS-ID}-{NN}] {Rule Title}`

**Issue 本文**:
```markdown
## {Rule Title}

**Rule ID**: {ADS-ID}-{NN}
**Level**: {critical/high/medium/low}
**MITRE Technique**: {TXXXX} - {Technique Name}
**Log Source**: {product} / {category or service}

### 説明
{Sigma ルールの description}

### ファイル
`004_Sigma/sigma-{ADS-ID}-{NN}_{actor}_{description}.yml`

### タスク
- [ ] Sigma ルールレビュー
- [ ] SIEM へのデプロイ
- [ ] チューニング & FP 評価
- [ ] テストケース作成

### 関連
- 親 Issue: #{parent-issue-number}
- ADS ファイル: `003_ADS/{ADS-filename}.yaml`
```

サブ Issue 登録コマンド：
```bash
gh api repos/{owner}/{repo}/issues/{parent-id}/sub_issues \
  -X POST -f sub_issue_id={child-id}
```
### Step 6: 作業用ブランチの削除

Step 4にてコミット/マージに使用した作業用ブランチを削除する。

---

## ADS Issue の標準フォーマット

ADS に対応する GitHub Issue（`.github/ISSUE_TEMPLATE/ads-issue.md` 参照）：

```markdown
## {ADS-ID}: {TXXXX} - {タイトル}

**Actor:** {Actor}
**MITRE Technique:** {TXXXX} - {Technique Name}
**Phase:** {Kill Chain Phase}

### 概要
{ADS description の要約}

### ADS ファイル
`003_ADS/{ADS-filename}.yaml`

### 関連
- Actor: {Actor}
- Branch: main
```

---

## @claude へのコマンド例

Issue コメントで以下のように依頼する：

※後日定義
