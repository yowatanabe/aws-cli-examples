# Create a security group with no inbound rules

- テスト等で擬似的に障害を起こすためにインバウントルールが無いセキュリティグループを一時的に作りたい場合に使用

## 前提
- AWS CloudShellまたはMacでの利用を想定
- 各コマンドの以下の変数は利用環境に応じて設定する

|変数|説明|
|---|---|
|SECURITY_GROUP_NAME|作成するセキュリティグループ名を指定する|
|TARGET_VPC_NAME|セキュリティグループを作成するVPCのNameタグの値を指定する|


## コマンド

1. 作成

```bash
SECURITY_GROUP_NAME=test-sg && \
TARGET_VPC_NAME=demo-vpc && \
TARGET_VPC_ID=$(aws ec2 describe-vpcs --query "Vpcs[?Tags[?Key==\`Name\`].Value|[0]==\`${TARGET_VPC_NAME}\`].VpcId" --output text) && \
aws ec2 create-security-group --group-name ${SECURITY_GROUP_NAME} --description "Security Group for TEST" --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=${SECURITY_GROUP_NAME}}]" --vpc-id ${TARGET_VPC_ID} && \
aws ec2 describe-security-groups --query "SecurityGroups[?GroupName==\`${SECURITY_GROUP_NAME}\`]" --output yaml
```

2. 削除

```bash
SECURITY_GROUP_NAME=test-sg && \
TARGET_SG_ID=$(aws ec2 describe-security-groups --query "SecurityGroups[?GroupName==\`${SECURITY_GROUP_NAME}\`].GroupId" --output text) && \
aws ec2 delete-security-group --group-id ${TARGET_SG_ID} && \
aws ec2 describe-security-groups --query "SecurityGroups[?GroupName==\`${SECURITY_GROUP_NAME}\`]" --output yaml
```
