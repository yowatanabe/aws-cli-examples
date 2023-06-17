# AWS CLI コマンド例

## 前提
- AWS CloudShellまたはMacでの利用を想定

### SecurityHubの統合の任意のサービス以外を全て無効化する

```
# 作業対象のリージョンを設定
TARGET_REGIONS=("ap-northeast-1" "us-east-1")

# 結果を受け入れるサービスを変数として設定(以下はAWS Configのみ結果を受け入れるようにする場合)
AWS_CONFIG=arn:aws:securityhub:*:*:product-subscription/aws/config
AWS_SECURITYHUB=arn:aws:securityhub:*:*:product-subscription/aws/securityhub

# 設定(変数で設定したサービス以外を無効化する)
for TARGET_REGION in ${TARGET_REGIONS[@]}; do
  echo $TARGET_REGION
  PRODUCTS=`aws securityhub list-enabled-products-for-import --query 'ProductSubscriptions[]' --region $TARGET_REGION --output text`
  for PRODUCT in $PRODUCTS; do
    if [[ $PRODUCT != $AWS_CONFIG ]] && [[ $PRODUCT != $AWS_SECURITYHUB ]]; then
      echo $PRODUCT
      aws securityhub disable-import-findings-for-product --product-subscription-arn $PRODUCT --region $TARGET_REGION
    fi
  done
done
```
