# RaspberryPi

## 初期設定
- SDカードのbootへ以下を作る
    - wpa_supplicant.confを作成
    ~~~
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1
    country=JP
    network={
        ssid="XXXX"
        psk="YYYY"
    }
    ~~~
    - sshを作成(内容は空)

- 起動すると初期設定のダイアログが出るのでそれに従って進める

- 更新
    - $sudo apt-get update
    - $sudo apt-get upgrade

- ファイルシステムの拡張
    - $sudo raspi-config - Advanced Options - Expand filesystem

- SSH
    - $sudo raspi-config - Interfacing Options - SSH - Yes

 - Samba
    - $sudo apt-get install -y Samba
    - この時点でホスト名が解決できるようになる
        - ping raspberrypi.local
        - ホスト名を変えたい場合 /etc/hosts, /etc/hostname   

<!--
- ヘッドレス設定
    - SDカード - boot以下
        - config.txtの最後に以下の行を追加
        ~~~
        dtoverlay=dwc2
        ~~~
        - cmdline.txtのrootwait以降に空白区切りで改行せずに以下を追加
        ~~~
        modules-load=dwc2,g_ether
        ~~~
    - Windowsに[Bonjour Print Services](https://support.apple.com/kb/DL999?viewlocale=ja_JP&locale=en_US)をインストール
    - 電源でない方(内側)のUSBをWindowsとつなげる
        - PiのLED点滅が終わったら - デバイスマネージャー - ネットワークアダプタ - USB Ethernet/RNDIS Gadgetがインストールされていることを確認する
-->

- VNC
    - $sudo raspi-config - Interfacing Options - VNC - Yes

<!--
    - Bluetooth
    - $sudo apt-get install -y blueman
    - $startx
        - 左上のラズベリーパイのアイコン - Preferences - Bluetooth Manager
        - デバイスを検索して見つかれば Device - Pair でペアリング
    - コマンドライン        
        - スキャン可能、不可能にする
            - $sudo hciconfig hci0 piscan
            - $sudo hciconfig hci0 noscan
        - 情報の表示
            - $sudo hciconfig -a
        - スキャンする
            - $sudo hciconfig scan
        - ping
            - $sudo l2ping -c 1 <scanで見つかったMACアドレス>
-->

- [状態調査](https://www.raspberrypi.org/documentation/raspbian/applications/vcgencmd.md)(vcgencmd)
    - 電源不足チェック
        - $vcgencmd get_throttled
        - throttled 0x0 と表示されればOK
    - 温度
        - $vcgencmd measure_temp
    - 電圧
        - $vcgencmd measure_volts
    - メモリ
        - $vcgencmd get_mem arm
        - $vcgencmd get_mem gpu
        - cat /proc/meminfo

- 解像度
    - 左上のラズベリーパイのアイコン - 設定 - Screen Configuration - 右クリック - 解像度を選択
    - Configure - 適用

- カメラモジュール
    - 有効化する
        - $sudo raspi-config - Interfacing Options - Camera - Yes
    - 正しく接続されているかチェック
        - $vcgencmd get_camera
    - 撮影、確認
        - $sudo raspistill -o XXX.jpg
        - $gpicview XXX.jpg
    - 動画(10秒)
        - raspivid -t 10000 -o XXX.h264

- [GPIO](https://www.raspberrypi.org/documentation/usage/gpio/)
    - ピンレイアウト確認
        - $pinout
    - [libgpiod](https://github.com/brgl/libgpiod.git)
        - 予めインストールしておく必要のあるもの
            - $sudo apt-get install -y autoconf
            - $sudo apt-get install -y libtool
            - $sudo apt-get install -y autoconf-archive
            - Doxygen生成に必要
                - $sudo apt-get install -y doxygen
                - $sudo apt-get install -y graphviz
        - クローン
            - $git clone -b v0.3.x https://github.com/brgl/libgpiod.git
                - masterブランチではなく、v0.3.xブランチでないとダメ
            - $cd libgpiod
            - インストール
                - $./autogen.sh --enable-tools=yes --prefix=/usr/local
                - $make
                - $sudo make install
            - ドキュメント生成
                - doxygen
                - doc/html/index.htmlをブラウザで開く
        - コマンドライン
            - libgpiod.so.0 が無いと怒られる場合LD_LIBRARY_PATHにインストール先を指
            - 例えば.bashrcに以下のように書いておく
            ~~~
            LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
            export LD_LIBRARY_PATH
            ~~~
            - gpiodetect
            - gpioinfo
            - gpioget
            - gpioset
            - gpiofind
            - gpiomon
            - 例
                - 
<!--
- シリアルデバッグ
    - シリアルデバッグを有効化する
        - $sudo raspi-config - Interfacing Options - Serial - Yes
        - 接続 ([GPIO](https://www.raspberrypi.org/documentation/usage/gpio/))
            - 黒 GND(14の隣, 15の向い,...等)
            - 緑 RXD(15)
            - 白 TXD(14)
            - 赤
        - [RLogin](http://nanno.dip.jp/softlib/man/rlogin/)等でCOMの設定をする
            - シリアル変換USBケーブルをWindowsに刺す
            - デバイスマネージャー - ポート(COMとLPT) - COM番号を覚えておく
            - [RLogin](http://nanno.dip.jp/softlib/man/rlogin/)から新規接続
                - プロトコル - COMにチェック
                - TCPポート - COM3等を選択(COM番号はデバイスマネージャーで調べる)
                - シリアル - 設定ボタン - ビット/秒を115200
-->

<!--
        - PL2303の場合のドライバインストール
            - 説1
                - http://www.ifamilysoftware.com/Drivers/PL-2303_Driver_Installer.exe をインストール

            - 説2
                - [ドライバ](http://www.prolific.com.tw/JP/ShowProduct.aspx?pcid=126&showlevel=0126-0126)をインストール
                - Win10の場合[修正プログラム](http://www.ifamilysoftware.com/news37.html)をインストールしておく

            - 説3
                - [秋月](http://akizukidenshi.com/catalog/faq/goodsfaq.aspx?goods=M-02746)からv1.5をインストール
--> 

## Windows

-  [RLogin](http://nanno.dip.jp/softlib/man/rlogin/)等でタブを押す度に音が鳴ってうるさい対策
    - Windowsキー右クリック - ファイル名を指定して実行 -  mmsys.cpl
        - サウンド - 一般の警告音 - Windows Background.wav をなしに変更

## そのた
- パスワード変更
    - $sudo raspi-config - Change Password
    - $sudo passwd root

- git
    - $sudo apt-get install -y git
    - 指定ディレクトリ名(hoge)でクローンする場合
        - $git clone https://.... hoge
    - ユーザ設定
        - $git config --global user.email "XXX@YYY"
        - $git config --global user.name "ZZZ"
    - $git commit
        - エディタが立ち上がるのでコミットメッセージを書く
        - エディタを変更する場合
            - $git config --global core.editor "emacs -nw"
    - $git push

- w3m
    - $sudo apt-get install -y w3m

- emacs
    - $sudo apt-get install -y emacs
    - .emacs.d/init.elを作成
    ~~~
        ;バックアップを作らない
        (setq make-backup-files nil)
        (setq auto-save-default nil)
    ~~~

<!--
- VSCode
    - $sudo -s
    - #. <( wget -O - https://code.headmelted.com/installers/apt.sh )
	- $code-oss で起動する
-->

## [PiStarter](https://github.com/horinoh/PiStarter.git)
