# git-flow ガイダンス資料

> **対象読者**: Gitの基本操作（commit / branch / merge）は理解しているが、git-flowを初めて学ぶ方
> **参考元**: [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/) (Vincent Driessen, 2010)

---

## 1. git-flowとは

git-flowは、Gitのブランチをどのように使い分けるかを定めた**ブランチ戦略（ブランチモデル）**です。

「どのブランチを本番にするか」「新機能はどこで開発するか」「緊急バグはどう直すか」といった判断をルール化することで、チーム開発での混乱を防ぎます。

### git-flowが向いているプロジェクト

- バージョン番号（v1.0.0 など）を付けてリリースするソフトウェア
- リリースのタイミングが明確に存在する開発サイクル

> **補足**: 常時デプロイ型のWebサービス（継続的デリバリー）には、より単純な **GitHub Flow** が推奨される場合があります。

---

## 2. ブランチの種類と役割

git-flowでは、ブランチを2種類に分類します。

### 2-1. メインブランチ（常に存在する）

```
main ────────────────────────────────────────── 本番リリース済みのコードのみ
develop ─────────────────────────────────────── 次のリリースに向けた統合ブランチ
```

| ブランチ | 役割 | 直接コミット |
|---------|------|------------|
| `main` | 本番環境に出ているコードのみを管理。タグでバージョンを記録する | **禁止**（マージのみ） |
| `develop` | 開発者が日々の成果を統合する場所。次のリリースに向けて育てていく | **禁止**（マージのみ） |

### 2-2. サポートブランチ（必要なときだけ作成し、用が済んだら削除する）

| ブランチ名 | 分岐元 | マージ先 | 用途 |
|-----------|--------|---------|------|
| `feature/*` | `develop` | `develop` | 新機能の開発 |
| `release/*` | `develop` | `main` と `develop` | リリース準備（バージョン確定・軽微な修正） |
| `hotfix/*` | `main` | `main` と `develop` | 本番環境の緊急バグ修正 |

---

## 3. このリポジトリで実施したシナリオ

以下のストーリーに沿って、git-flowの全ルールを網羅する操作を行いました。

```
【開発したアプリの機能一覧】
  - ログイン機能（v1.0.0でリリース）
  - ダッシュボード機能（v1.0.0でリリース）
  - プロフィール機能（v1.1.0でリリース）
  - 設定機能（v1.1.0でリリース）
  - 緊急hotfix：認証の脆弱性修正（v1.0.1でリリース）
```

---

## 4. コミットメッセージの接頭辞（Conventional Commits）

このリポジトリのコミットメッセージには、変更の種類を示す**接頭辞（プレフィックス）**が付いています。これは [Conventional Commits](https://www.conventionalcommits.org/) と呼ばれる規約に基づいたもので、コミットログを一目で分類できるようにします。

### 書式

```
<接頭辞>: <変更内容の要約>
```

### 接頭辞の一覧

| 接頭辞 | 意味 | 使用例（このリポジトリ） |
|--------|------|----------------------|
| `feat:` | 新機能の追加 | `feat: ログイン機能を追加` |
| `fix:` | バグの修正 | `fix: 認証処理のSQLインジェクション脆弱性を緊急修正` |
| `chore:` | 機能に影響しない作業（バージョン更新・設定変更など） | `chore: バージョンをv1.0.0に設定` |
| `docs:` | ドキュメントのみの変更 | `docs: READMEにセットアップ手順を追記` |
| `style:` | コードの動作に影響しない整形・フォーマット修正 | `style: インデントを統一` |
| `refactor:` | バグ修正でも新機能でもないコードの改善 | `refactor: 認証処理を関数に切り出し` |
| `test:` | テストコードの追加・修正 | `test: ログイン処理の単体テストを追加` |
| `perf:` | パフォーマンス改善 | `perf: クエリのN+1問題を解消` |

### 接頭辞とgit-flowブランチの対応関係

接頭辞を見るだけで、そのコミットがどのブランチで行われたかが推測できます。

| ブランチ | よく使う接頭辞 | 理由 |
|---------|--------------|------|
| `feature/*` | `feat:` | 新機能開発が主な作業 |
| `release/*` | `fix:`, `chore:` | バグ修正とバージョン設定のみ許可 |
| `hotfix/*` | `fix:`, `chore:` | 緊急バグ修正とバージョン更新 |

> **ポイント**: `release/*` や `hotfix/*` のブランチに `feat:` のコミットが現れた場合、git-flowのルール違反のサインです。コミット履歴を見るだけでルール遵守状況を確認できます。

---

## 5. コミットツリーの読み方

```
*   c2aac98 (HEAD -> develop) Merge branch 'release/1.1.0' into develop
|\
| | *   ff71778 (tag: v1.1.0, main) Merge branch 'release/1.1.0'
| | |\
| | |/
| |/|
| * | cc5be5e fix: ダッシュボードのグラフ表示崩れを修正
| * | d53228d chore: バージョンをv1.1.0に設定
|/ /
* |   7236e45 Merge branch 'hotfix/1.0.1' into develop
|\ \
* \ \   b549f1c Merge branch 'feature/settings' into develop
|\ \ \
| * | | 2de4dac feat: 通知設定機能を追加
| * | | caab005 feat: テーマ切り替え機能を追加
|/ / /
* | |   5efa341 Merge branch 'feature/profile' into develop
|\ \ \
| * | | ec79203 feat: アバター画像アップロード機能を追加
| * | | 768e8d8 feat: プロフィール情報表示を追加
|/ / /
* | |   ef707bc Merge branch 'release/1.0.0' into develop
|\ \ \
| | | *   d6b0f5a (tag: v1.0.1) Merge branch 'hotfix/1.0.1'
| | | |\
| | | |/
| | |/|
| | * | 1f547f6 chore: バージョンをv1.0.1に更新
| | * | 67dccbf fix: 認証処理のSQLインジェクション脆弱性を緊急修正
| | |/
| | *   434f245 (tag: v1.0.0) Merge branch 'release/1.0.0'
| | |\
| | |/
| |/|
| * | 3f7d895 fix: セッション管理の軽微なバグを修正
| * | 66c621b chore: バージョンをv1.0.0に設定
|/ /
* |   4970610 Merge branch 'feature/dashboard' into develop
|\ \
| * | 9fe5df0 feat: グラフによるデータ可視化を追加
| * | aa4ece7 feat: ダッシュボードの統計情報表示を追加
|/ /
* |   3a816df Merge branch 'feature/login' into develop
|\ \
| |/
|/|
| * c56ac46 feat: セッション管理を追加
| * 147109a feat: ログイン機能を追加
|/
* c72ff94 first commit
```

### 見方のコツ

- **`*` (アスタリスク)** がコミットを表します
- **`|` や `\` や `/`** がブランチの流れを表します
- **インデントが深い（右にずれている）ライン**がサポートブランチです
- **`tag: v1.x.x`** が付いているコミットがリリースポイントです
- **マージコミット**（`Merge branch '...'`）でブランチの線が合流しています

---

## 6. 各ブランチの操作詳細

### 6-1. feature ブランチ

新機能を1つの単位として独立して開発します。

```bash
# developから分岐して作成
git checkout develop
git checkout -b feature/login

# 開発作業・コミットを繰り返す
git add .
git commit -m "feat: ログイン機能を追加"
git commit -m "feat: セッション管理を追加"

# developへ --no-ff でマージ（★重要）
git checkout develop
git merge --no-ff feature/login -m "Merge branch 'feature/login' into develop"

# ブランチを削除
git branch -d feature/login
```

**なぜ `--no-ff` が必要か？**

```
【--no-ff なし（Fast-forward）】        【--no-ff あり（マージコミットあり）】
develop ────────────────              develop ──────────┐
                    ↑                                  │(マージコミット)
feature  ──●──●──●                   feature  ──●──●──●
（どこがfeatureか分からなくなる）        （featureの塊が明確に残る）
```

`--no-ff` を指定することで、たとえFeature内のコミットが1つでも、必ずマージコミットが作られます。これにより「どの機能がいつ取り込まれたか」を後から追跡できます。

---

### 6-2. release ブランチ

developが「次のリリースに十分な状態」になったらreleaseブランチを切ります。

```bash
# developから分岐して作成
git checkout develop
git checkout -b release/1.0.0

# リリース準備作業のみ行う（バージョン番号確定、軽微なバグ修正）
git commit -m "chore: バージョンをv1.0.0に設定"
git commit -m "fix: セッション管理の軽微なバグを修正"

# ① mainへマージ & タグを付ける
git checkout main
git merge --no-ff release/1.0.0 -m "Merge branch 'release/1.0.0'"
git tag -a v1.0.0 -m "Release version 1.0.0"

# ② developへも必ずマージする（リリースブランチでの修正を反映）
git checkout develop
git merge --no-ff release/1.0.0 -m "Merge branch 'release/1.0.0' into develop"

# ブランチを削除
git branch -d release/1.0.0
```

**releaseブランチのルール**
- 大型の新機能は追加してはいけない（developで次のリリース向けに温める）
- バグ修正・ドキュメント更新・バージョン設定のみ許可
- mainとdevelopの**両方**にマージすることが必須

---

### 6-3. hotfix ブランチ

本番環境（`main`）に緊急の不具合が発生したとき、`develop` の開発を止めずに対応します。

```bash
# mainから分岐して作成（developからではない！）
git checkout main
git checkout -b hotfix/1.0.1

# バグ修正
git commit -m "fix: 認証処理のSQLインジェクション脆弱性を緊急修正"
git commit -m "chore: バージョンをv1.0.1に更新"

# ① mainへマージ & タグを付ける
git checkout main
git merge --no-ff hotfix/1.0.1 -m "Merge branch 'hotfix/1.0.1'"
git tag -a v1.0.1 -m "Release version 1.0.1 - hotfix"

# ② developへも必ずマージする（修正をdevelopにも取り込む）
git checkout develop
git merge --no-ff hotfix/1.0.1 -m "Merge branch 'hotfix/1.0.1' into develop"

# ブランチを削除
git branch -d hotfix/1.0.1
```

**hotfixの重要ポイント**

```
main ────────●────────────── ← hotfixはmainから分岐
             │
hotfix/1.0.1 ●──●
             │   ↘
develop ─────●────●────●──── ← developにも必ずフィードバック
```

このリポジトリでは、`feature/profile` と `feature/settings` の開発が進んでいる最中に `hotfix/1.0.1` を対応しました。hotfixがdevelopの開発に**干渉しない**ことがコミットツリーで確認できます。

---

## 7. タグによるリリース管理

git-flowでは、`main` へのマージのたびに必ずタグを付けます。

```bash
# アノテーションタグの作成（メッセージ付き・推奨）
git tag -a v1.0.0 -m "Release version 1.0.0"

# タグ一覧の確認
git tag -l -n
```

このリポジトリでのタグ:

| タグ | 種別 | 内容 |
|-----|------|------|
| `v1.0.0` | 通常リリース | ログイン・ダッシュボード機能 |
| `v1.0.1` | hotfixリリース | 認証の脆弱性修正 |
| `v1.1.0` | 通常リリース | プロフィール・設定機能追加 |

---

## 8. git-flow 全体フロー図

```
main     ──●──────────────────────────●────────────────●──────────────●──
           │first                    ↑v1.0.0          ↑v1.0.1       ↑v1.1.0
           │commit                   │(release)        │(hotfix)     │(release)
develop  ──●──────────●──────────────●────────●──●────●──────────────●──
               ↑         ↑            ↑   ↑    ↑    ↑         ↑
           feature/  feature/     release/ │  feature/ feature/  release/
           login     dashboard    1.0.0  hotfix/ profile  settings  1.1.0
                                         1.0.1
```

**フローのまとめ:**

1. `develop` から `feature` を切る
2. 機能完成 → `develop` へ `--no-ff` マージ → `feature` 削除
3. 準備完了 → `develop` から `release` を切る
4. `release` でリリース準備 → `main` にマージ → タグ付け → `develop` にもマージ → `release` 削除
5. 本番バグ発生 → `main` から `hotfix` を切る
6. `hotfix` で修正 → `main` にマージ → タグ付け → `develop` にもマージ → `hotfix` 削除

---

## 9. よくある間違いと注意点

### ❌ featureブランチをmainに直接マージする
→ developを経由せずにmainに入ると、未テストの機能が本番に入る危険があります。

### ❌ hotfixをdevelopにマージし忘れる
→ 本番で直したバグが次のリリースで再発します。hotfixは必ずmainとdevelopの**両方**にマージします。

### ❌ releaseブランチで新機能を追加する
→ releaseは安定化のためのブランチです。新機能は次のdevelopサイクルで開発します。

### ❌ mainに直接コミットする
→ mainへの変更は必ずrelease/hotfixブランチを経由します。

### ❌ --no-ffを省略する
→ Fast-forwardマージになりフィーチャーの境界が失われます。後から「どの機能が含まれているか」を追跡しにくくなります。

---

## 10. 補足：GitHubのIssueとの連携

git-flowにGitHubのIssue管理を組み合わせると、「何のためにこのブランチを作ったか」をコード履歴と紐づけて追跡できます。

### 方針

#### ① ブランチ名にIssue番号を含める

ブランチを作成するときに、対応するIssue番号をブランチ名に含めます。

```bash
# Issue #42「ログイン機能の実装」に対応する場合
git checkout -b feature/#42-login

# hotfixの場合も同様
git checkout -b hotfix/#87-fix-auth-vulnerability
```

命名規則：`<ブランチ種別>/#<Issue番号>-<内容>`

ブランチ一覧を見るだけで、どのIssueに対応した作業かが分かります。

#### ② PRの本文に `closes #番号` を記載する

`feature` ブランチを `develop` へマージするときは、Pull Requestを通じて行います。そのPR本文に以下のキーワードを含めることで、**PRがマージされた時点でIssueが自動的にクローズ**されます。

```markdown
## 概要
ログイン機能を実装しました。

closes #42
```

> `closes` の代わりに `fixes` / `resolves` でも同じ効果があります。

### Issue〜マージまでの全体の流れ

```
① Issue #42「ログイン機能の実装」を作成
        ↓
② feature/#42-login ブランチを develop から作成
        ↓
③ 開発・コミット
     feat: ログイン機能を追加
     feat: セッション管理を追加
        ↓
④ develop へ Pull Request を作成
     タイトル: feat: ログイン機能の実装
     本文:     closes #42
        ↓
⑤ PR がマージされる（--no-ff）
        ↓
⑥ Issue #42 が自動クローズ
```

### まとめ

| 対応 | 方法 | 効果 |
|------|------|------|
| ブランチとIssueの紐づけ | ブランチ名に `#番号` を含める | 作業とIssueの関係が一目で分かる |
| PRとIssueの紐づけ | PR本文に `closes #番号` を記載 | マージと同時にIssueが自動クローズされる |

---

## 11. 補足：GitHubのブランチ保護ルール

git-flowのルールをGitHub上で技術的に強制するための設定です。GitHubには設定機能が2種類あり、用途によって使い分けます。

| 機能 | 場所 | 用途 |
|------|------|------|
| **Branch protection rules** | Settings → Branches → Branch protection rules | 特定ブランチの操作（push / merge）を保護する |
| **Rulesets** | Settings → Branches → Rulesets | ブランチの作成自体を命名規則で制限する |

> **適用範囲について**: これらの設定はいずれも**リモートリポジトリ（GitHub）にのみ**適用されます。ローカルでは引き続きどんなブランチ名でも作成できますが、規則に違反したブランチをGitHubへpushしようとした時点で拒否されます。

---

### Branch protection rules：`main` ブランチ（最も厳格に保護）

`main` は本番コードのみを管理するため、直接pushを完全に禁止し、必ずレビューを通す構成にします。

| 設定項目 | 推奨値 | 理由 |
|---------|--------|------|
| **Require a pull request before merging** | ✅ 有効 | 直接pushを禁止。必ずPR経由にする |
| └ Required approvals | `1` 以上 | レビューなしのマージを防ぐ |
| └ Dismiss stale pull request approvals when new commits are pushed | ✅ 有効 | 承認後に差し込まれた変更を見落とさない |
| **Require status checks to pass before merging** | ✅ 有効 | CIが通ったコードのみマージ可能にする |
| └ Require branches to be up to date before merging | ✅ 有効 | マージ前に最新状態との差分を解消させる |
| **Require linear history** | ❌ 無効 | git-flowは `--no-ff` でマージコミットを作るため、有効にすると矛盾する（後述） |
| **Do not allow bypassing the above settings** | ✅ 有効 | 管理者も例外なくルールに従う |
| **Restrict who can push to matching branches** | ✅ 有効 | release・hotfix担当者のみに限定 |

### Branch protection rules：`develop` ブランチ（中程度の保護）

`develop` はfeatureブランチの統合先です。mainよりは緩めつつ、直接pushは禁止します。

| 設定項目 | 推奨値 | 理由 |
|---------|--------|------|
| **Require a pull request before merging** | ✅ 有効 | featureブランチは必ずPR経由でマージ |
| └ Required approvals | `1` 以上 | コードレビューを必須化 |
| **Require status checks to pass before merging** | ✅ 有効 | テスト・Lintが通ることを保証 |
| **Do not allow bypassing the above settings** | ✅ 有効 | 管理者も同様に従う |

### Rulesets：ブランチ命名規則の技術的強制

**Settings → Branches → Rulesets** で以下のように設定することで、規則に違反したブランチ名でのpushをGitHubが拒否します。

```
【Ruleset の設定】
対象 (Target branches): All branches
ルール: Restrict creations（ブランチ作成を制限）
許可するパターン (Bypass list に追加):
  - main
  - develop
  - feature/*
  - release/*
  - hotfix/*
```

上記以外の名前でpushしようとすると、GitHubがエラーで拒否します。ローカルでの作業は制限されないため、開発者は一時的なブランチを自由に作れますが、リモートへの反映段階でルールが適用されます。

### 保護レベルの全体像

```
main        ─── 最厳格：PR必須 / 承認必須 / CI必須 / 直接push完全禁止  ┐
develop     ─── 中程度：PR必須 / 承認必須 / CI必須 / 直接push禁止      ┤ Branch protection rules
                                                                      ┘
feature/*   ─── 命名規則のみ強制（pushの内容は制限なし）               ┐
release/*   ─── 命名規則のみ強制（pushの内容は制限なし）               ┤ Rulesets
hotfix/*    ─── 命名規則のみ強制（pushの内容は制限なし）               ┘
その他       ─── push自体を拒否                                        ┘ Rulesets
```

### `Require linear history` を有効にしてはいけない理由

git-flowは `--no-ff` によるマージコミットを戦略の中核に置いています。`Require linear history`（squash / rebaseのみ許可）を有効にするとマージコミットが作れなくなり、フィーチャーの境界がコミットツリーから消えてしまいます。この設定はGitHub Flow向けであり、git-flowとは相性が悪いため **必ず無効** にします。

---

## 12. よく使うコマンドチートシート

```bash
# ブランチ確認（全ブランチ表示）
git branch -a

# コミットツリーをグラフで確認
git log --all --oneline --graph --decorate

# タグ一覧
git tag -l -n

# 特定バージョンの状態を確認
git checkout v1.0.0
git checkout develop  # 戻るとき
```
