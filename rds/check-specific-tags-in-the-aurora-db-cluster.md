# Check specific tags in the Aurora DB cluster

## 前提
- AWS CloudShellまたはMacでの利用を想定

## Aurora DBクラスターの特定のタグを確認する

例. DBクラスターのタグ`AutoStop`と`AutoStart`の値を確認する (DBクラスターは`demo-db1-aurora`と`demo-db2-aurora`)

#### コマンド
```
# aws rds describe-db-clusters (タグを複数指定)
aws rds describe-db-clusters --query 'DBClusters[].[DBClusterIdentifier,TagList[?Key==`AutoStop` || Key==`AutoStart`]]' --output text


# aws rds describe-db-clusters (部分一致)
aws rds describe-db-clusters --query "DBClusters[].[DBClusterIdentifier,TagList[?contains(Key,\`Auto\`)]]" --output text


# aws rds list-tags-for-resource (「--resource-name」で対象のDBクラスターのARNを指定する必要がある)
aws rds list-tags-for-resource --resource-name arn:aws:rds:{リージョン}:{AWSアカウントID}:cluster:{DBクラスター名} \
--query "TagList[?contains(Key,\`Auto\`)]" \
--output text
```

#### 実行例
```
$ aws rds describe-db-clusters --query 'DBClusters[].[DBClusterIdentifier,TagList[?Key==`AutoStop` || Key==`AutoStart`]]' --output text
demo-db1-aurora
AutoStop        yes
AutoStart       yes
demo-db2-aurora
AutoStop        yes
AutoStart       yes


$ aws rds describe-db-clusters --query "DBClusters[].[DBClusterIdentifier,TagList[?contains(Key,\`Auto\`)]]" --output text
demo-db1-aurora
AutoStop        yes
AutoStart       yes
demo-db2-aurora
AutoStop        yes
AutoStart       yes


$ aws rds list-tags-for-resource --resource-name arn:aws:rds:{リージョン}:{AWSアカウントID}:cluster:{DBクラスター名} \
> --query "TagList[?contains(Key,\`Auto\`)]" \
> --output text
AutoStop        yes
AutoStart       yes
```

## (参考)タグの値を変更する

#### コマンド
```
# 1つのタグを指定
aws rds add-tags-to-resource \
--resource-name arn:aws:rds:{リージョン}:{AWSアカウントID}:cluster:{DBクラスター名} \
--tags "[{\"Key\": \"AutoStop\",\"Value\": \"no\"}]"


# 複数のタグを指定
aws rds add-tags-to-resource \
--resource-name arn:aws:rds:{リージョン}:{AWSアカウントID}:cluster:{DBクラスター名} \
--tags "[{\"Key\": \"AutoStop\",\"Value\": \"no\"},{\"Key\": \"AutoStart\",\"Value\": \"no\"}]"
```

#### 複数のDBクラスターのタグの値を一括で変更

1. 作業対象のDBクラスターのARNを指定
```
TARGET_DB_CLUSTERS_ARN=(
"arn:aws:rds:{リージョン}:{AWSアカウントID}:cluster:{DBクラスター名}"
"arn:aws:rds:{リージョン}:{AWSアカウントID}:cluster:{DBクラスター名}"
)
```

2. 変更後のタグの値を設定(no|yes)
```
TAG_VALUE=no
```

3. 変更するタグを指定

- 1つのタグを変更する場合
```
TARGET_TAGS="[{\"Key\": \"AutoStop\",\"Value\": \"$TAG_VALUE\"}]"
```

- 複数のタグを変更する場合
```
TARGET_TAGS="[{\"Key\": \"AutoStop\",\"Value\": \"$TAG_VALUE\"},{\"Key\": \"AutoStart\",\"Value\": \"$TAG_VALUE\"}]"
```

1. 設定
```
for TARGET_DB_CLUSTER_ARN in ${TARGET_DB_CLUSTERS_ARN[@]}; do
  aws rds add-tags-to-resource --resource-name $TARGET_DB_CLUSTER_ARN --tags "$TARGET_TAGS"
  aws rds describe-db-clusters --db-cluster-identifier $TARGET_DB_CLUSTER_ARN --query "DBClusters[].[DBClusterIdentifier,TagList[?contains(Key,\`Auto\`)]]" --output text
done
```
