# MagicBand+ Research
This is an unofficial hobby project.<br>
Translated from Japanese to English with Google Translate or deepL.　Translations may contain errors.<br><br>
アメリカのディズニーパークで導入されているMagicBand+、および一部のディズニークルーズで導入されているDisneyBandの動作に関して調べるためのリポジトリです。<br/>
This repository is dedicated to investigating the operation of MagicBand+ (used at Disney parks in the US) and DisneyBand (used on select Disney Cruise Line ships).

# Beacon Data Format for MagicBand+
MagicBand+は、パーク内のショーやアトラクション、銅像やクリスマスツリーなどのフォトロケーションに反応して光ったり振動したりします。
この効果は、パークに設置されているBluetoothビーコンから制御データが発信され、バンドがそれを受信（場合により応答）することで実現しています。<br/>
MagicBand+ lights up and vibrates in response to elements within the park, such as shows, attractions, and photo locations (e.g., statues or Christmas trees). These effects are achieved when Bluetooth beacons installed throughout the park transmit control data, which the band receives (and occasionally responds to).<br/><br/>

ビーコンの中身はBluetoothのアドバタイジングデータであり、制御データはManufacturerDataとして格納・送信されます。
ここに格納されるデータによって、LEDを特定の色・パターンで光らせたり、振動モーターをリズミカルに駆動したりすることができます。<br/>
The beacon content consists of Bluetooth advertising data, with control data stored and transmitted as "ManufacturerData." The data contained here allows the band to illuminate its LEDs in specific colors or patterns and activate its vibration motor rhythmically.<br/><br/>

データ形式についてはまだまだ謎が多いですが、[EMCOT](https://emcot.world/Disney_MagicBand%2B_Bluetooth_Codes)のサイトや[jjdb210](https://github.com/jjdb210/Diz_BLE/tree/main)さんがパークで調べたデータなどから少しずつ構造が分かってきています。
それらの情報を元に私自信でも調べてみました。<br/>
以下、現時点でわかっていることをまとめていきます。<br/>
While much about the data format remains a mystery, the structure is gradually being revealed through posts by individuals who have collected data in the parks. I have also conducted my own investigations based on this information.
The following is a summary of what is currently known.<br/><br/>

# データ形式 / Data Format

## プロトコル / Protocol
| Type | size | value | note |
| ---- | ---- | ----  | ---- |
| Company ID  | 2bytes  | 0x8301 | ID for Walt Disney    |
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
- Command1
  -  Command No : `E9`
  -  Packet size : `04`
  -  Data : `00 20 0F 00`<br/><br/>
  
**Ex2** `83 01 E1 00 E9 05 00 01 0E FD B0`<br/>
- Company ID : `83 01`
- Command1
  -  Command No : `E1`
  -  Packet size : `00`
  -  Data : none
- Command2
  -  Command No : `E9`
  -  Packet size : `05`
  -  Data : `00 01 0E FD B0`
<br/>


# コマンド一覧 / Command List

## Shows / Attractions
MagicBand+はパークのショーやアトラクションに合わせて光ったり振動したりします。<br/>
これは設置されている送信機からコマンドを送ることで実現しています。すなわち一対多の一方向通信です。

### 0xE1~0xE3 - Sequence No.
E9コマンドと同じパケット内、E9コマンドの前に配置されるコマンドです。
常にパケット数は0のようです。<br/><br/>

用途としては、新しいE9コマンドを送る度にE1-E3を切り替えることで、MagicBand+がコマンドの更新を判別して適切に反応できるようにするためのものと思われます。<br/>
E9コマンドのみでも反応はしますが、このコマンドを使うことで反応の精度が良くなる気がします。
（あくまで実際の挙動からの推測です）

### 0xE9 - LED & Vibration Effect
LEDやバイブレーションの演出をするためのコマンドです。<br/>
ショーやアトラクションなど幅広い場所で使われています。<br/><br/>

必要な点滅パターンに応じてパラメーターを追加していく形で、パケット数は可変です。

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

## Purpose Unknown
### 0x03
### 0x05
### 0x0A
### 0x0F - (General Purpose?)
### 0x10
### 0xC1
**Example** `83 01 C1 0B 01 00 00 00 00 0B 53 CF F4 01 34`
### 0xC7
**Example** `83 01 C7 0A 01 01 B5 B5 00 00 44 33 33 31`
### 0xC8
**Examples** `83 01 C8 01 B2`
### 0xCB
**Example** `83 01 CB 03 02 00 20 CC 03 40 31 00` (Includes CC Command)
### 0xCF
**Example** `83 01 CF 0B 00 C4 20 22 2B 59 8F 74 02 EF 17`
### 0xE4
**Example** `83 01 E4 09 FF 00 00 00 5C 00 00 00 AB`
### 0xE5
**Example** `83 01 E5 01 AB C3 0A 01 00 0C 0B 03 08 3A 60 BA 25` (Includes C3 Command)
### 0xE6
**Example** `83 01 E6 0E 01 00 00 00 00 00 00 00 00 00 00 01 02 A6`
### 0xEA
**Example** `83 01 EA 14 01 00 81 0F 58 5C 58 F4 48 82 D0 65 29 D1 46 02 08 30 7B 40`
### 0xEF
**Example** `83 01 EF 05 01 A6 00 01 00`
**Example** `83 01 EF 08 01 A6 20 05 00 00 9D 00`


# 参考資料 / References
https://emcot.world/Disney_MagicBand%2B_Bluetooth_Codes <br/>
https://github.com/jjdb210/Diz_BLE/tree/main <br/>
