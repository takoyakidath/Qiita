---
title: BitLockerでロックされたディスクにUbuntuをインストールする方法（回復モードから解除）
tags:
  - Windows
  - Ubuntu
  - BitLocker
  - diskpart
private: false
updated_at: '2025-04-12T17:58:52+09:00'
id: 4bea8572e4b2ad87adfa
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

Ubuntuをインストールしようとしたら、WindowsのBitLockerでディスクがロックされていてインストールできない…。  
そんなときに実際に行った手順を簡単にメモしておきます。


## 背景

WindowsがプリインストールされたPCにUbuntuを入れようとしたところ、インストーラーが「ディスクにアクセスできない」とエラー。  
調べてみたら、BitLockerが原因でディスクが保護されている状態でした。


## 手順

1. **Windowsの回復モードに入る**  
   BitLockerの画面→Esc→スキップ→トラブルシューティング→詳細→コマンドプロンp
2. **`diskpart` でディスクを初期化する**

```bash
diskpart
list disk
select disk {対象のディスク番号}
clean
exit
```

> ※対象のディスクは間違えないように注意！全データが消えます。

3. **UbuntuのインストールUSBから再起動してインストールするだけ！**


## おわりに

BitLockerの回復キーがわからなくても、対象のディスクを初期化してしまえばUbuntuは問題なくインストールできます。  
ただし、**全データが消えるので注意**。バックアップを忘れずに！

