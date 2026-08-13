# MagicBand+ Research
This is unofficial hobby project.<br><br>
アメリカのディズニーパークで導入されているMagicBand+、および一部のディズニークルーズで導入されているDisneyBandの動作に関して調べるためのリポジトリです。

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
MagicBand+はパークのショーやアトラクションに合わせて光ったり振動したりします。<br/>
これは設置されている送信機からコマンドを送ることで実現しています。すなわち一対多の一方向通信です。

### 0xE9 - LED & Vibration Effect

## Interactive
WDW内ではキャラクターの銅像が設置されており、そこに近づくとMagicBand+が反応、手を振るなどの行動を起こすとセリフや音楽が流れる、という演出があります。<br/>
他にも、Disney Springsのクリスマスツリーといったデコレーションにも同じ機能があったりします。<br/><br/>

このとき、銅像やデコレーションからは常にビーコンデータが発信され、それを受信したMagicBand+が反応し、動作の検知を開始します。<br/>
手を振るなどの動作が検知されると、今度はMagicBand+からもデータが送信され、それを銅像などが受信することで演出がトリガーされます。<br/><br/>

以下は、このインタラクティブ体験に使われるコマンドです。<br/>

### 0xC3 - Interactive
このコマンドは銅像やオブジェなどから不特定多数に送信され、MagicBand+のインタラクティブ機能を立ち上げるものです。<br/>

**Ex** 
`83 01 C3 0A 01 00 1F 0B 03 08 3A 60 BC 25`
- Company ID : `83 01`
- Command
  -  Command No : `C3`
  -  Packet size : `0A`
  -  Data
    - `0100`  : Unknown
    - `1F`    : Random
    - `0B`    : Unknown
    - `03`    : Lighting Pattern 1
    - `08`    : Lighting Pattern 2
    - `3A`    : Vibration Pattern 1(Higher 4bits) & 2(Lower 4bits)
    - `60BC25`: Unknown
<br/>

### 0xC4 - Interactive Response

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
