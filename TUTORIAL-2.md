# 開発チュートリアル②


## 目次

9. 確認事項
10. AWS IoT Analyticsの設定
11. Amazon QuickSightの設定
12. 更新スケジュールの設定
13. 終わりに

※ ファイル左上の「:=」のようなアイコンに各セクションのリンクがあります。


## 9. 確認事項

*チュートリアルを進める前に、以下の事項を確認*

- リージョンは「**バージニア北部（us-east-1）**」

→ 最もAWSツールの利用料が安いリージョン。アジアパシフィック（東京）はAmazon QuickSightがうまく機能しないため注意(2023.3.31 現在)。

- AWSに接続するRaspberry Piが**１台**のみ

****
## 10. AWS IoT Analyticsの設定

*AWS IoT Analyticsを構成するコンポーネントの説明*

- チャネル：AWS IoT Coreに送信されたメッセージを収集する。AWS IoT Analyticsの玄関。

- パイプライン：チャネルに到達したメッセージの加工や属性の計算、フィルタリングができる。チャネルとデータストアを接続している。

- データストア：パイプラインから来たメッセージを保存する。

- データセット；データストアに保存されている情報をもとに、SQLクエリによってデータセット（本チュートリアルでは表形式）を作成する。

- rule（ルール）:AWSリソースに関する一連の条件のこと。

- role（ロール）：AWS アカウントの AWS リソースへの一時的なアクセスを付与するツール。


### 10.1 各コンポーネントの一括設定

#### ① AWSマネジメントコンソールにログインし、AWS IoT Analyticsを開く。

![15](https://user-images.githubusercontent.com/91422957/229293515-2548602e-fd2e-4e78-bb3e-47a4d4f2a93c.png)


#### ② リソースプレフィックス（各コンポーネントの接頭語）とMQTTトピック「sdk/test/python」を入力する。その後、「リソースを作成」をクリック。

![16](https://user-images.githubusercontent.com/91422957/229293586-546f6e21-f235-4bdb-8ed5-3f7393a59c93.png)


#### ③ 各コンポーネントが一括で作成される。

![17](https://user-images.githubusercontent.com/91422957/229293906-a0102a2c-bd8a-487c-9622-a0bfcc4890c8.png)


#### ④ チャネル、パイプライン、データストア、データセットが作成されたか確認する（roleは「IAM」のコンソール画面で確認できる）。

![18](https://user-images.githubusercontent.com/91422957/229293946-4f3ce32f-58ab-404a-b594-7069219810ae.png)

![19](https://user-images.githubusercontent.com/91422957/229294319-9af9b5f2-011c-482b-a491-a7871876f92d.png)

![20](https://user-images.githubusercontent.com/91422957/229294339-7dae527d-be93-46de-9ed2-785feafa62ff.png)

![21](https://user-images.githubusercontent.com/91422957/229294424-37c44eda-24f3-477b-9fcf-ca3db7a73d84.png)


### 10.2 コンテンツの作成とデータ抽出設定

#### ① 作成された「データセット」をクリックし、「コンテンツ」をクリック。

![22](https://user-images.githubusercontent.com/91422957/229294567-e50b12a6-b624-4490-a8e4-4ede4a95f134.png)


#### ② データストアにメッセージ（データ）が到達したことを確認して、「今すぐ実行」をクリック。

![23](https://user-images.githubusercontent.com/91422957/229294677-5ac6c747-b981-4716-bae3-548cb7379bf4.png)


#### ③ あらかじめ設定されているSQLクエリが、データストアからデータを抽出してコンテンツ(csvファイル)を作成する。成功すると画面上に緑のポップアップが表示される。失敗の場合は赤いポップアップが表示される。その後、作成されたコンテンツをクリック。

![24](https://user-images.githubusercontent.com/91422957/229294773-4ae56aa1-829d-4f16-82f9-e7d60672822c.png)


#### ④ 作成されたコンテンツのプレビューが見れる。結果を全て見たい場合は「ダウンロード」をクリックする。

![25](https://user-images.githubusercontent.com/91422957/229295161-35d9f630-1d85-4366-a94c-ec57087841e2.png)


#### ⑤ データセットの「詳細」画面に戻り、SQLクエリの「編集」をクリック。

![26](https://user-images.githubusercontent.com/91422957/229295183-6a8f5d96-57ca-41be-b79c-3288cd026c0b.png)


#### ⑥ SQLクエリを修正する。「テストクエリ」をクリックしてクエリが正常に動くか確認し、「更新」をクリック。

![27](https://user-images.githubusercontent.com/91422957/229295393-72b427a8-8aa4-4cce-9b30-c2aa7c2e2f28.png)

****
## 11. Amazon QuickSightの設定

### 11.1 Amazon QuickSightへのサインアップ

#### ① AWSマネジメントコンソールにログインし、Amazon QuickSightを開く。

![28](https://user-images.githubusercontent.com/91422957/229334698-701db0ea-dd46-419a-8b93-78b659b4467b.png)



#### ② 別途Amazon QuickSightのアカウントを作る必要があるため、サインアップする。

![29](https://user-images.githubusercontent.com/91422957/229334743-866ea1ca-9ff0-4152-912c-0643b4353750.png)


#### ③ プラン、リージョン、アカウント名、メールアドレスを入力する。本チュートリアルでは以下のプラン、リージョンを選択する。

- プラン：エンタープライズ版（スタンダード版は未検証）
- リージョン：米国東部（バージニア北部）

*※各プランによって無料枠や利用料金および機能に違いがあります。詳しくはこちら。*

> **aws.amazon.com**<br>
> [料金 - Amazon QuickSight | AWS](https://aws.amazon.com/jp/quicksight/pricing/)<br>
参考日: 2022.12.22


#### ④ Amazon QuickSightに「AWS IoT Analytics」のアクセス権を追加する。これでサインアップが完了する。

![30](https://user-images.githubusercontent.com/91422957/229334769-b13f2cd5-f74e-4d23-91d7-339c55263e25.png)


### 11.2 データセットの追加とビジュアルの作成

#### ① サインアップが完了するとAmazon QuickSightのダッシュボードが表示される。続いて「新しい分析」をクリック。

![31](https://user-images.githubusercontent.com/91422957/229334791-321e36ba-2b38-493d-b17f-e892114aa649.png)


#### ② 「新しいデータセット」をクリック。

![32](https://user-images.githubusercontent.com/91422957/229334818-84dfc867-934a-4d70-a892-7810963fddc3.png)


#### ③ 「AWS IoT Analytics」を選択。

![33](https://user-images.githubusercontent.com/91422957/229334873-55d2681b-cf6b-4a22-a71a-68438f8ea5c4.png)


#### ④ 可視化したいAWS IoT Analyticsのデータセットを選択し、「データソースを作成」をクリック。

![34](https://user-images.githubusercontent.com/91422957/229334914-946cea63-0645-4913-a34b-a6b888b6a9f4.png)


#### ⑤ 正常にデータセットが作成されたら、「視覚化する」」をクリック。
※SPICE（＝Amazon QuickSightのインメモリエンジン、ストレージのようなもの）の容量が足りない時はエラーが発生します。その場合はSPICEの容量を追加購入してください。

![35](https://user-images.githubusercontent.com/91422957/229334973-94a53530-5efc-4198-a383-add3eeb55a7a.png)


#### ⑥　分析画面に遷移し、ビジュアル（グラフ）が作成できる。

![36](https://user-images.githubusercontent.com/91422957/229334990-5da1b0bc-346a-4412-82f8-402ce0590380.png)


#### ⑦ 好みの設定でビジュアルを作成、編集する。

![37](https://user-images.githubusercontent.com/91422957/229335210-2fc65afb-37c0-4393-9479-389ca8c38b2f.png)


*Amazon QuickSightやビジュアル作成方法の詳しい内容はこちら*
> **docs.aws.amazon.com**<br>
> [Amazon QuickSight でのデータの視覚化 - Amazon QuickSight](https://docs.aws.amazon.com/ja_jp/quicksight/latest/user/working-with-visuals.html)<br>
参考日: 2023.3.31

****
## 12. 更新スケジュールの設定

*データセットはデフォルトでは自動的に更新されない。手動でも更新は可能だが、自動化のためにデータセットの更新スケジュールを設定する。*

### 12.1 AWS IoT Analytics側データセットの更新スケジュール

#### ①　 AWS IoT Analyticsで作成したデータセットのスケジュール画面を開き、「編集」をクリック。

![38](https://user-images.githubusercontent.com/91422957/229338177-9999ad57-1aef-45d4-9c0e-0595a4b0582d.png)


#### ② 更新の頻度を選択する。選択したい頻度がリストにあればそのまま「更新」をクリック。

![39](https://user-images.githubusercontent.com/91422957/229338222-821a08e3-0f76-4382-bd9b-50255a3e10c1.png)


#### ③ cronでスケジュール管理をするため、好みの時間を入力し「更新」をクリック。

![40](https://user-images.githubusercontent.com/91422957/229338248-dca6009e-b81f-4602-9dda-0432252da9bb.png)


#### ④ 更新スケジュールがcron式で設定される。
![41](https://user-images.githubusercontent.com/91422957/229338360-edbc723f-66ee-474f-b9ae-cfebd9832942.png)



### 12.2 Amazon QuickSight側データセットの更新スケジュール

#### ① Amazon QuickSightのダッシュボードを開き、画面左の「データセット」をクリック。

![42](https://user-images.githubusercontent.com/91422957/229338441-f286f537-5214-4917-beac-728178c5f038.png)


#### ② 更新スケジュールを設定したいデータセットをクリックする。

![43](https://user-images.githubusercontent.com/91422957/229338569-e20d273f-6530-4fed-ab3d-d5222c1fb832.png)


#### ③ 「更新」をクリック。

![44](https://user-images.githubusercontent.com/91422957/229338625-c5051660-3fed-436b-a0f2-a92a6e466c5f.png)


#### ④ 「新しいスケジュールの追加」をクリック。

![45](https://user-images.githubusercontent.com/91422957/229338679-0024adb9-6d68-43bf-8cc6-3b0fc8cda994.png)


#### ⑤ タイムゾーンを"Asia/Tokyo"、スケジュールの開始時刻と頻度を設定する。頻度は最短でも「毎時」のため注意。その後「保存」をクリック。

![46](https://user-images.githubusercontent.com/91422957/229338725-38a77ec7-ece5-4d78-bb0a-e97641b71cf7.png)


#### ⑥ 更新スケジュールが設定される。

![47](https://user-images.githubusercontent.com/91422957/229338842-2372b6b1-edcf-45bc-82eb-9f97fe097e3a.png)

****
## 13. 終わりに

以上で全ての開発チュートリアルは完了です！<br>
お疲れ様でした！

ラズパイやセンサー、AWSを使った農業IoTの開発や、その他研究活動などの参考となれば幸いです。

ご不明な点、誤っている点等ございましたら、「README.md」に記載の連絡先やGitリポジトリのIssuesにてお問い合わせください。

ありがとうございました。