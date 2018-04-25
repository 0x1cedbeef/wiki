<!-- TITLE: Forensics -->
<!-- SUBTITLE: A quick summary of Forensics -->

# USB
## キーボード入力の**key code**

キーボードは**押した**ときと**離した**ときの情報をOSに送信する
情報の受け取り方は実行中のプログラムやアプリケーションによって異なり、例えばvimなどでテキストを入力しているならそれは文字として認識されるし、ゲームだったら（wasdキーが）移動に割り当てられる。


[規格書より優しい解説ページ](https://docs.mbed.com/docs/ble-hid/en/latest/api/md_doc_HID.html)
マウスについての記載もページ下部にあり

[USB規格書PDF](http://www.usb.org/developers/hidpage/Hut1_12v2.pdf)
key codeについてはSection 10を参照

## データ転送

