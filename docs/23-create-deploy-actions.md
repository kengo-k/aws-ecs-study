# 自動デプロイ用のワークフローの実装

下記の処理を行うワークフローを作成する

- ソースコードを変更した際に新しいイメージを作成しECRにPUSHする
- 新しいイメージを元にタスクを更新してサービスをデプロイする

## 事前準備

手動で作成したECRイメージ(webapp, backend, restapi)は削除しておく。

※スクショは省略

## ワークフローの作成

下記の内容でワークフローファイルを作成する

```
name: Deploy work flow for Amazon ECS

on:
  pull_request:
    branches:
      - main
    types: [closed]

env:
  AWS_REGION: ap-northeast-1
  ECS_WEBAPP_SERVICE: ecs-sample-webapp
  ECS_RESTAPI_SERVICE: ecs-sample-restapi
  WEBAPP_CONTAINER_NAME: ecs-sample-webapp
  BACKEND_CONTAINER_NAME: ecs-sample-backend
  RESTAPI_CONTAINER_NAME: ecs-sample-restapi
  WEBAPP_TAG: 1.0.0
  BACKEND_TAG: 1.0.0
  RESTAPI_TAG: 1.0.0
  SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}


jobs:
  deploy:
    name: deploy
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: webapp push to ECR
      id: build-image-webapp
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
      run: |
        cd sample-webapp
        docker build -t $ECR_REGISTRY/$WEBAPP_CONTAINER_NAME:$WEBAPP_TAG .
        docker tag $ECR_REGISTRY/$WEBAPP_CONTAINER_NAME:$WEBAPP_TAG $ECR_REGISTRY/$WEBAPP_CONTAINER_NAME:latest
        docker push $ECR_REGISTRY/$WEBAPP_CONTAINER_NAME:$WEBAPP_TAG
        docker push $ECR_REGISTRY/$WEBAPP_CONTAINER_NAME:latest

    - name: backend push to ECR
      id: build-image-backend
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
      run: |
        cd sample-backend
        docker build -t $ECR_REGISTRY/$BACKEND_CONTAINER_NAME:$BACKEND_TAG .
        docker tag $ECR_REGISTRY/$BACKEND_CONTAINER_NAME:$BACKEND_TAG $ECR_REGISTRY/$BACKEND_CONTAINER_NAME:latest
        docker push $ECR_REGISTRY/$BACKEND_CONTAINER_NAME:$BACKEND_TAG
        docker push $ECR_REGISTRY/$BACKEND_CONTAINER_NAME:latest

    - name: restapi push to ECR
      id: build-image-restapi
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
      run: |
        cd sample-restapi
        docker build -t $ECR_REGISTRY/$RESTAPI_CONTAINER_NAME:$RESTAPI_TAG .
        docker tag $ECR_REGISTRY/$RESTAPI_CONTAINER_NAME:$RESTAPI_TAG $ECR_REGISTRY/$RESTAPI_CONTAINER_NAME:latest
        docker push $ECR_REGISTRY/$RESTAPI_CONTAINER_NAME:$RESTAPI_TAG
        docker push $ECR_REGISTRY/$RESTAPI_CONTAINER_NAME:latest

    - name: Deploy to ECS
      run: |
        aws ecs update-service --service $ECS_WEBAPP_SERVICE --cluster ecs-lesson-cluster --force-new-deployment
        aws ecs update-service --service $ECS_RESTAPI_SERVICE --cluster ecs-lesson-cluster --force-new-deployment

    - name: Success
      uses: rtCamp/action-slack-notify@v2.0.2
      if: success()
      env:
        SLACK_TITLE: Deploy flow / Success
        SLACK_COLOR: good
        SLACK_MESSAGE: Message for success

    - name: Failure
      uses: rtCamp/action-slack-notify@v2.0.2
      if: failure()
      env:
        SLACK_TITLE: Deploy flow / Failure
        SLACK_COLOR: danger
        SLACK_MESSAGE: Message for failure

    - name: Cancel
      uses: rtCamp/action-slack-notify@v2.0.2
      if: cancelled()
      env:
        SLACK_TITLE: Deploy flow / cancel
        SLACK_COLOR: danger
        SLACK_MESSAGE: Message for Cancel
```