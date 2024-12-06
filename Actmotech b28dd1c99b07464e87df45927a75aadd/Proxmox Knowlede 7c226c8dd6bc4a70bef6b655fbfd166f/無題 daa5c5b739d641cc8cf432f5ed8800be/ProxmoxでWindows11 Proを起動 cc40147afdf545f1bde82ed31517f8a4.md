# ProxmoxでWindows11 Proを起動

![unnamed.png](Proxmox%E3%81%A6%E3%82%99VM%E4%BD%9C%E6%88%90%E5%BE%8C%E3%81%AEAgent%E5%B0%8E%E5%85%A5%2007556cbc1877471eb16cfa7aa913b428/unnamed.png)

# はじめに

---

Proxmox環境で踏み台のWindowsが欲しかったのと、

Proxmox7からWindowsのバージョンで11/2022と表示されており

これならWindows11 proが使えるのではないか？と思い調べながらOSインストールまでしました。

※HOMEエディションはRDPできないので気をつけましょう。

# インストールの準備

---

WinodowsのISOイメージは以下

https://www.microsoft.com/ja-jp/software-download/windows11

virtIOドライバーのISOイメージは以下

https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso

上記２つをProxmoxのISO画面からダウンロードを実施

# VMの作成

---

- メモリは8GB以上がお勧めです。常時7GB前後ぐらい消費していました。
- CPUは4コアあれば十分でした。
- ネットワークをVirtIOを選択します。
- VMを作った後CD/DVDドライブを選択し、virtIOドライバーのISOを追加します。

![Untitled](Proxmox%E3%81%A6%E3%82%99Windows11%20Pro%E3%82%92%E8%B5%B7%E5%8B%95%20cc40147afdf545f1bde82ed31517f8a4/Untitled.png)

# Windowsのインストール

---

①通常通りのインストールで問題ありませんが、ハードディスクのドライバーがインストールされていないため、

ディスクが見えない状態になります。

そのためドライバーの読み込みから「Dドライブ(virtio-win-0.1.215) > vioscsi > w11 > amd64」を選択します。

※CPUの種類によって最後が変わるので気をつけてください。

②同じくネットワークのドライバーもドライバーの読み込みから、

「Dドライブ(virtio-win-0.1.215) > NetKVM> w11 > amd64」を選択します。

※CPUの種類によって最後が変わるので気をつけてください。

①②のドライバーをインストールしたら、Windowsのインストールが開始されます。

インストール完了後は、RDP接続の許可をするようにしてください。

時折リモート先からGUIが必要になるので、これで自宅メンテが捗ります。

Hyper-Vも利用できるのでちょっと何か検証したい時にも仮想マシンを動かすこともできるかな？と思います。