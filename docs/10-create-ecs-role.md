# ECSを操作するためのロールを定義する

## IAMサービスを表示する

![](images/10/01.png)

## メニューより「ロール」を選択する

![](images/10/02.png)

## 「ロールの作成」ボタンをクリックする

![](images/10/03.png)

## 下記の内容を設定し「次へ」ボタンをクリックする

- 信頼されたエンティティタイプ: AWSサービス
- サービス: Elastic Container Service
- ユースケース: Elastic Container Service Task

![](images/10/04.png)

## 下記の内容を設定し「次へ」ボタンをクリックする

- ポリシー名を入力して候補を絞り込む: Amazon ECS
- AmazonECSTaskExecutionRolePolicyをチェックON

![](images/10/05.png)

## 下記の内容を設定し「ロールを作成」ボタンをクリックする

- ロール名: ecsTaskExecutionRole

![](images/10/06.png)