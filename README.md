# MagicBand+ Research
This is unofficial hobby project.<br><br>
アメリカのディズニーパークで導入されているMagicBand+、および一部のディズニークルーズで導入されているDisneyBandの動作に関して調べるためのリポジトリです。

`83 01 E9 04 00 20 0F 00`


# Beacon Data Format for MagicBand+
MagicBand+は、パーク内のショーやアトラクション、銅像やクリスマスツリーなどのフォトロケーションに反応して光ったり振動したりします。
この効果は、パークに設置されているBluetoothビーコンから制御データが発信され、バンドがそれを受信することで実現しています。

ビーコンの中身はBluetoothのアドバタイジングデータであり、制御データはManufacturerDataとして格納・送信されます。
ここに格納されるデータによって、LEDを特定の色・パターンで光らせたり、振動モーターをリズミカルに駆動したりすることができます。

データ形式についてはまだまだ謎が多いですが、実際にパークでデータ収集した方などの投稿から少しずつ構造が分かってきています。
その情報を元に私自信でも調べてみました。
以下、現時点でわかっていることをまとめていきます。

# データ形式 / Data Format

## プロトコル / Protocol
| Type | size | value | note |
| ---- | ---- | ----  | ---- |
| Company ID  | 2bytes  | 0x8301 |          |
| Command     | 1byte   |  |          |
| Packet Size | 1byte   | 0~ |          |
| Packet Data | 0~n bytes |  | Size must be same as 'Packet Size' |
| Command2    | 1byte   |  | Option |
| Packet2 Size| 1byte   | 0~ | Option |
| Packet2 Data| 0~n bytes |  | Option / Size must be same as 'Packet Size'|
| ...         |         |  |          |

基本構成としては、Disney社のCompanyIDから始まり、その後コマンド、パケットサイズ、そしてパケット本体と続きます。
またアドバタイズデータとして収まる範囲であれば、同じ形式でさらにコマンドを増やすこともできます。

## 例 / Example

*Ex1 `83 01 E9 04 00 20 0F 00`<br>
- Company ID : `83 01`
- Data
  -  Command ID : `E9`
  -  Packet size : `04`
  -  Data : `00 20 0F 00`<br/>
  
*Ex2 `83 01 E1 00 E9 05 00 01 0E FD B0`<br>
- Company ID : `83 01`
- Data1
  -  Command ID : `E1`
  -  Packet size : `00`
  -  Data : none
- Data2
  -  Command ID : `E9`
  -  Packet size : `05`
  -  Data : `00 01 0E FD B0`
<br>


# コマンド一覧 / Command List

## 0xE9 - LED & Vibration Effect
## 0xCC - Ping?
## 0xC0 - Ping Response
## 0xC3 - Interactive
## 0xC4 - Interactive Response
## 0x0F - Unknown (General Purpose?)
