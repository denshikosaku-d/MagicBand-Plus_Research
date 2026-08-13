# MagicBand+ Research
This is unofficial hobby project.<br><br>
アメリカのディズニーパークで導入されているMagicBand+、および一部のディズニークルーズで導入されているDisneyBandの動作に関して調べるためのリポジトリです。

# MagicBand+の動作の仕組み / How MagicBands Work in the parks

## Shows / Attractions
## Statues and Others

# Beacon Data Format for MagicBand+
MagicBand+は、パーク内のショーやアトラクション、銅像やクリスマスツリーなどのフォトロケーションに反応して光ったり振動したりします。
この効果は、パークに設置されているBluetoothビーコンから制御データが発信され、バンドがそれを受信（場合により応答）することで実現しています。<br/><br/>

ビーコンの中身はBluetoothのアドバタイジングデータであり、制御データはManufacturerDataとして格納・送信されます。
ここに格納されるデータによって、LEDを特定の色・パターンで光らせたり、振動モーターをリズミカルに駆動したりすることができます。<br/><br/>

データ形式についてはまだまだ謎が多いですが、実際にパークでデータ収集した方などの投稿から少しずつ構造が分かってきています。
その情報を元に私自信でも調べてみました。<br/>
以下、現時点でわかっていることをまとめていきます。<br/><br/>

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

<br/>
基本構成としては、Disney社のCompanyIDから始まり、その後コマンド、パケットサイズ、そしてパケット本体と続きます。<br/>
またアドバタイズデータとして収まる範囲であれば、同じ形式でさらにコマンドを増やすこともできます。<br/><br/>

## 例 / Example

**Ex1** `83 01 E9 04 00 20 0F 00`<br/>
- Company ID : `83 01`
- Data
  -  Command ID : `E9`
  -  Packet size : `04`
  -  Data : `00 20 0F 00`<br/><br/>
  
**Ex2** `83 01 E1 00 E9 05 00 01 0E FD B0`<br/>
- Company ID : `83 01`
- Data1
  -  Command ID : `E1`
  -  Packet size : `00`
  -  Data : none
- Data2
  -  Command ID : `E9`
  -  Packet size : `05`
  -  Data : `00 01 0E FD B0`
<br/>


# コマンド一覧 / Command List

## Shows / Attractions
### 0xE9 - LED & Vibration Effect

## Interactive
### 0xC3 - Interactive
パーク内の銅像に近づくとMagicBand+が反応し、装着者が手を振ったりすると、銅像からセリフが流れる、というインタラクティブ演出があります。<br/>
他にもDisney Springsのクリスマスツリーなどでも似た演出があるようですが、それらに使われていると思われるコマンドです。<br/>
このコマンドは、銅像やオブジェなどから不特定多数に送信されるものです。

### 0xC4 - Interactive Response
上記と同じインタラクティブ体験用のコマンドです。
銅像などから発信されたC3コマンドのデータを受信すると、「インタラクティブ体験が近くにありますよ」と装着者に知らせ、同時に動作の検知を開始します。
MagicBand+に内蔵されているセンサーで腕を振るなどの動作が検知されると、「応えた人がいるよ」という応答としてこのC4コマンドがバンドから送信されます。

## Park Utilities
### 0xCC - Ping?
このコマンドを受信することでMagicBand+からC0コマンドを含むデータが発信されます。<br/>
ある種のPingコマンドなのではないかと言われていますが、具体的な用途は不明です。<br/>
またこのコマンドを受信したバンドは、コマンド全般の受信感度や精度が向上するような印象があります。<br/><br/>

**Example** `83 01 CC 03 00 00 00`<br/>
パークで多く観測されているらしいデータです。<br/>
このデータを受信したバンドからは約15分間C0コマンドを含むビーコンが発信されます。<br/>

### 0xC0 - Ping Response

### 0x0F - Unknown (General Purpose?)
