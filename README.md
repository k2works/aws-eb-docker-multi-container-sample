# 目的
AWS Elastic BeanstalkのMulti-containerを使ってDockerアプリケーションのステージング・本番環境をセットアップする

# 前提
| ソフトウェア     | バージョン    | 備考         |
|:---------------|:-------------|:------------|
| docker         | 1.10.3       |             |
| docker-compose | 1.6.2        |             |
| vagrant        | 1.7.4        |             |
| aws-cli        | 1.10.17      |             |
| EB CLI         | 3.7.4        |             |

+ [DockerMultiContanierを使ったRailsアプリケーションサンプルのセットアップが完了している](https://github.com/k2works/docker-multi-container-sample)
+ AWSのアカウントを作っている

# 構成

+ AWS環境の設定
+ アプリケーションのデプロイ

# 詳細
## AWS環境の設定
### IAMユーザー・グループの作成
AWSにログインしてIAM管理コンソールに移動する

グループ画面で管理グループ・開発グループを作成する

以下のポリシーをグループごとにアタッチする

+ Administrators
    + AdministratorAccess  
+ Developers
    + AmazonEC2ContainerRegistryFullAccess
    + AmazonEC2ReadOnlyAccess
    + AWSElasticBeanstalkFullAccess
    + AmazonEC2ContainerServiceFullAccess

ユーザー画面で管理ユーザーと開発ユーザーを作成する

ユーザー作成時に生成される認証情報をダウンロードして保存する

+ 管理ユーザーのパスワードを設定する
+ 開発ユーザーのパスワードを設定する
+ 管理ユーザーを管理グループに所属させる
+ 開発ユーザーを開発グループに所属させる

作業が完了したらログアウトして管理ユーザーでIAMユーザーのサインインリンクからログインする

### ElasticBeanstalkのサンプルアプリケーションを作成する
Multi-container Dockerを選択する

サンプルアプリケーションを作成したら認証情報のロールに移動して２件ロールが新規追加されていることを確認する

aws-elasticbeanstalk-ec2-roleロールを選択してポリシーのアタッチからAmazonEC2ContainerServiceforEC2Roleを追加する


### aws-cliツールをセットアップする
以下のコマンドを実行してユーザー作成時にダウンロードした認証情報を登録する
```
$ aws configure
AWS Access Key ID [None]: XXXXXXXXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXXXXXXXXXX
Default region name [None]: ap-northeast-1
Default output format [None]: json
```

### ElasticBeanstalkにアプリケーションをデプロイする
docker-composeを実行する
```
$ docker-compose up
```
http://127.0.0.1:8080/ に接続して動作を確認する  

動作を確認したらCtr-cで終了する

`Dockerrun.aws.json`ファイルを作成する
```
$ cp ../Dockerrun.aws.json .
```

ElasticBeanstalkアプリケーションを作成する
```
$ eb init
Select a default region
1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-southeast-1 : Asia Pacific (Singapore)
7) ap-southeast-2 : Asia Pacific (Sydney)
8) ap-northeast-1 : Asia Pacific (Tokyo)
9) ap-northeast-2 : Asia Pacific (Seoul)
10) sa-east-1 : South America (Sao Paulo)
11) cn-north-1 : China (Beijing)
(default is 3): 8

Select an application to use
1) 初めての Elastic Beanstalk アプリケーション
2) [ Create new Application ]
(default is 2): 2

Enter Application Name
(default is "sample-app"): 
Application sample-app has been created.

It appears you are using Multi-container Docker. Is this correct?
(y/n): y

Select a platform version.
1) Multi-container Docker 1.9.1 (Generic)
2) Multi-container Docker 1.6.2 (Generic)
(default is 1): 1
Do you want to set up SSH for your instances?
(y/n): y

Type a keypair name.
(Default is aws-eb): 
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/vagrant/.ssh/aws-eb.
Your public key has been saved in /home/vagrant/.ssh/aws-eb.pub.
The key fingerprint is:
3f:b3:66:fa:6e:36:21:98:27:15:73:b9:5a:13:ad:68 aws-eb
The key's randomart image is:
+--[ RSA 2048]----+
|           o     |
|        o + .    |
|         = +     |
|        E =      |
|       =So .     |
|      + +..      |
|       o .+.     |
|          *+     |
|        .O+.     |
+-----------------+
WARNING: Uploaded SSH public key for "aws-eb" into EC2 for region ap-northeast-1.
```

アプリケーションをデプロイするステージング環境を作成する
```
$ eb create staging-env
  Application name: sample-app
  Region: ap-northeast-1
  Deployed Version: app-160412_152744
  Environment ID: e-aiaih8xpwm
  Platform: 64bit Amazon Linux 2016.03 v2.1.0 running Multi-container Docker 1.9.1 (Generic)
  Tier: WebServer-Standard
  CNAME: UNKNOWN
  Updated: 2016-04-12 06:27:48.863000+00:00
Printing Status:
INFO: createEnvironment is starting.
INFO: Using elasticbeanstalk-ap-northeast-1-262470114399 as Amazon S3 storage bucket for environment data.
INFO: Created load balancer named: awseb-e-a-AWSEBLoa-1E7M9QKUEILAV
INFO: Created security group named: awseb-e-aiaih8xpwm-stack-AWSEBSecurityGroup-IL3KMMB5GVP
INFO: Created Auto Scaling launch configuration named: awseb-e-aiaih8xpwm-stack-AWSEBAutoScalingLaunchConfiguration-114XT0PO7D80S
INFO: Environment health has transitioned to Pending. Initialization in progress (running for 28 seconds). There are no instances.
INFO: Added instance [i-762b82e9] to your environment.
INFO: Waiting for EC2 instances to launch. This may take a few minutes.
INFO: Created Auto Scaling group named: awseb-e-aiaih8xpwm-stack-AWSEBAutoScalingGroup-1S4J7Z39L4N1A
INFO: Created Auto Scaling group policy named: arn:aws:autoscaling:ap-northeast-1:262470114399:scalingPolicy:ae77efeb-79b4-4589-8b80-119187e4cde2:autoScalingGroupName/awseb-e-aiaih8xpwm-stack-AWSEBAutoScalingGroup-1S4J7Z39L4N1A:policyName/awseb-e-aiaih8xpwm-stack-AWSEBAutoScalingScaleUpPolicy-W3Z6DUS1KQBF
INFO: Created Auto Scaling group policy named: arn:aws:autoscaling:ap-northeast-1:262470114399:scalingPolicy:9115cd43-984c-4aa8-bb68-fb9a31147e5b:autoScalingGroupName/awseb-e-aiaih8xpwm-stack-AWSEBAutoScalingGroup-1S4J7Z39L4N1A:policyName/awseb-e-aiaih8xpwm-stack-AWSEBAutoScalingScaleDownPolicy-1JF8M2SERZGFH
INFO: Created CloudWatch alarm named: awseb-e-aiaih8xpwm-stack-AWSEBCloudwatchAlarmHigh-OB88BQQRRXZA
INFO: Created CloudWatch alarm named: awseb-e-aiaih8xpwm-stack-AWSEBCloudwatchAlarmLow-1H9WDXSGR4E8M
INFO: Starting new ECS task with awseb-staging-env-aiaih8xpwm:1.
INFO: ECS task: arn:aws:ecs:ap-northeast-1:262470114399:task/46a1c2b6-754e-4988-a2d6-b6875e0eae7d is RUNNING.
INFO: Environment health has transitioned from Pending to Ok. Initialization completed 6 seconds ago and took 8 minutes.
INFO: Successfully launched environment: staging-env
                                
Alert: An update to the EB CLI is available. Run "pip install --upgrade awsebcli" to get the latest version.
```

デプロイが完了したらデプロイ先のURLでアプリケーションが動作しているか確認する(CNAMEがアプリケーションのURL)
```
$ eb status
  Environment details for: staging-env
    Application name: sample-app
    Region: ap-northeast-1
    Deployed Version: app-160412_152744
    Environment ID: e-aiaih8xpwm
    Platform: 64bit Amazon Linux 2016.03 v2.1.0 running Multi-container Docker 1.9.1 (Generic)
    Tier: WebServer-Standard
    CNAME: staging-env.4e6ba2nazh.ap-northeast-1.elasticbeanstalk.com
    Updated: 2016-04-12 06:36:42.264000+00:00
    Status: Ready
    Health: Green
  Alert: An update to the EB CLI is available. Run "pip install --upgrade awsebcli" to get the latest version.
```

EC2インスタンスにSSHでつないでコンテナの稼働状況を確認する
```
$ eb ssh
INFO: Attempting to open port 22.
INFO: SSH port 22 open.
The authenticity of host '54.238.115.81 (54.238.115.81)' can't be established.
ECDSA key fingerprint is e4:f8:b6:2b:21:30:a7:49:58:8e:90:86:2d:15:22:c3.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '54.238.115.81' (ECDSA) to the list of known hosts.
 _____ _           _   _      ____                       _        _ _
| ____| | __ _ ___| |_(_) ___| __ )  ___  __ _ _ __  ___| |_ __ _| | | __
|  _| | |/ _` / __| __| |/ __|  _ \ / _ \/ _` | '_ \/ __| __/ _` | | |/ /
| |___| | (_| \__ \ |_| | (__| |_) |  __/ (_| | | | \__ \ || (_| | |   <
|_____|_|\__,_|___/\__|_|\___|____/ \___|\__,_|_| |_|___/\__\__,_|_|_|\_\
                                       Amazon Linux AMI

This EC2 instance is managed by AWS Elastic Beanstalk. Changes made via SSH 
WILL BE LOST if the instance is replaced by auto-scaling. For more information 
on customizing your Elastic Beanstalk environment, see our documentation here: 
http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html
[ec2-user@ip-10-134-139-139 ~]$ sudo docker ps
CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS              PORTS                         NAMES
ade1ad6ec4da        k2works/my-rails-app-nginx:latest   "nginx -g 'daemon off"   6 minutes ago       Up 5 minutes        0.0.0.0:80->80/tcp, 443/tcp   ecs-awseb-staging-env-aiaih8xpwm-1-proxy-baf5eb9d91bf9af7bb01
d30be4d08f5b        k2works/my-rails-app:latest         "rails server -b 0.0."   6 minutes ago       Up 6 minutes        0.0.0.0:3000->3000/tcp        ecs-awseb-staging-env-aiaih8xpwm-1-web-fa98b3f2dc81cea46c00
2051f2a786aa        k2works/my-rails-app-mysql:latest   "docker-entrypoint.sh"   9 minutes ago       Up 9 minutes        3306/tcp                      ecs-awseb-staging-env-aiaih8xpwm-1-db-9adfe6d7a2c8e791d101
3f2745e1f996        amazon/amazon-ecs-agent:latest      "/agent"                 10 minutes ago      Up 10 minutes       127.0.0.1:51678->51678/tcp    ecs-agent
```

続いて本番環境を作成する
```
$ eb create production-env --cname k2works-sample-app
$ eb status production-env
Environment details for: production-env
  Application name: sample-app
  Region: ap-northeast-1
  Deployed Version: app-160412_155204
  Environment ID: e-gsmyfpmcq4
  Platform: 64bit Amazon Linux 2016.03 v2.1.0 running Multi-container Docker 1.9.1 (Generic)
  Tier: WebServer-Standard
  CNAME: k2works-sample-app.ap-northeast-1.elasticbeanstalk.com
  Updated: 2016-04-12 07:03:07.410000+00:00
  Status: Ready
  Health: Green
```

#### 後片付け
```
$ eb terminate --a

The application "sample-app" and all its resources will be deleted.
This application currently has the following:
Running environments: 2
Configuration templates: 0
Application versions: 2

To confirm, type the application name: sample-app
Removing application versions from s3.
INFO: deleteApplication is starting.
INFO: Invoking Environment Termination workflows.
INFO: Deleted CloudWatch alarm named: awseb-e-aiaih8xpwm-stack-AWSEBCloudwatchAlarmLow-1H9WDXSGR4E8M 
INFO: Deleted CloudWatch alarm named: awseb-e-aiaih8xpwm-stack-AWSEBCloudwatchAlarmHigh-OB88BQQRRXZA 
INFO: Environment health has transitioned from Ok to Info. Terminate in progress (running for 7 seconds).
INFO: Environment health has transitioned from Ok to Info. Terminate in progress (running for 33 seconds).
INFO: Deleted Auto Scaling group named: awseb-e-aiaih8xpwm-stack-AWSEBAutoScalingGroup-1S4J7Z39L4N1A
INFO: Deleted Auto Scaling launch configuration named: awseb-e-aiaih8xpwm-stack-AWSEBAutoScalingLaunchConfiguration-114XT0PO7D80S
INFO: Deleted security group named: awseb-e-aiaih8xpwm-stack-AWSEBSecurityGroup-IL3KMMB5GVP
WARN: Environment health has transitioned from Info to Severe. Terminate in progress (running for 2 minutes). None of the instances are sending data.
INFO: Deleting SNS topic for environment staging-env.
INFO: deleteApplication completed successfully.
INFO: Deleted Auto Scaling group named: awseb-e-gsmyfpmcq4-stack-AWSEBAutoScalingGroup-SICLLZTURE55
INFO: Deleted Auto Scaling launch configuration named: awseb-e-gsmyfpmcq4-stack-AWSEBAutoScalingLaunchConfiguration-IKS5DYUQ26UX
INFO: Deleted security group named: awseb-e-gsmyfpmcq4-stack-AWSEBSecurityGroup-1HYCWVJ4JYW05
INFO: Deleted load balancer named: awseb-e-g-AWSEBLoa-1933J3Y9FFCQO
INFO: Deleting SNS topic for environment production-env.
INFO: deleteApplication completed successfully.
INFO: The environment termination step is done.
INFO: The application has been deleted successfully.
                                
$ exit
$ vagrant destroy
```

# 参照

+ [AWS Elastic Beanstalk](http://docs.aws.amazon.com/ja_jp/elasticbeanstalk/latest/dg/Welcome.html)

