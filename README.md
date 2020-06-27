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
    ~~~
    $sudo apt-get update
    $sudo apt-get upgrade
    $sudo apt-get clean
    $sudo apt-get autoclean
    ~~~

- ファイルシステムの拡張
    ~~~
    $sudo raspi-config - Advanced Options - Expand filesystem
    ~~~

- SSH
    ~~~
    $sudo raspi-config - Interfacing Options - SSH - Yes
    ~~~
    - Windows Power Shell からログイン
        ~~~
        ssh pi@raspberrypi.local
        ~~~
    - もしくは[RLogin](http://nanno.dip.jp/softlib/man/rlogin/)等でログイン

 - Samba
    ~~~
    $sudo apt-get install -y Samba
    ~~~
    - この時点でホスト名が解決できるようになる
        - ping raspberrypi.local
        - ホスト名を変えたい場合 /etc/hosts, /etc/hostname   

    - 設定
        ~~~
        $mkdir /home/pi/share
        ~~~
        - /etc/samba/smb.conf
        ~~~
        [share]
        comment = Share
        path = /home/pi/share
        public = yes
        read only = no
        browsable = yes
        force user = pi
        ~~~
        ~~~
        $sudo smbpasswd -a pi
        ~~~
        ~~~
        $sudo systemctl restart smbd
        ~~~
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
    ~~~
    $sudo raspi-config - Interfacing Options - VNC - Yes
    ~~~
    - [VNCViewer](https://www.realvnc.com/en/connect/download/viewer/)等でログインする
    - リモートで(モニタを直接接続せずに)X-Windowを起動したい場合、HDMIダミープラグを刺しておくと良い

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
        ~~~
        $vcgencmd get_throttled
        ~~~
        - throttled 0x0 と表示されればOK
    - 温度
        ~~~
        $vcgencmd measure_temp
        ~~~
    - 電圧
        ~~~
        $vcgencmd measure_volts
        ~~~
    - メモリ
        ~~~
        $vcgencmd get_mem arm
        $vcgencmd get_mem gpu
        $cat /proc/meminfo
        ~~~

- 解像度
    - 左上のラズベリーパイのアイコン - 設定 - Screen Configuration - 右クリック - 解像度を選択
    - Configure - 適用

- カメラモジュール
    - 有効化する
        ~~~
        $sudo raspi-config - Interfacing Options - Camera - Yes
        ~~~
    - 正しく接続されているかチェック
        ~~~
        $vcgencmd get_camera
        ~~~
    - 撮影、確認
        ~~~
        $sudo raspistill -o XXX.jpg
        $gpicview XXX.jpg
        ~~~
    - 動画(10秒の例)
        ~~~
        $raspivid -t 10000 -o XXX.h264
        ~~~
    - 動体検知
        - インストール
            ~~~
            $sudo apt-get install -y motion
            ~~~
        - 起動、停止
            ~~~
            $sudo motion
            $sudo service motion stop
            ~~~
        - ブラウザから
            - 設定
                - http://localhost:8080
            - ストリーム表示
                - http://localhost:8081
        - 設定
            - /etc/motion/motion.conf 
            ~~~
            # 静止画、動画を保存する/しない設定
            output_pictures on
            ffmpeg_output_movies on
            # 保存先
            target_dir /var/lib/motion
            ~~~

- [GPIO](https://www.raspberrypi.org/documentation/usage/gpio/)
    - ピンレイアウト確認
        ~~~
        $pinout
        ~~~
    - pigpio
        ~~~
        $sudo systemctl enable pigpiod
        $sudo systemctl start pigpiod
        $sudo systemctl restart pigpiod //!< リスタートする場合
        ~~~
        - センサーにはI2CやSPIを使うものがあるので有効にしておく
            ~~~
            $sudo raspi-config - 
            ~~~
        - サンプルプログラム
            - 点滅
                ~~~
                import pigpio
                import time
                PIN = 14
                pi = pigpio.pi()
                pi.set_mode(PIN, pigpio.OUTPUT)
                pi.write(PIN, pigpio.HIGH)
                while True:
                    pi.write(PIN, pigoio.LOW)
                    time.sleep(1)
                    pi.write(PIN, pigoio.HIGH)
                    time.sleep(1)
                ~~~
            - PWM
                ~~~
                ...
                pi.set_PWM_frequency(PIN, 50) # 周波数(Hz)... デフォルト800
                pi.set_PWM_range(PIN, 100) # [25, 40000]
                DUTY = 30 # [0, set_PWM_range()] ... ここでは[0, 100]
                pi.set_PWM_dutycycle(PIN, DUTY) # Highの割合 ... ここでは 30/100
                ~~~
        - 実行
            ~~~
            $python2 XXX.py
            ~~~
    - [WiringPi](http://wiringpi.com/download-and-install/)
        - インストール
            ~~~
            $sudo apt-get install wiringpi
            ~~~
        - インストールされているかチェック
            ~~~
            $gpio -v
            $gpio readall
            ~~~
            - unable to determin board type... model: 17 とエラーになる(Pi4)場合(以下のようにしてバージョンを上げる)
                ~~~
                $cd /tmp
                $wget https://project-downloads.drogon.net/wiringpi-latest.deb
                $sudo dpkg -i wiringpi-latest.deb
                ~~~
        - [Sunfounderサンプル](https://github.com/sunfounder/davinci-kit-for-raspberry-pi.git)
            ~~~
            $git clone https://github.com/sunfounder/davinci-kit-for-raspberry-pi.git
            ~~~
            - コンパイル例
                ~~~
                $gcc 1.1.1_BlinkingLed.c -o BlinkingLed -lwiringPi
                ~~~
        
    - [libgpiod](https://github.com/brgl/libgpiod.git)
        - 予めインストールしておく必要のあるもの
            ~~~
            $sudo apt-get install -y autoconf
            $sudo apt-get install -y libtool
            $sudo apt-get install -y autoconf-archive
            ~~~
            - Doxygen生成に必要
                ~~~
                $sudo apt-get install -y doxygen
                $sudo apt-get install -y graphviz
                ~~~
        - クローン
            ~~~
            $git clone -b v0.3.x https://github.com/brgl/libgpiod.git
            ~~~
            - masterブランチではなく、v0.3.xブランチでないとダメ
            ~~~
            $cd libgpiod
            ~~~
            - インストール
                ~~~
                $./autogen.sh --enable-tools=yes --prefix=/usr/local
                $make
                $sudo make install
                ~~~
            - ドキュメント生成
                ~~~
                $doxygen
                ~~~
                - doc/html/index.htmlをブラウザで開く
        - コマンドライン
            - libgpiod.so.0 が無いと怒られる場合LD_LIBRARY_PATHにインストール先を指
            - 例えば.bashrcに以下のように書いておく
                ~~~
                LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
                export LD_LIBRARY_PATH
                ~~~
            - gpiodetect
                - GPIOチップをリストアップ
            - gpioinfo
                - GPIOチップのラインをリストアップ
            - gpioget
                - GPIOラインの値をゲット
            - gpioset
                - GPIOラインへ値をセット
            - gpiofind
                - 名前からGPIOチップとラインをゲット
            - gpiomon
            - 例
                ~~~
                # ヘルプ表示
                $gpioXXX -h

                # GPIOチップをリストアアップ
                $gpiodetect
                gpiochip0 ... (54 lines)

                # GPIOチップのラインをリストアップ
                $gpioinfo gpiochip0
                gpiochip0 - 54 lines
                    line    0:  unmaed unused   input active-high
                    ...
                    line    53: unmaed unused   input active-high
                
                # 値のゲット
                $gpioget gpiochip1 23
                0
                
                # 値のセット
                $gpioset gpiochp1 23=1

                # GPIOチップとラインを名前から見つける
                $gpiofind "USR-LED-2"
                gpiochiop1 23

                $gpioset --mode=wait `gpiofind "USR-LED-2"`=1

                $gpioset --mode=signal --background gpiochip1 23=0 24=0
                $gpioset gpiochio1 23=1
            
                $gpioget --active-low gpiochip1 23 24
                1 1
                
                $gpiomon --num-events=3 --rising-edge gpiochip2 3
                $gpiomon --format="%e %o %s %n" --falling-edge gpiochip1 4
                $gpiomon --num-events=1 --silent `gpiofind "USR-IN"`
                $gpiomon --silent --num-events=1 gpiochip0 2 3 5
                ~~~

- GPIOからのオーディオ出力をオフする場合
    - オンになっていると一部の電子パーツで動作がおかしくなるらしい
    - /boot/config.txt
        ~~~
        # コメントアウトする
        #dtparam=audio=on
        ~~~

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

## [Vulkan](https://github.com/horinoh/PiVK/tree/master/)

## そのた
- パスワード変更
    ~~~
    $sudo raspi-config - Change Password
    $sudo passwd root
    ~~~

- 空き容量を調査
    ~~~
    $sudo du -sh /*
    ~~~
    - /var/cache が容量を食っている場合は以下のようにしてみる
        ~~~
        $sudo apt-get clean
        $sudo apt-get autoclean
        ~~~

- git
    - インストール
        ~~~
        $sudo apt-get install -y git
        ~~~
    - 指定ディレクトリ名(TARGETDIR)でクローンする場合
        ~~~
        $git clone https://.... TARGETDIR
        ~~~
    - ユーザ設定
        ~~~
        $git config --global user.email "XXX@YYY"
        $git config --global user.name "ZZZ"
        ~~~
    - コミット
        ~~~
        $git commit
        ~~~
        - エディタが立ち上がるのでコミットメッセージを書く
        - エディタを変更する場合
            ~~~
            $git config --global core.editor "emacs -nw"
            ~~~
    - プッシュ
        ~~~
        $git push
        ~~~

- w3m
    ~~~
    $sudo apt-get install -y w3m
    ~~~

- emacs
    ~~~
    $sudo apt-get install -y emacs
    ~~~
    - .emacs.d/init.elを作成
        ~~~
            ;バックアップを作らない
            (setq make-backup-files nil)
            (setq auto-save-default nil)
        ~~~

- apache
    ~~~
    $sudo apt-get install -y apache2
    ~~~
    - http://raspberrypi.local  へアクセスできるようになる
    - 起動、停止、確認
        ~~~
        $sudo systemctl enable apache2
        $sudo systemctl disable apache2
        $sudo systemctl status apache2
        ~~~
    - 設定
        - /etc/apache2/以下、apache2.conf等
    - コンテンツ
        - /var/www/html
    - ログ
        - /var/log/apach2

- mathmatica
    ~~~
    $sudo apt-get install -y wolfram-engine
    ~~~
- ソフトウエアキーボード
    ~~~
    $sudo apt-get install -y matchbox-keyboard
    ~~~
    - アクセサリ - keyboard で起動
- ベンチマーク
    - UnixBench
        <!--
        $sudo apt-get install -y build-essential
        -->
         ~~~
        $git clone https://github.com/kdlucas/byte-unixbench.git
        $cd byte-unixbench/UnixBench
        $./Run
        ~~~
    - [GeeXLab](https://geeks3d.com/geexlab/) (3D)
        ~~~
        $unzip GeeXLab_XXX.zip
        $cd GeeXLab_FREE_rpi_gl21
        // demo.shを編集し、起動したいデモのコメントアウトを外して起動する
        $./demo.sh
        // 直接コマンドライン起動してもよい
        $./GeeXLab /demofile=\"demopack01/d11-gears/main.xml\"
        ~~~
    - iPerf (ネットワーク)

<!--
- VSCode
    - $sudo -s
    - #. <( wget -O - https://code.headmelted.com/installers/apt.sh )
	- $code-oss で起動する
-->

## [PiStarter](https://github.com/horinoh/PiStarter.git)
