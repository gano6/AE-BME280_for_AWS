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


## 開発環境および使用サービス
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
- AWS IoT Analytics
- Amazon QuickSight

*※ AWSアカウントおよびIAMユーザーの作成が必要*

### VScode
ラズパイとのSSH接続、プログラミング用で使用