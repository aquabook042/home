# CLAUDE.md

このファイルはプロジェクト固有の指示・規則を記述します。

---

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

## ディレクトリ構成（参考）

```
/
├── ADS/          # ADS ファイル群
├── Sigma/        # Sigma ルール群
└── Signal/       # Signal ファイル群
```
