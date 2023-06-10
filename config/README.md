# AWS CLI コマンド例

## 前提
- AWS CloudShellまたはMacでの利用を想定

### AWS Configのデータ保存期間の設定を指定のリージョンに対して一括で行う

```
# 作業対象のリージョンを設定
TARGET_REGIONS=("ap-northeast-1" "us-east-1")

# データ保存期間の日数を指定(デフォルトは7年)
DAY=365

# 設定処理
for TARGET_REGION in ${TARGET_REGIONS[@]}; do
  echo $TARGET_REGION
  aws configservice put-retention-configuration --retention-period-in-days $DAY --region $TARGET_REGION --output text
done
```

### 有効化した AWS Config を無効化する

```
aws configservice delete-configuration-recorder --configuration-recorder-name default
aws configservice delete-delivery-channel --delivery-channel-name default
```

### 有効化した AWS Config を無効化する(指定リージョンで一括設定)

```
# 作業対象のリージョンを設定
TARGET_REGIONS=("ap-northeast-1" "us-east-1")

# 設定処理
for TARGET_REGION in ${TARGET_REGIONS[@]}; do
  echo $TARGET_REGION
  aws configservice delete-configuration-recorder --configuration-recorder-name default --region $TARGET_REGION
  aws configservice delete-delivery-channel --delivery-channel-name default --region $TARGET_REGION
done
```
