---
name: publish
description: ストック管理アプリ(index.html)の修正を検証してGitHubに公開し、反映まで確認する。「公開して」「デプロイして」「GitHubに反映して」というとき。
---

# アプリを公開する手順

必ずこの順番どおりに実行する。プロジェクトの CLAUDE.md のルールが前提。

## 1. 変更内容の確認

- `git status` と `git diff --stat` で変更を確認し、何が変わるかをユーザーに1〜2行で伝える
- 変更が何もなければ「公開する変更がない」と伝えて終了

## 2. 公開前チェック（機械チェック）

JavaScriptの構文エラーを確認する。エラーが出たら**公開を中止**して修正する:

```
node -e "const fs=require('fs');const html=fs.readFileSync('index.html','utf8');(html.match(/<script>([\s\S]*?)<\/script>/g)||[]).forEach((s,i)=>{const c=s.replace(/<\/?script>/g,'');try{new Function(c);console.log('script #'+i+': OK')}catch(e){console.log('script #'+i+': エラー '+e.message);process.exit(1)}})"
```

## 3. 公開前チェック（人の目が必要な場合のみ）

以下に当てはまる変更のときだけ、公開前に「ローカルでブラウザ動作確認したか」をユーザーに確認する:

- localStorageに保存するデータ構造（キー名・プロパティ名）を変えた
- 判定ロジック・ポイント計算を変えた

文言修正・表示調整などの軽微な変更は、確認なしでそのまま進めてよい。

## 4. コミットと公開

1. 変更内容が分かる日本語メッセージで `git commit`（末尾に `Co-Authored-By: Claude <noreply@anthropic.com>` を付ける）
2. `git push`
3. pushが拒否されたら: `git fetch origin` → リモートの新規コミットの内容を `git diff` で確認（手動「Upload files」の可能性が高い）→ 内容が同一または古いだけならローカルを正としてマージ → 再push。**リモートにローカルにない修正が含まれていた場合は、force pushせずユーザーに相談する**

## 5. 反映確認と報告

1. `gh api repos/misaki821/kaimono/pages/builds/latest --jq '{status: .status, commit: .commit}'` でビルド状態を確認。`built` になるまで待つ（30秒間隔・最大5回。それでも `building` のままなら「まもなく反映される」と伝えてよい）
2. ユーザーに報告する内容:
   - 公開URL: https://misaki821.github.io/kaimono/
   - 反映を確認するには **Ctrl+F5**（キャッシュ無視の再読み込み）が必要なこと
   - 変更した機能を実際に1回操作して確認してほしいこと
