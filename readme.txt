【ハンズオン】AWS CloudFormationで作るログ運用と監視システム
https://zenn.dev/hiyanger/books/6d510a8f66093a

パラメータ
https://docs.google.com/spreadsheets/d/14I7IyAuSmmkQ5u-7lDA_6KAGNicgFs7y-U6-5Ao0cO4/edit#gid=0

■設計
※パラメータはparameters.yaml的な感じで用意すること

①iam.yaml：IAMユーザー、IAMグループ★
aws cloudformation deploy --stack-name iam-stack --template-file iam.yaml --parameter-overrides $(cat parameters.yaml) --capabilities CAPABILITY_NAMED_IAM

IAMのスタック作成時はオプション
--capabilities CAPABILITY_NAMED_IAM
が必要

https://www.kabegiwablog.com/entry/2019/02/21/140000

②lambda
スタック削除するときにオブジェクトファイルも一緒に削除する
aws cloudformation deploy --stack-name lambda-stack --template-file lambda.yaml --parameter-overrides $(cat parameters.yaml) --capabilities CAPABILITY_NAMED_IAM
https://dev.classmethod.jp/articles/custom-resource-empty-s3-objects/
https://atsushinotes.com/deploy_lambda_from_cloudformation-by-yaml/#index_id10
https://www.yokoyan.net/entry/2019/01/22/181500
https://dev.classmethod.jp/articles/note-about-using-delete-response-with-cfn-response-module-in-lambda-backed-custom-resource/#toc-1

②network.yaml：VPC、サブネット（Pub2、Pri1）、ルートテーブル、IGW、NGW★
aws cloudformation deploy --stack-name network-stack --template-file network.yaml --parameter-overrides $(cat parameters.yaml)

https://zenn.dev/tmasuyama1114/articles/aws-cloudformation-basics
フローログ https://awstut.com/2021/12/25/vpc-flowlog/

③ec2.yaml:ALB、EC2、SG★
aws s3 cp s3 s3://policy-cfn-20230612-1510/S3_BucketPolicy/ --recursive
aws cloudformation deploy --stack-name ec2-stack --template-file ec2.yaml --parameter-overrides $(cat parameters.yaml) --capabilities CAPABILITY_NAMED_IAM

ALB https://dev.classmethod.jp/articles/alb-cfn-template-to-study/
バケポリ https://dev.classmethod.jp/articles/alb-access-log-permission-error-s3-bucket/
下位テンプレートからの出力受取 https://qiita.com/tyoshitake/items/ff08855d629ba4097125
CWlogs設定 https://business.ntt-east.co.jp/content/cloudsolution/column-try-28.html https://zenn.dev/kanoe/articles/0ed13385a4982b

④cloudwatch_s3.yaml：CW、kinesis、SNS、Health（ネストしてもいいかも）
aws s3 cp iamrole s3://policy-cfn-20230612-1510/IAMRole/ --recursive
aws cloudformation deploy --stack-name cloudwatch-s3-stack --template-file cloudwatch-s3.yaml --parameter-overrides $(cat parameters.yaml) --capabilities CAPABILITY_NAMED_IAM

ALBアクセスログ https://www.shigemk2.com/entry/2021/12/23/203333
CloudWatch S3 https://qiita.com/rhino_10/items/6a710c1d29e7088ea6a0

④cloudwatch_health.yaml：CW、kSNS、Health（ネストしてもいいかも）
検知アラート：ステータスチェック（インスタンス停止）★ / ログのerror、Error、ERROR / AWSメンテ
aws cloudformation deploy --stack-name cloudwatch-health-stack --template-file cloudwatch-health.yaml --parameter-overrides $(cat parameters.yaml)

SNS https://qiita.com/takachan/items/c37455a610dc928aec0a
Metric https://engineers.weddingpark.co.jp/aws-cloudformation-cloudwatch-alarm/
メトリクスフィルター https://blog.serverworks.co.jp/tech/2020/05/11/post-84287/
Health https://zenn.dev/mn87/articles/832a31fff0e2ab

ステータスチェックのテスト sudo ifdown eth0

■確認したいこと
・スタック更新時の動作確認（EC2消えないか、一回AWSコンソールでさわるとどうなるか）★
　- cfnで作成したリソースはコンソールで編集されると消せない
　- 上書き更新は可能
  - 同じyaml内の一部リソースは変更しても、変更されていない他のリソースは再作成されない。
・S3バケット削除が手間 DeletionPolicy で Retain？ https://inokara.hateblo.jp/entry/2018/06/03/014139 ★Lambdaのカスタムリソースで対処
・スタックの削除でSNSが消えない★
・関数まとめ　ref sub getatt importvalue★
・deleteとdeploy同時にできないか→シェルでコマンドつなげるしかないっぽい★
・リソース名精査
・コード精査（不明なパラメータなどコメント）

■その他メモ
・refとsub
https://tycoh.hatenablog.com/entry/2019/06/03/172849

・subとimportvalueの結合
ServiceToken: {Fn::ImportValue: !Sub "${sysname}-s3delete-lambda-function"}
https://qiita.com/3244/items/b03b60fb82081aada617
https://repost.aws/ja/knowledge-center/cloudformation-fn-sub-function
https://intothelambda.com/blog/docker-rec-radiko-version2/

・Former2 既存コードの出力
https://dev.classmethod.jp/articles/former2/

・内部エラー
発生したら繰り返し実行すればできるようになる（バグっぽい？）
Resource handler returned message: "An internal error has occurred (Service: Ec2, Status Code: 500, Request ID: 0f1f02e8-5597-4b3f-b912-af38b5afbb8e)" (RequestToken: 0cde47fe-7e65-5cdd-855c-afbb1893f86e, HandlerErrorCode: GeneralServiceException)
