# 開発チュートリアル①


## 目次
1. Raspberry PiおよびAE-BME280のセットアップ
2. リポジトリのインストール
3. AWSアカウントおよびIAMユーザーの作成
4. AWS IoT Coreのセットアップ
5. pubsub.pyの修正
6. start.shの修正
7. cronでスケジュール管理

****
## 1. Raspberry PiおよびAE-BME280のセットアップ
<img src="/Users/matsugano/AE-BME280_for_AWS/images/1.RaspPi_setup/RaspPi4_ModelB.png">

### ●必要なもの
- Raspberry Pi本体
- AE-BME280
- microSDカード
- ACアダプター
- USBマウス
- USBキーボード
- HDMIケーブル
- 無線または有線LAN（インターネットに接続が必要）
- ジャンパーワイヤ（メス・メス）
- はんだごてセット

*~SSH接続して遠隔でRaspberry Piを操作する場合（推奨）~*
- PC
- VSCodeなどのエディタ

### 1.1 Raspberry Piのセットアップをする
*セットアップ方法は以下を参考*

> **python-academia.com**<br>
> [RaspberryPi(ラズベリーパイ)の初期設定・購入したらやること](https://python-academia.com/raspberrypi-initialsetting/)<br>
参考日: 2022.10.14

> **sozorablog.com**<br>
>[OSインストールから初期設定まで｜セットアップ手順のすべて](https://sozorablog.com/raspberrypi_initial_setting/)<br>
参考日: 2022.11.3

### 1.2 AE-BME280をRaspberry Piに配線接続する
#### ① AE-BME280をRaspberry Piに配線接続する

*配線するピンの組み合わせ*
| Raspberry Pi側GPIOピン | センサー側ピン |
| :---: | :---:|
| **1**(3v3 Power) | **1**(VDD) |
| **6**(*GND) | **2**(GND) |
| **17**(3v3 Power) | **3**(CSB) |
| **3**(SDA) | **4**(SDI) |
| **9**(GND) | **5**(SOD) |1
| **5**(SCL) | **6**(SCK) |

*AE-BME280のピン位置*

<img src="/Users/matsugano/AE-BME280_for_AWS/images/1.RaspPi_setup/AE-BME280_pin_num.jpg">

※ J1~J3もジャンパする

*Raspberry PiのGPIOピン位置*

<img src="/Users/matsugano/AE-BME280_for_AWS/images/1.RaspPi_setup/RaspPi_pinout.png">

参考:
> **pinout.xyz**<br>
> [Raspberry Pi GPIO Pinout](https://pinout.xyz/)<br>
参考日: 2022.11.5


#### ② I2C通信の有効化

#### ③ I2C通信ツールの導入
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install i2c-tools
$ sudo apt-get install python-smbus
```

*接続できているか確認*
```
i2cdetect -y 1
```

「76」と表示されていれば接続成功

参考:
> **dev.classmethod.jp**<br>
> [[初心者向け] 温度湿度気圧センサーのデータをRaspberry PiからAWS IoT Coreに送ってみた](https://dev.classmethod.jp/articles/for-begginer-temp-hum-puressure-data-send-to-awsiotcore-via-raspberrypi/)<br>
参考日: 2022.11.5

### 1.3 Raspberry Piにケースをつける
Raspberry Pi（特にRaspberry Pi 4 Model B）は発熱を起こしやすい。<br>
放熱とパフォーマンス維持のため、ヒートシンク付きの専用ケースをつけて動作させることを推奨する。

*※以下のようなメタルケースがおすすめ*
> **amazon.co.jp**<br>
> [Raspberry Pi 4 ケース 超薄型アルミニウム合金ケース+ヒートシンク+ 35x35x10 冷却大 ファン](https://www.amazon.co.jp/gp/product/B087D4MYRB/ref=ppx_yo_dt_b_asin_title_o05_s00?ie=UTF8&psc=1)<br>
参考日: 2022.10.16

****
## 2. リポジトリのインストール
### 「AE-BME280_for_AWS」をインストールする
#### 方法①
```
git clone https://github.com/gano6/AE-BME280_for_AWS.git
```
*※ "git"のインストールが必要*

#### 方法②
「<> code」→ 「Download ZIP」

****
## 3. AWSアカウントおよびIAMユーザーの作成
### 3.1 AWSアカウントの作成
*作成方法は以下を参考*

> **aws.amazon.com**<br>
> [AWS アカウント作成の流れ【AWS 公式】](https://aws.amazon.com/jp/register-flow/)<br>
参考日: 2023.3.18

### 3.2 IAMユーザーの作成
*作成方法は以下を参考*

> **docs.aws.amazon.com**<br>
> [AWS アカウント での IAM ユーザーの作成 - AWS Identity and Access Management](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_users_create.html)<br>
参考日: 2023.3.18

*※ 最低限アタッチするポリシーは以下の通りである*

- AdministratorAccess
- AWSIoTFullAccess
- AWSIoTAnalyticsFullAccess
- AWSQuickSightIoTAnalyticsAccess

## 4. AWS IoT Coreのセットアップ

*セットアップ前の必要事項*

- リージョンは「バージニア北部（us-east-1）」
- Raspberry Piに「git」をインストール
- Raspberry Piに「Python（ver3.6以上）」をインストール

### ① AWSマネジメントコンソールにログインし、AWS IoT Coreを開く。

<img src="/Users/matsugano/AE-BME280_for_AWS/images/4.AWS_IoT_Core_setup/1.png">

### ② 「１個のデバイスを接続」をクリック。

<img src="/Users/matsugano/AE-BME280_for_AWS/images/4.AWS_IoT_Core_setup/2.png">

### ③ 「デバイスを準備する」の内容に従い、完了したら「次へ」をクリック。

<img src="/Users/matsugano/AE-BME280_for_AWS/images/4.AWS_IoT_Core_setup/3.png">

### ④ 「新しいモノを作成」を選択し、「モノの名前」を記入する。完了したら「次へ」をクリック。

<img src="/Users/matsugano/AE-BME280_for_AWS/images/4.AWS_IoT_Core_setup/4.png">

### ⑤ プラットフォームOSで「Linux/MacOS」、SDKで「Python」を選択し、「次へ」をクリック。

<img src="/Users/matsugano/AE-BME280_for_AWS/images/4.AWS_IoT_Core_setup/5.png">

### ⑥ 作成された接続キットをRaspberry Piにダウンロードし解凍する（接続キットはここでしか入手できない）。完了したら「次へ」をクリック。

<img src="/Users/matsugano/AE-BME280_for_AWS/images/4.AWS_IoT_Core_setup/6.png">

### ⑦ ダウンロードした接続キット内の「start.sh」に権限を付与し、実行する。接続成功するとサブスクリプションの部分に「Hello World!」と表示される。完了したら「次へ」をクリック。

<img src="/Users/matsugano/AE-BME280_for_AWS/images/4.AWS_IoT_Core_setup/7.png">

### ⑧ デバイスがAWS IoT Coreに接続されていることを確認したら「モノを表示」をクリック。

<img src="/Users/matsugano/AE-BME280_for_AWS/images/4.AWS_IoT_Core_setup/8.png">

### ⑨ 左サイドメニューの「"すべてのデバイス"→"モノ"」のページに遷移し、先ほど記入した名前のモノが表示される。

<img src="/Users/matsugano/AE-BME280_for_AWS/images/4.AWS_IoT_Core_setup/9.png">