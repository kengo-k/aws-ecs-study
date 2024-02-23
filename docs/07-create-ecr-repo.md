# ECRリポジトリの作成

## Elastic Container Serviceを選択する

![](images/07/01.png)

## メニューからリポジトリを選択する

![](images/07/02.png)

## プライベートリポジトリを選択する

![](images/07/03.png)

## 「リポジトリを作成」ボタンをクリックする

![](images/07/04.png)

## 下記の内容を設定して「リポジトリを作成」ボタンをクリックする

![](images/07/05.png)

## さらに下記の内容を設定して「リポジトリを作成」ボタンをクリックする

![](images/07/06.png)

## 下記の内容を設定して「リポジトリを作成」ボタンをクリックする

![](images/07/07.png)

## 作成したリポジトリの一覧を確認する

この後に使用するコマンドを控えておくために作成したリポジトリを選択する
(ここでは先頭のecs-sample-backendを選択)

![](images/07/08.png)

## 「プッシュコマンドの表示」ボタンをクリックする

![](images/07/09.png)

## 表示されているコマンドをメモしておく

![](images/07/10.png)

### 1

```
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin 155743614390.dkr.ecr.ap-northeast-1.amazonaws.com
```

### 2

```
docker build -t ecs-sample-backend .
```

### 3

```
docker tag ecs-sample-backend:latest 155743614390.dkr.ecr.ap-northeast-1.amazonaws.com/ecs-sample-backend:latest
``` 

### 4

```
docker push 155743614390.dkr.ecr.ap-northeast-1.amazonaws.com/ecs-sample-backend:latest
```