# MagicBand+ Research
This is unofficial hobby project.<br><br>
アメリカのディズニーパークで導入されているMagicBand+、および一部のディズニークルーズで導入されているDisneyBandの動作に関して調べるためのリポジトリです。

`83 01 E9 04 00 20 0F 00`


## Beacon Data Format for MagicBand+
MagicBand+は、パーク内のショーやアトラクション、銅像やクリスマスツリーなどのフォトロケーションに反応して光ったり振動したりします。
この効果は、パークに設置されているBluetoothビーコンから制御データが発信され、バンドがそれを受信することで実現しています。

ビーコンの中身はBluetoothのアドバタイジングデータであり、制御データはManufacturerDataとして格納・送信されます。
ここに格納されるデータによって、LEDを特定の色・パターンで光らせたり、振動モーターをリズミカルに駆動したりすることができます。

データ形式についてはまだまだ謎が多いですが、実際にパークでデータ収集した方などの投稿から少しずつ構造が分かってきています。
その情報を元に私自信でも調べてみました。
以下、現時点でわかっていることをまとめていきます。

### データ形式 / Data Format
Company ID (2bytes)
Command1 (1byte)
Packet1 Size (1byte)
Packet1 Data (1-n bytes)
Command2 (1byte)
Packet2 Size (1byte)
Packet2 Data (1-n bytes)
...

| Type | size | note |
| ---- | ---- | ---- |
| Company ID  | 2bytes  |          |
| Command     | 1byte   |          |
| Packet Size | 1byte   |          |
| Packet Data | n bytes | Size must be same as 'Packet Size' |
| Command2    | 1byte   | If needed |
| Packet2 Size| 1byte   | If needed |
| Packet2 Data| n bytes | If needed |
| ...         |         |          |

