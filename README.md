# AE-BME280 for AWS

<img width="1432" alt="Amazon QuickSight（気温）" src="https://user-images.githubusercontent.com/91422957/225275332-7de792fc-4503-4331-bbc0-e8faac83a7a4.png">


## 概要
農業系卒業論文の執筆にあたり開発

### 機能
- 気温、相対湿度の計測（データロガー）
- 計測データをAWSに送信
- 計測データをリアルタイムにグラフ表示

### 開発のゴール
AWSを用いた安価な農業向けIoTクラウドシステムの構築


## 開発環境
### Raspberry Pi 4 Model B
| 項目 | 内容 |
| :---: | :---: |
| CPU | Quad-Core Cortex-A72 |
| OS |  Raspberry Pi OS 64-bit |
| RAM | 4GB |
| 開発言語 | Python 3.7.3 |

### AE-BME280
気温、相対湿度計測用センサーとして使用

### AWS
- AWS IoT Core
- AWS IoT Device SDK v2 for Python
- AWS IoT Analytics
- Amazon QuickSight

*※ "AWSアカウント"および"IAMユーザー"の作成が必要*

### VSCode(Visual Studio Code)
Raspberry PiとのSSH接続、プログラミング用で使用

## アーキテクチャ図
<img width="1021" alt="アーキテクチャ図_AE-BME280" src="https://user-images.githubusercontent.com/91422957/225512322-5068981d-965b-423d-8264-b3aa56aa2530.png">


## 当リポジトリのインストール方法
*開発で当リポジトリを複製、改変する場合はこちらから*<br>
*※ 「connect_device_package」は各個人のAWSアカウントからダウンロードが必要*

#### 方法①
```
git clone https://github.com/gano6/AE-BME280_for_AWS.git
```
*※ "git"のインストールが必要*

#### 方法②
「<span style="color: green; "><> code</span>」→ 「Download ZIP」


## 開発チュートリアル

#### [開発チュートリアル①](./TUTORIAL-1.md)

#### [開発チュートリアル②](./TUTORIAL-2.md)


## 作者<br>
**Reo Matsugano**<br>
e-mail: reo.matsugano@gmail.com

*お問い合わせは上記メールアドレス,またはIssuesまで*


## ライセンス

