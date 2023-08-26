# Backup folder to another bucket

- S3バケット直下のフォルダを別のS3バケットの同名のフォルダ配下のbackupフォルダにバックアップする

## 前提

- AWS CloudShellまたはMacでの利用を想定

## コマンド

```bash
SRC_BUCKET=XXXXX  # バックアップ元のS3バケットを指定
DST_BUCKET=XXXXX  # バックアップ先のS3バケットを指定

for prefix in $(aws s3 ls s3://${SRC_BUCKET} | awk '{if($1 == "PRE"){print $2}}') ; do
  echo backup target:${prefix}
  aws s3 sync s3://${SRC_BUCKET}/${prefix} s3://${DST_BUCKET}/${prefix}backup --only-show-errors
done
```
