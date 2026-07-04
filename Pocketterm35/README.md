# Pocketterm35

- [参考](https://docs.waveshare.com/PocketTerm35/Software-Guide)

## PiImager
- SD へイメージを書き込む
    - デバイス (Pi5)、OS (64bit)、ストレージ (SD) を選択
    - ホスト名
        ~~~
        raspberrypi
        ~~~
    - ローカライゼーション
        ~~~
        CapitalCity         : Tokyo (Japan)
        TimeZone            : Asia/Tokyo
        キーボードレイアウト : us
        ~~~
    - ユーザ名
        ~~~
        ユーザ名 : pi (任意)
        パスワード : 任意
        ~~~
    - WIFI
        ~~~
        SSID       :
        パスワード  : 
        ~~~
    - SSH
        - SSH を有効化
        - パスワード認証を行う
    - Pi Connect
        - とりあえずオフにした

## A
- SD のルートディレクトリにある config.txt のけつに以下を追加する
    ~~~
    dtparam=i2c_arm=on
    dtoverlay=waveshare-35dpi-4b
    dtoverlay=waveshare-35dpi-5b
    dtoverlay=dwc2,dr_mode=host
    ~~~
- [ PocketTerm35 DTBO](https://files.waveshare.com/wiki/common/3.5HDMI_E_DTBO.zip) を DL
    - 解凍する
    - .dtbo ファイルを overlays/ 以下へコピーする

    

