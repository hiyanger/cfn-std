■設計
※パラメータはparameters.yaml的な感じで用意すること

①iam.yaml：IAMユーザー、IAMグループ★

IAMのスタック作成時はオプション
--capabilities CAPABILITY_NAMED_IAM
が必要

https://www.kabegiwablog.com/entry/2019/02/21/140000

②network.yaml：VPC、サブネット（Pub2、Pri1）、ルートテーブル、IGW、NGW★
aws cloudformation deploy --stack-name network-stack --template-file network.yaml --parameter-overrides $(cat parameters_network.yaml)
https://zenn.dev/tmasuyama1114/articles/aws-cloudformation-basics
フローログ https://awstut.com/2021/12/25/vpc-flowlog/

④s3.yaml:S3★
ネストさせてる
aws s3 cp S3_BucketPolicy s3://policy-cfn-20230612-1510/S3_BucketPolicy/ --recursive
aws cloudformation deploy --stack-name s3-stack --template-file s3.yaml --parameter-overrides $(cat parameters_s3.yaml)
https://qiita.com/yoshiokaCB/items/f8c36a6fe250856fd672
https://dev.classmethod.jp/articles/cloudformation-s3bucket-type/
cfnバケポリ https://dev.classmethod.jp/articles/multi-access-restricted-s3-cfn/

③ec2.yaml:ALB、EC2、SG★
aws s3 cp S3_BucketPolicy s3://policy-cfn-20230612-1510/S3_BucketPolicy/ --recursive
aws cloudformation deploy --stack-name ec2-stack --template-file ec2.yaml --parameter-overrides $(cat parameters/parameters_ec2.yaml) --capabilities CAPABILITY_NAMED_IAM
ALB https://dev.classmethod.jp/articles/alb-cfn-template-to-study/
バケポリ https://dev.classmethod.jp/articles/alb-access-log-permission-error-s3-bucket/
下位テンプレートからの出力受取 https://qiita.com/tyoshitake/items/ff08855d629ba4097125
CWlogs設定 https://business.ntt-east.co.jp/content/cloudsolution/column-try-28.html

cloudwatch.yaml：CW、kinesis、SNS、Health（ネストしてもいいかも）
aws s3 cp IAMRole s3://policy-cfn-20230612-1510/IAMRole/ --recursive
aws cloudformation deploy --stack-name cloudwatch-stack --template-file cloudwatch.yaml --parameter-overrides $(cat parameters/parameters_cloudwatch.yaml) --capabilities CAPABILITY_NAMED_IAM
ALBアクセスログ https://www.shigemk2.com/entry/2021/12/23/203333
CloudWatch S3 https://qiita.com/rhino_10/items/6a710c1d29e7088ea6a0

■考慮が必要
・ネストどっかで使いたい:S3はネスト
・リソース名は実用性もたせてスタック名にする？→あとで書き出して考える
・S3のポリシーまわりいる？
・スタック更新時の動作確認（EC2消えないか、一回AWSコンソールでさわるとどうなるか）
　→cfnで作成したリソースは編集されると消せない
・S3バケット削除が手間 DeletionPolicy で Retain？ https://inokara.hateblo.jp/entry/2018/06/03/014139

■その他メモ
・refとsub
https://tycoh.hatenablog.com/entry/2019/06/03/172849


6/13 ALBアクセスログ
ec2.yaml 60 ネストしてるせいでバケット名とれない。どうするか検討。