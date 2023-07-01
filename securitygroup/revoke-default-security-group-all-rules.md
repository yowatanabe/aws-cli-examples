# Revoke default security group all rules

- SecurityHubの[EC2.2](https://docs.aws.amazon.com/ja_jp/securityhub/latest/userguide/ec2-controls.html#ec2-2)の対策用コマンド
- 商用環境でデフォルトのセキュリティグループを使用することはあまりないのでデフォルトのセキュリティグループのインバウンド/アウトバウンドルールを削除する方針になることが多いが、マネジメントコンソールから削除するのは地味に大変なのでコマンドを作成

## 前提
- AWS CloudShellまたはMacでの利用を想定

## コマンド

1. アカウントで有効になっているリージョンを取得
```bash
REGIONS=`aws ec2 describe-regions --query "Regions[].RegionName" --output text`
```

2. アカウントで有効になっているリージョンのデフォルトのセキュリティグループのインバウント/アウトバウンドルールを削除
```bash
for REGION in ${REGIONS[@]}; do
  echo $REGION
  GROUP_IDS=$(aws ec2 describe-security-groups --query 'SecurityGroups[?GroupName==`default`].GroupId' --region $REGION --output text)
  for GROUP_ID in ${GROUP_IDS[@]}; do
    echo $GROUP_ID
    aws ec2 revoke-security-group-ingress --group-id $GROUP_ID --ip-permissions "`aws ec2 describe-security-groups --region $REGION --output json --group-ids $GROUP_ID --query "SecurityGroups[0].IpPermissions"`" --region $REGION --output text
    aws ec2 revoke-security-group-egress --group-id $GROUP_ID --ip-permissions "`aws ec2 describe-security-groups --region $REGION --output json --group-ids $GROUP_ID --query "SecurityGroups[0].IpPermissionsEgress"`" --region $REGION --output text
  done
done
```

- ルール削除が実行された場合
```
ap-northeast-1
sg-xxxxxxxxxxxxxxxxxxx
True
True
```

- 既にルール削除済みの場合
```
ap-northeast-1
sg-xxxxxxxxxxxxxxxxxxx

An error occurred (MissingParameter) when calling the RevokeSecurityGroupIngress operation: Either 'ipPermissions' or 'securityGroupRuleIds' should be provided.

An error occurred (MissingParameter) when calling the RevokeSecurityGroupEgress operation: Either 'ipPermissions' or 'securityGroupRuleIds' should be provided.
```

3. 確認
```bash
for REGION in ${REGIONS[@]}; do
  echo $REGION
  GROUP_IDS=$(aws ec2 describe-security-groups --query 'SecurityGroups[?GroupName==`default`].GroupId' --region $REGION --output text)
  for GROUP_ID in ${GROUP_IDS[@]}; do
    echo $GROUP_ID
    aws ec2 describe-security-groups --region $REGION --output json --group-ids $GROUP_ID --query "SecurityGroups[0].IpPermissions" --region $REGION
    aws ec2 describe-security-groups --region $REGION --output json --group-ids $GROUP_ID --query "SecurityGroups[0].IpPermissionsEgress" --region $REGION
  done
done
```

- ルール削除後
```
ap-northeast-1
sg-xxxxxxxxxxxxxxxxxxx
[]
[]
```
