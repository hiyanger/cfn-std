■設計
※パラメータはparameters.yaml的な感じで用意すること

①iam.yaml：IAMユーザー、IAMグループ★

IAMのスタック作成時はオプション
--capabilities CAPABILITY_NAMED_IAM
が必要

https://www.kabegiwablog.com/entry/2019/02/21/140000

②network.yaml：VPC、サブネット（Pub2、Pri1）、ルートテーブル、IGW、NGW★
aws cloudformation deploy --stack-name network-stack --template-file network.yaml --parameter-overrides $(cat parameters/parameters_network.yaml)

https://zenn.dev/tmasuyama1114/articles/aws-cloudformation-basics
フローログ https://awstut.com/2021/12/25/vpc-flowlog/

③ec2.yaml:ALB、EC2、SG★
aws s3 cp s3 s3://policy-cfn-20230612-1510/S3_BucketPolicy/ --recursive
aws cloudformation deploy --stack-name ec2-stack --template-file ec2.yaml --parameter-overrides $(cat parameters/parameters_ec2.yaml) --capabilities CAPABILITY_NAMED_IAM

ALB https://dev.classmethod.jp/articles/alb-cfn-template-to-study/
バケポリ https://dev.classmethod.jp/articles/alb-access-log-permission-error-s3-bucket/
下位テンプレートからの出力受取 https://qiita.com/tyoshitake/items/ff08855d629ba4097125
CWlogs設定 https://business.ntt-east.co.jp/content/cloudsolution/column-try-28.html

④cloudwatch_s3.yaml：CW、kinesis、SNS、Health（ネストしてもいいかも）
aws s3 cp iamrole s3://policy-cfn-20230612-1510/IAMRole/ --recursive
aws cloudformation deploy --stack-name cloudwatch-s3-stack --template-file cloudwatch_s3.yaml --parameter-overrides $(cat parameters/parameters_cloudwatch_3.yaml) --capabilities CAPABILITY_NAMED_IAM

ALBアクセスログ https://www.shigemk2.com/entry/2021/12/23/203333
CloudWatch S3 https://qiita.com/rhino_10/items/6a710c1d29e7088ea6a0

④cloudwatch_health.yaml：CW、kSNS、Health（ネストしてもいいかも）
検知するアラートはEC2ダウンとログのerror、Error、ERRORのみ。
aws cloudformation deploy --stack-name cloudwatch-health-stack --template-file cloudwatch_health.yaml --parameter-overrides $(cat parameters_cloudwatch_health.yaml)

SNS https://qiita.com/takachan/items/c37455a610dc928aec0a
Metric https://engineers.weddingpark.co.jp/aws-cloudformation-cloudwatch-alarm/

■考慮が必要
・ネストどっかで使いたい:S3はネスト★
・リソース名は実用性もたせてスタック名にする？→あとで書き出して考える
・S3のポリシーまわりいる？
・スタック更新時の動作確認（EC2消えないか、一回AWSコンソールでさわるとどうなるか）
　→cfnで作成したリソースは編集されると消せない
・S3バケット削除が手間 DeletionPolicy で Retain？ https://inokara.hateblo.jp/entry/2018/06/03/014139
・関数まとめ　ref sub getatt importvalue

■その他メモ
・refとsub
https://tycoh.hatenablog.com/entry/2019/06/03/172849
・subとimportvalueの結合
https://qiita.com/3244/items/b03b60fb82081aada617
https://repost.aws/ja/knowledge-center/cloudformation-fn-sub-function


6/13 ALBアクセスログ
ec2.yaml 60 ネストしてるせいでバケット名とれない。どうするか検討。