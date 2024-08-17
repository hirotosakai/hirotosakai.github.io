# むかし作ったMac用ソフトウェア

かつて http://www.fan.gr.jp/~sakai/ で配布していたMac用ソフトウェアを公開しています。

## DropLHa

LHA書庫を作るアプリケーション

- [v2 (for Mac OS X 10.2〜)](https://github.com/hirotosakai/droplha2)
- [v3 (for Mac OS X 10.4〜)](https://github.com/hirotosakai/droplha3)

## DropUnLHa

LHA書庫を解凍するアプリケーション

- [v1 (for Mac OS X 10.2〜)](https://github.com/hirotosakai/droplha2)
- [v2 (for Mac OS X 10.4〜)](https://github.com/hirotosakai/droplha3)

## Trunks

AppleGlotが生成するGlossaryファイルの簡易エディタ

- [リポジトリへ](https://github.com/hirotosakai/trunks)

## Camino ExtraFonts

フォントをはじめとする日本人ユーザー向けの設定が行えるCamino Webブラウザの環境設定パネル

- [リポジトリへ](https://github.com/hirotosakai/extrafonts)


# ビルド/テスト環境の構築

前記のソフトウェアは、主にMac OS X 10.2〜10.5上で開発したものです。ソースコードをビルド/テストする環境の構築方法を紹介します。

## ビルド環境

PowerPC, i386, x86_64のバイナリが生成できるMac OS X 10.5の環境を、[VMWare Fusion](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion)で構築するのが簡単です。

## Mac OS X (Intel x86_64)のテスト環境

VMWare FusionでMac OS X 10.5 Server (64ビット)の環境が作れます。その他の仮想化ソフトでも作れるかと思います。

## Mac OS X (Intel i386)のテスト環境

i386は選択肢が限られます。VMWare FusionでMac OS X 10.5 Server (32ビット)の環境を作ります。

### VMWare FusionでMac OS X 10.4 (Intel i386)を動かす

非サポートですが10.4も少し工夫すれば作れます。なお、Universal Binaryになった10.4.4以降のMac OS X ServerのDVDが必要です。

1. 「Mac OS X 10.5 Server (32ビット)」の仮想マシンを作る
2. Finderで仮想マシンを右クリックし、「パッケージの内容を開く」を選ぶ
3. vmxファイルをテキストエディタで開き、以下の行を追加する

 ```
cpuid.2.eax = "00000101101100001011000100000001"
cpuid.2.ebx = "00000000010101100101011111110000"
cpuid.2.ecx = "00000000000000000000000000000000"
cpuid.2.edx = "00101100101101000011000001001000"
 ```

4. DVDをセットして起動、インストールをすすめる

### VMWare Fusionで一部の記号が正しく打てない

USキーボードと認識されることがあります。vmxファイルに次の行を追加します。

```
keyboard.vusb.idVendor = 0x05AC
keyboard.vusb.idProduct = 0x020D
```

## Mac OS 9.2, Mac OS X (PowerPC)の環境

PowerPC搭載Macで動作するMac OS 9.2やMac OS Xの環境は、[QEMU](https://www.qemu.org/)で作れます。[UTM](https://mac.getutm.app/)を使うのも良いでしょう。

### インストールDVDのisoイメージを作る

ディスクユーティリティで「DVD/CDマスター」形式を指定して作るのが簡単ですが、コマンドで作ることもできます。DVDを接続して、

```
diskutil list
```

してデバイスファイル名（他に何も接続していなければたいがい /dev/disk2）を調べ、

```
sudo umount /dev/disk2
sudo dd if=/dev/disk2 of=./os.iso bs=1m
```

すると、os.isoができます。

### ハードディスクイメージを作る

フォーマットをrawに、ファイル名を〜.imgにすると、Finderでファイルを出し入れできて使い勝手が良いです。4GBのrawフォーマットのdisk.imgを作るには、次のようにします。

```
qemu-img create -f raw disk.img 4000M
```

### Mac OS X（PowerPC版）を動かす

DVD（disk1.iso）から起動してハードディスク（macosx.img）にインストールするには、次のコマンドを実行します。仮想マシンのスペックとして、メモリ：512MB、画面解像度：1280x768, 1677万色 を指定しています。

```
qemu-system-ppc -L pc-bios -M mac99,via=pmu \
-m 512 -g 1280x768x32 \
-netdev user,id=mynet0 -device sungem,netdev=mynet0 \
-boot d -cdrom ./disk1.iso \
-drive file=./macosx.img,format=raw,media=disk
```

インストール後は、ハードディスクから起動します。

```
qemu-system-ppc -L pc-bios -M mac99,via=pmu \
-m 512 -g 1280x768x32 \
-netdev user,id=mynet0 -device sungem,netdev=mynet0 \
-boot c \
-drive file=./macosx.img,format=raw,media=disk
```

### Mac OS 9.2を動かす

DVD（macos9.iso）から起動してハードディスク（macos9.img）にインストールするには、次のコマンドを実行します。時計の時刻が正しく表示されるよう `-rtc base=localtime` を追加します。

```
qemu-system-ppc -L pc-bios -M mac99,via=pmu \
-m 512 -g 1280x768x32 -rtc base=localtime \
-netdev user,id=mynet0 -device sungem,netdev=mynet0 \
-boot d -cdrom ./macos9.iso \
-drive file=./macos9.img,format=raw,media=disk
```

インストール後は、ハードディスクから起動します。

```
qemu-system-ppc -L pc-bios -M mac99,via=pmu \
-m 512 -g 1280x768x32 -rtc base=localtime \
-netdev user,id=mynet0 -device sungem,netdev=mynet0 \
-boot c \
-drive file=./macos9.img,format=raw,media=disk
```

### QEMUで一部の記号が正しく打てない

USキーボードをエミュレートするため、JISキーボード搭載Macでは一部の記号が正しく打てません。QEMUでキーボード配列を変更できないようなので、Mac側で[Karabiner-Elements](https://karabiner-elements.pqrs.org/)を使って対応します。

1. キーマップを変更するファイル [us-keyboard.json](us-keyboard.json) を ~/.config/karabiner/assets/complex_modifications へ置く
2. Karabiner-Elements の設定画面で Complex Modifications タブを開き、Add rule ボタンを押す
3. 「JIS キーボード使いの ASCII 配列な環境向けの設定」から必要な設定を Enable にする
