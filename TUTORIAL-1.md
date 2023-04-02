# 開発チュートリアル①


## 目次

1. はじめに
2. Raspberry PiおよびAE-BME280のセットアップ
3. リポジトリのインストール
4. AWSアカウントおよびIAMユーザーの作成
5. AWS IoT Coreの設定
6. "aws-iot-device-sdk-python-v2"の置換
7. start.shの修正
8. cronでスケジュール管理

※ ファイル左上の「:=」のようなアイコンに各セクションのリンクがあります。


## 1. はじめに
### 1.1 ご挨拶
本チュートリアルをお読みいただきありがとうございます。<br>
「AE-BME280_for_AWS」では、開発チュートリアルを2部に分けて公開しております。基本的には農業IoT向けの開発としていますが、その他の目的でご利用、開発される際にも十分お使いいただけます。ご不明な点、誤っている点などございましたらぜひお問合せください。<br>
何卒よろしくお願いいたします。

### 1.2 Pythonのエラーについて
本チュートリアルでは、主な開発言語としてPythonを用います。<br>
「AE-BME280_for_AWS」を開発する際に、最も発生していたエラーの解決方法を以下にまとめます。<br>
Pytohn初学者の方は参考にしていただけますと幸いです。

## *ModuleNotFoundError*

指定したモジュールが存在しない、または見つからない場合に発生するエラー。

<解決方法 1>
- 該当モジュールをインストールする
```
$ pip3 install モジュール名
```

<解決方法 2>
- 該当モジュールをインポートする

```
import モジュール名
```

<解決方法 3>
- 指定したいモジュールのパスを通す

以下のように「PYTHONPATH」を使った方法がおすすめ

*① bash_profileをvimで開く*
```
vi ~/.bash_profile
```

*② PYTHONPATHを記載する*

「i」で記入
```
PYTHONPATH=該当モジュールの絶対パス
```
モジュールのパスが複数の場合
```
PYTHONPATH=モジュール1の絶対パス:モジュール2の絶対パス
```

「escキー」→「:wq!」で保存、vimが閉じる

*③ 編集したbash_profileを反映させる*
```
source ~/.bash_profile
```

****
## 2. Raspberry PiおよびAE-BME280のセットアップ
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

**SSH接続して遠隔でRaspberry Piを操作する場合（推奨）**
- PC
- VSCodeなどのエディタ

### 2.1 Raspberry Piのセットアップをする
*セットアップ方法は以下を参考*

> **python-academia.com**<br>
> [RaspberryPi(ラズベリーパイ)の初期設定・購入したらやること](https://python-academia.com/raspberrypi-initialsetting/)<br>
参考日: 2022.10.14

> **sozorablog.com**<br>
>[OSインストールから初期設定まで｜セットアップ手順のすべて](https://sozorablog.com/raspberrypi_initial_setting/)<br>
参考日: 2022.11.3

### 2.2 AE-BME280をRaspberry Piに配線接続する
#### ① AE-BME280をRaspberry Piに配線接続する

*配線するピンの組み合わせ*
| Raspberry Pi側GPIOピン | センサー側ピン |
| :---: | :---:|
| **1**(3v3 Power) | **1**(VDD) |
| **6**(GND) | **2**(GND) |
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
$ i2cdetect -y 1
```

「76」と表示されていれば接続成功

参考:
> **dev.classmethod.jp**<br>
> [[初心者向け] 温度湿度気圧センサーのデータをRaspberry PiからAWS IoT Coreに送ってみた](https://dev.classmethod.jp/articles/for-begginer-temp-hum-puressure-data-send-to-awsiotcore-via-raspberrypi/)<br>
参考日: 2022.11.5

### 2.3 Raspberry Piにケースをつける
Raspberry Pi（特にRaspberry Pi 4 Model B）は発熱を起こしやすい。<br>
放熱とパフォーマンス維持のため、ヒートシンク付きの専用ケースをつけて動作させることを推奨する。

*※以下のようなメタルケースがおすすめ*
> **amazon.co.jp**<br>
> [Raspberry Pi 4 ケース 超薄型アルミニウム合金ケース+ヒートシンク+ 35x35x10 冷却大 ファン](https://www.amazon.co.jp/gp/product/B087D4MYRB/ref=ppx_yo_dt_b_asin_title_o05_s00?ie=UTF8&psc=1)<br>
参考日: 2022.10.16

****
## 3. リポジトリのインストール
### 「AE-BME280_for_AWS」をインストールする
#### 方法①
```
git clone https://github.com/gano6/AE-BME280_for_AWS.git
```
*※ "git"のインストールが必要*

#### 方法②
「<> code」→ 「Download ZIP」

****
## 4. AWSアカウントおよびIAMユーザーの作成
### 4.1 AWSアカウントの作成
*作成方法は以下を参考*

> **aws.amazon.com**<br>
> [AWS アカウント作成の流れ【AWS 公式】](https://aws.amazon.com/jp/register-flow/)<br>
参考日: 2023.3.18

### 4.2 IAMユーザーの作成
*作成方法は以下を参考*

> **docs.aws.amazon.com**<br>
> [AWS アカウント での IAM ユーザーの作成 - AWS Identity and Access Management](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_users_create.html)<br>
参考日: 2023.3.18

*※ 最低限アタッチするポリシーは以下の通りである*

- AdministratorAccess
- AWSIoTFullAccess
- AWSIoTAnalyticsFullAccess
- AWSQuickSightIoTAnalyticsAccess

****
## 5. AWS IoT Coreの設定

*設定前の必要事項*

- リージョンは「バージニア北部（us-east-1）」
- 接続するRaspberry Piが１台のみである
- Raspberry Piに「git」をインストール
- Raspberry Piに「Python（ver3.6以上）」をインストール

### 5.1 AWSとRaspberry Piを接続する

#### ① AWSマネジメントコンソールにログインし、AWS IoT Coreを開く。

![1](https://user-images.githubusercontent.com/91422957/226090789-28dfe43d-ba57-4555-9fda-b0b2b6a23519.png)

#### ② 「１個のデバイスを接続」をクリック。

![2](https://user-images.githubusercontent.com/91422957/226090807-f49c3cf3-fd34-497c-a6c9-9d3499661f8d.png)


#### ③ 「デバイスを準備する」の内容に従い、完了したら「次へ」をクリック。

![3](https://user-images.githubusercontent.com/91422957/226090826-87edf9d2-1fb7-4079-85f7-fe0b22e5106f.png)

#### ④ 「新しいモノを作成」を選択し、「モノの名前」を記入する。完了したら「次へ」をクリック。

![4](https://user-images.githubusercontent.com/91422957/226090852-b5b5677b-d72c-4d32-8d1b-80cd647b6e31.png)

#### ⑤ プラットフォームOSで「Linux/MacOS」、SDKで「Python」を選択し、「次へ」をクリック。

![5](https://user-images.githubusercontent.com/91422957/226090867-d1535808-0e82-423d-8cd1-eea753f1edae.png)

#### ⑥ 作成された接続キットをRaspberry Piにダウンロードし解凍する（接続キットはここでしか入手できない）。完了したら「次へ」をクリック。

![6](https://user-images.githubusercontent.com/91422957/226090880-48a55706-3d3b-4b7c-9144-6a70feeb2dbf.png)

#### ⑦ ダウンロードした接続キット内の「start.sh」に権限を付与し、実行する。接続成功するとサブスクリプションの部分に「Hello World!」と表示される。完了したら「次へ」をクリック。

![7](https://user-images.githubusercontent.com/91422957/226090893-aa09eb46-cefc-4dda-bce1-6e64518ed249.png)


#### ⑧ デバイスがAWS IoT Coreに接続されていることを確認したら「モノを表示」をクリック。

![8](https://user-images.githubusercontent.com/91422957/226090909-3729a5e1-677a-4e50-8cd4-af4041fcb8f2.png)


#### ⑨ 左サイドメニューの「"すべてのデバイス"→"モノ"」のページに遷移し、先ほど記入した名前のモノが表示される。

![9](https://user-images.githubusercontent.com/91422957/226165472-7b9a6a1b-3bf5-4986-a97e-e5c98b69e98c.png)

### 5.2 証明書とポリシーの確認

- 証明書
> 一部の AWS 製品が AWS アカウント およびユーザーの認証に使用する認証情報。別名 X.509 証明書 です。証明書はプライベートキーと組み合わせられます。

- ポリシー
> ポリシーでは通常、特定のアクションへのアクセスを許可し、オプションで、それらのアクションを EC2 インスタンスや Amazon S3 バケットなどの特定のリソースで実行することを許可することができます。また、ポリシーにより、アクセスを明示的に拒否することもできます。

参考:

> **docs.aws.amazon.com**<br>
> [AWS 用語集 - AWS 全般のリファレンス](https://docs.aws.amazon.com/ja_jp/general/latest/gr/glos-chap.html#C)<br>
参考日: 2023.3.19
****

#### ① 先ほど作成した「モノ」のページから、「証明書」をクリックする。

![10](https://user-images.githubusercontent.com/91422957/226165550-a3b11cd1-d8ed-49f8-a794-d5f178eb45e2.png)


#### ② 「証明書ID」をクリックする。

![11](https://user-images.githubusercontent.com/91422957/226165574-97e762b1-b491-4a15-996f-160ebcf24972.png)


#### ③ 証明書に「*"モノの名前"* - Policy」と記載された「ポリシー」がアタッチされていることを確認する。

![12](https://user-images.githubusercontent.com/91422957/226165590-9eb4f0c1-2f04-4634-94f9-9deb02702d24.png)


#### ④ 証明書に「モノ」がアタッチされていることを確認する。

![13](https://user-images.githubusercontent.com/91422957/226165607-5a883119-e226-4640-8fae-388f3face08b.png)

****
## 6. "aws-iot-device-sdk-python-v2"の置換
「5. AWS IoT Coreのセットアップ」でダウンロードした「connect_device_package」内の「aws-iot-device-sdk-python-v2」を、「3. リポジトリのインストール」でインストールした「aws-iot-device-sdk-python-v2」に置換する。

置換後、前者は削除する。


****
## 7. start.shの修正

「5. AWS IoT Coreのセットアップ」でダウンロードした「connect_device_package」内の「start.sh」を修正する。

以下の記述（"A"とする)を次のように修正する。

*before*
```
python3 aws-iot-device-sdk-python-v2/samples/pubsub.py --endpoint example.iot.us-east-1.amazonaws.com --ca_file root-CA.crt --cert example.cert.pem --key example.private.key --client_id basicPubSub --topic sdk/test/Python --count 0
```

*after*
```
until A;do sleep 0;done
```

修正後、「3. リポジトリのインストール」でインストールした「connect_device_package」内の「start.sh」は削除する。

****
## 8. cronでスケジュール管理
- cron
> cronとは、多くのUNIX系OSで標準的に利用される常駐プログラム（デーモン）の一種で、利用者の設定したスケジュールに従って指定されたプログラムを定期的に起動してくれるもの。

参考:<br>
**e-words.jp**
> [cronとは - 意味をわかりやすく - IT用語辞典 e-Words](https://e-words.jp/w/cron.html)<br>
参考日: 2023.3.19


### ログファイルを作成する
cronの実行結果を確認するため、正常終了時とエラー発生時の２つのログファイルを作成する。

#### ① 正常終了時用ログファイルを作成する。
```
$ touch /var/log/example.log
```

#### ② エラー発生時用ログファイルを作成する。

```
$ touch /var/log/example_error.log
```

#### ③ 作成したファイルに実行権限を与える。
```
$ sudo chmod 777 /var/log/example.log
```

```
$ sudo chmod 777 /var/log/example_error.log
```


### crontabを編集する
#### ① crontabを開く。
```
$ crontab -e
```

#### ② nanoが開くので、末尾の記述を削除し以下を記載する。
```
SHELL=/bin/bash
LAUGUAGE=ja_jp.UTF-8
USER=Raspberry Piのユーザー名


# start.shを実行し、AWSに計測データを送信する
# 同時に２種類のログを取る
# 10分毎に計測する場合

*/10 * * * * cd connect_device_packageの絶対パス; ./start.sh >> /var/log/example.log 2>> /var/log/eample_error.log
```

時間指定についての補足:

**qiita.com**
> [cron を使用した時間指定 - Qiita](https://qiita.com/nemutas/items/3f5816eabbf0eda5e6a9)<br>
> 参考日: 2023.3.19

#### ③ AWS IoT Coreの「MQTTテストクライアント」を開き、MQTTトピック「sdk/test/python」を入力してサブスクライブする。cronが正常に動作していたら、データが送信される。

![14](https://user-images.githubusercontent.com/91422957/229339575-1434833b-a5e0-4133-a4a4-46b41a8030a5.png)


****

以上で開発チュートリアル①は完了です!<br>
”試しにAWSに計測データを送ってみたい”といった目的であれば、この段階で完了です。<br>

次はAWS IoT Coreに送信したデータをAWS IoT Analyticsで加工、保存、データセット作成を行い、AmazonQuickSightでグラフ表示します。

### 「開発チュートリアル②」に進みましょう！