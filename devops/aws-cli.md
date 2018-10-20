# AWS Cli

configure

```bash
$ aws configure
```



```bash

aws deploy get-deployment-group --region=ap-northeast-2 --application-name=sre-codedeploy-test --deployment-group-name=sre-bluegreen-test --output=text | grep "AUTOSCALINGGROUPS" | awk {print $3} > commandResult

```

