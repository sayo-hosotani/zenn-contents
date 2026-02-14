---
title: "WSLでオートマウントをOFFにするとVSCodeでリモートログインできなくなる"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["WSL", "VS Code"]
published: true
published_at: 2025-12-30 00:00
---

# WSLでホスト側のフォルダ参照を制限するとVSCodeでリモート接続できなくなる

WSL2のUbuntuに外部からSSH接続するにあたって、接続ユーザーが触れるフォルダを制限したかった。
WSLではデフォルトでWindows側のドライブがマウントされるので、Windows側のフォルダをWSL側ですべて触れてしまう。
WSL2の設定で、下記のようにWindowsフォルダのオートマウントをOFFにすればよいのだが、そうすると今度はVSCodeのリモート接続が通らなくなる。

```conf:wsl.conf
[automount]
enabled = false # オートマウントOFF
[interop]
appendWindowsPath = false # Windows側の環境変数を展開しない
```

# VSCodeの拡張機能のフォルダだけは参照権限が必要である

下記の設定で対応できる。

```conf:/etc/fstab
C:\\Users\\<Windows側のユーザー名>\\.vscode\\extensions\\ms-vscode-remote.remote-wsl-0.104.3 /mnt/c/Users/<Windows側のユーザー名>/.vscode/extensions/ms-vscode-remote.remote-wsl-0.104.3 drvfs metadata,notime,gid=1000,uid=1000,defaults 0 0
```

(この記事は、以前スクラップにしていた内容を移動したものです)

