# check-db-cluster-parameter-group-for-specific-name

## 前提
- AWS CloudShellまたはMacでの利用を想定

## 特定の名前のDBクラスターパラメータグループの特定の設定項目の値を確認する

例. 以下の名前のDBクラスターパラメータグループのタイムゾーン`timezone`と論理レプリケーション`rds.logical_replication`の値を確認する
- demo-cluster-parameter-group-1
- demo-cluster-parameter-group-2
- demo-cluster-parameter-group-3

#### 名前に「demo」が含まれるDBクラスターパラメータグループを取得
```
DB_CLUSTER_PARAMETER_GROUPS=$(aws rds describe-db-cluster-parameter-groups --query "DBClusterParameterGroups[?contains(DBClusterParameterGroupName,\`demo\`)].DBClusterParameterGroupName" --output text)
```

#### for文でaws rds describe-db-cluster-parametersを実行
```
for DB_CLUSTER_PARAMETER_GROUP in $DB_CLUSTER_PARAMETER_GROUPS; do
    echo "DBClusterParameterGroupName is $DB_CLUSTER_PARAMETER_GROUP"
    aws rds describe-db-cluster-parameters --db-cluster-parameter-group-name $DB_CLUSTER_PARAMETER_GROUP --query 'Parameters[?ParameterName==`rds.logical_replication` || ParameterName==`timezone`].{ParameterName:ParameterName,ParameterValue:ParameterValue}' --output table
    echo ""
done
```

#### 実行例
```
$ for DB_CLUSTER_PARAMETER_GROUP in $DB_CLUSTER_PARAMETER_GROUPS; do
>     echo "DBClusterParameterGroupName is $DB_CLUSTER_PARAMETER_GROUP"
>     aws rds describe-db-cluster-parameters --db-cluster-parameter-group-name $DB_CLUSTER_PARAMETER_GROUP --query 'Parameters[?ParameterName==`rds.logical_replication` || ParameterName==`timezone`].{ParameterName:ParameterName,ParameterValue:ParameterValue}' --output table
>     echo ""
> done
DBClusterParameterGroupName is demo-cluster-parameter-group-1
-----------------------------------------------
|         DescribeDBClusterParameters         |
+--------------------------+------------------+
|       ParameterName      | ParameterValue   |
+--------------------------+------------------+
|  rds.logical_replication |  0               |
|  timezone                |  UTC             |
+--------------------------+------------------+

DBClusterParameterGroupName is demo-cluster-parameter-group-2
-----------------------------------------------
|         DescribeDBClusterParameters         |
+--------------------------+------------------+
|       ParameterName      | ParameterValue   |
+--------------------------+------------------+
|  rds.logical_replication |  0               |
|  timezone                |  UTC             |
+--------------------------+------------------+

DBClusterParameterGroupName is demo-cluster-parameter-group-3
-----------------------------------------------
|         DescribeDBClusterParameters         |
+--------------------------+------------------+
|       ParameterName      | ParameterValue   |
+--------------------------+------------------+
|  rds.logical_replication |  0               |
|  timezone                |  UTC             |
+--------------------------+------------------+
```
