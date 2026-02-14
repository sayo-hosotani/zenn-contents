---
title: "Claude CodeをAGENTS.mdに対応させてCLAUDE.mdを削除する"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Claude Code"]
published: true
published_at: 2026-02-15 00:00
---

# Claude CodeにもAGENTS.mdを読んでほしい！

Claude CodeとCodexを併用するときは、CLAUDE.mdとAGENTS.mdを同期するのが面倒です。
CLAUDE.mdに一言、@AGENTS.mdと書けばいいだけなのですが、いまいち美しくない。

そこで、hooksを使って、Claude CodeにAGENTS.mdを自動で読んでもらうことにしました。

## どうすればいいのか？

CLAUDE.mdはいつ読み込まれるのでしょうか？
[Claude Code Docs : Claude Codeを拡張する](https://code.claude.com/docs/ja/features-overview)には、下記の記載があります。

> 機能がどのように読み込まれるかを理解する
>
> 各機能はセッションの異なるポイントで読み込まれます。以下のタブは、各機能がいつ読み込まれるか、およびコンテキストに何が入るかを説明しています。
>
> CLAUDE.md
> 時期： セッション開始
> 読み込まれるもの： すべての CLAUDE.md ファイルの完全なコンテンツ（管理、ユーザー、プロジェクトレベル）。
> 継承： Claude は作業ディレクトリからルートまで CLAUDE.md ファイルを読み取り、それらのファイルにアクセスするときにサブディレクトリ内のネストされたものを検出します。詳細は Claude がメモリを検索する方法 を参照してください。

セッション開始時に、常に読み込んでいることが分かります。

[Claude Code Docs : Hooksリファレンス](https://code.claude.com/docs/ja/hooks)には、下記の記載があります。

> SessionStart
> Claude Codeが新しいセッションを開始するか、既存のセッションを再開するときに実行されます（現在、内部的には新しいセッションを開始します）。既存の問題や最近のコードベースの変更などの開発コンテキストを読み込んだり、依存関係をインストールしたり、環境変数をセットアップしたりするのに便利です。


「既存のセッションを再開するときに実行されます（現在、内部的には新しいセッションを開始します）。」の記載が気になります。
内部的に新しいセッションを開始しているということは、resumeでセッション再開するときにも、CLAUDE.mdを読み直しているということなのでしょうか？この記載だけではちょっとわかりません。

まあとにかく、SessionStartのHooksを設定すればよさそうです。

## もともとの挙動を確認しておく

### AGENTS.mdとCLAUDE.mdの両方があるとき

まずは、状態確認から。
こんなフォルダを作成します。

```
.
├── AGENTS.md
└── CLAUDE.md
```

.claudeと.codexもない状態です。
mdの内容はこんな感じ。

```:CLAUDE.md
起動したら「claude.md!!!」と言ってください。
```

```:AGENTS.md
起動したら「agents.md!!!」と言ってください。
```

それぞれのエージェントを呼び出します。

![Claude Codeがclaude.md!!!と答えている](/images/20260215-claude-hooks-agents-md/1.claude-before.png)
![Codexがagents.md!!!と答えている](/images/20260215-claude-hooks-agents-md/2.codex-before.png)

Claude CodeがCLAUDE.mdを、CodexがAGENTS.mdを読み込んでいることが分かります。

### AGENTS.mdだけがあるとき

AGENTS.mdを残したまま、CLAUDE.mdを削除してみましょう。

![Claude Codeが何と答えていいかわからず、ファイルを探索している](/images/20260215-claude-hooks-agents-md/3.claude-remove-claude-md.png)

claudeがAGENTS.mdを読めておらず、プロジェクトファイルを確認しに行っていることが分かります。

### CLAUDE.mdだけがあるとき

ちなみに、AGENTS.mdを削除してCLAUDE.mdを残すと、こうなります。

![Codexが何と答えていいかわからず、適当に答えている](/images/20260215-claude-hooks-agents-md/4.codex-remove-agents-md.png)

当たり前ですが、codexはCLAUDE.mdを読みませんね。

## Hooksを追加してみる

では、hooksを実装します。

```json:.claude/settings.json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "cat AGENTS.md"
          }
        ]
      }
    ]
  }
}
```

SessionStartでは、startup（起動時のセッション開始）、resume（セッション再開時）、clear（セッションクリア契機のセッション開始）、compact（コンパクト実行時）の4種類のmatcher（実行契機）を指摘できます。
resumeの場合はセッション開始時の読み込み内容がコンテキストに残っていると思うので、ロード契機から除外しました。
compact後はAGENTS.mdの内容を確実に拾ってほしいので、ロードすることにしました。

## 読み込めたことを確認する

では、Claude Codeの動作確認です。

![Claude Codeがagents.md!!!と答えている](/images/20260215-claude-hooks-agents-md/5.claude-after.png)

AGENTS.mdが自動的に読み込まれました。
これで、CLAUDE.mdを削除できます。


# 参考情報
[Claude Code Docs : Claude Code フックの使い始め](https://code.claude.com/docs/ja/hooks-guide)
[Claude Code Docs : Hooksリファレンス](https://code.claude.com/docs/ja/hooks)

