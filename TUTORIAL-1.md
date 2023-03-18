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
![RaspPi4_ModelB](https://user-images.githubusercontent.com/91422957/226090638-b4531217-3597-4238-9ab0-c7dad8554085.png)

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

![AE-BME280_pin_num](https://user-images.githubusercontent.com/91422957/226090687-cf9ab2e2-22a9-4272-8774-0c118d3c4891.jpg)

※ J1~J3もジャンパする

*Raspberry PiのGPIOピン位置*

<img width="492" alt="RaspPi_pinout" src="https://user-images.githubusercontent.com/91422957/226090707-b2af6174-c612-4031-a869-f7e4fdc60c1a.png">

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

![1](https://user-images.githubusercontent.com/91422957/226090789-28dfe43d-ba57-4555-9fda-b0b2b6a23519.png)

### ② 「１個のデバイスを接続」をクリック。

![2](https://user-images.githubusercontent.com/91422957/226090807-f49c3cf3-fd34-497c-a6c9-9d3499661f8d.png)


### ③ 「デバイスを準備する」の内容に従い、完了したら「次へ」をクリック。

![3](https://user-images.githubusercontent.com/91422957/226090826-87edf9d2-1fb7-4079-85f7-fe0b22e5106f.png)

### ④ 「新しいモノを作成」を選択し、「モノの名前」を記入する。完了したら「次へ」をクリック。

![4](https://user-images.githubusercontent.com/91422957/226090852-b5b5677b-d72c-4d32-8d1b-80cd647b6e31.png)

### ⑤ プラットフォームOSで「Linux/MacOS」、SDKで「Python」を選択し、「次へ」をクリック。

![5](https://user-images.githubusercontent.com/91422957/226090867-d1535808-0e82-423d-8cd1-eea753f1edae.png)

### ⑥ 作成された接続キットをRaspberry Piにダウンロードし解凍する（接続キットはここでしか入手できない）。完了したら「次へ」をクリック。

![6](https://user-images.githubusercontent.com/91422957/226090880-48a55706-3d3b-4b7c-9144-6a70feeb2dbf.png)

### ⑦ ダウンロードした接続キット内の「start.sh」に権限を付与し、実行する。接続成功するとサブスクリプションの部分に「Hello World!」と表示される。完了したら「次へ」をクリック。

![7](https://user-images.githubusercontent.com/91422957/226090893-aa09eb46-cefc-4dda-bce1-6e64518ed249.png)


### ⑧ デバイスがAWS IoT Coreに接続されていることを確認したら「モノを表示」をクリック。

![8](https://user-images.githubusercontent.com/91422957/226090909-3729a5e1-677a-4e50-8cd4-af4041fcb8f2.png)


### ⑨ 左サイドメニューの「"すべてのデバイス"→"モノ"」のページに遷移し、先ほど記入した名前のモノが表示される。

![9](https://user-images.githubusercontent.com/91422957/226090925-93261528-ad5e-48dd-ba62-23aa3dee0f54.png)
