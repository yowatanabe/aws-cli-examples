# Create network ACL to deny all

- テスト等で擬似的に障害を起こすために全ての通信を拒否するネットワークACLを一時的に作りたい場合に使用
- 手順書内の手順としてテスト用ネットワークACLを作成する時に使用すると手順書が多少スッキリするはず。普段ちょっと試す程度ならマネジメントコンソールからポチポチしたほうが早いかもしれない。

## 前提
- AWS CloudShellまたはMacでの利用を想定
- 各コマンドの以下の変数は利用環境に応じて設定する

|変数|説明|
|---|---|
|NETWORK_ACL_NAME|作成するネットワークACL名を指定する|
|TARGET_VPC_NAME|ネットワークACLを作成するVPCのNameタグの値を指定する|


## コマンド

1. 作成

```bash
NETWORK_ACL_NAME=test-nacl && \
TARGET_VPC_NAME=demo-vpc && \
TARGET_VPC_ID=$(aws ec2 describe-vpcs --query "Vpcs[?Tags[?Key==\`Name\`].Value|[0]==\`${TARGET_VPC_NAME}\`].VpcId" --output text) && \
aws ec2 create-network-acl --tag-specifications "ResourceType=network-acl,Tags=[{Key=Name,Value=${NETWORK_ACL_NAME}}]" --vpc-id ${TARGET_VPC_ID}
```

1. 削除

```bash
NETWORK_ACL_NAME=test-nacl && \
TARGET_NACL_ID=$(aws ec2 describe-network-acls --query "NetworkAcls[?Tags[?Key==\`Name\`].Value|[0]==\`${NETWORK_ACL_NAME}\`].NetworkAclId" --output text) && \
aws ec2 delete-network-acl --network-acl-id ${TARGET_NACL_ID} && \
aws ec2 describe-network-acls --query "NetworkAcls[?Tags[?Key==\`Name\`].Value|[0]==\`${NETWORK_ACL_NAME}\`]"
```
