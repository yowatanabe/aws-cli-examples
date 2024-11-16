# Remove all inbound and outbound rules from the default security groups

AWS SecurityHub [EC2.2](https://docs.aws.amazon.com/securityhub/latest/userguide/ec2-controls.html#ec2-2) checks if the default security group allows inbound or outbound traffic. If the security group allows inbound or outbound traffic, the control fails.

The default security group is usually not used in the AWS account used for work. I decided to remove the inbound/outbound rules of the default security group as EC2.2 remediation on SecurityHub. Still, removing them from the management console is not easy, so I prepared a bash script to delete them easily.

This script is intended for use with AWS CloudShell or Mac.


## 1. Get the regions enabled in your account

```bash
REGIONS=`aws ec2 describe-regions --query "Regions[].RegionName" --output text`
```

## 2. Remove inbound/outbound rules for the default security group for the region enabled on your account

```bash
for REGION in ${REGIONS[@]}; do
  echo "# ------------------------------------ #"
  echo "REGION:" $REGION
  GROUP_IDS=$(aws ec2 describe-security-groups --query 'SecurityGroups[?GroupName==`default`].GroupId' --region $REGION --output text)
  for GROUP_ID in ${GROUP_IDS[@]}; do
    echo "SecurityGroup:" $GROUP_ID
    # Delete inbound rules
    aws ec2 revoke-security-group-ingress --group-id $GROUP_ID --ip-permissions "`aws ec2 describe-security-groups --region $REGION --output json --group-ids $GROUP_ID --query "SecurityGroups[0].IpPermissions"`" --region $REGION --output json > /dev/null 2>&1 && echo "Deleted inbound rules" || echo "No inbound rules exist"
    # Delete outbound rules
    aws ec2 revoke-security-group-egress --group-id $GROUP_ID --ip-permissions "`aws ec2 describe-security-groups --region $REGION --output json --group-ids $GROUP_ID --query "SecurityGroups[0].IpPermissionsEgress"`" --region $REGION --output json > /dev/null 2>&1 && echo "Deleted outbound rules" || echo "No outbound rules exist"
  done
done
```

### When the rule deletion is executed

```bash
# ------------------------------------ #
... (other regions omitted)
# ------------------------------------ #
REGION: ap-northeast-1
SecurityGroup: sg-xxxxxxxxxxxxxxxxx
Deleted inbound rules
Deleted outbound rules
SecurityGroup: sg-xxxxxxxxxxxxxxxxx # If there is more than one default security group
Deleted inbound rules
Deleted outbound rules
# ------------------------------------ #
... (other regions omitted)
```

### If the rule has already been deleted

```bash
# ------------------------------------ #
... (other regions omitted)
# ------------------------------------ #
REGION: ap-northeast-1
SecurityGroup: sg-xxxxxxxxxxxxxxxxx
No inbound rules exist
No outbound rules exist
# ------------------------------------ #
... (other regions omitted)
```


## 3. Check inbound/outbound rules for the default security group for the region enabled on your account

```bash
for REGION in ${REGIONS[@]}; do
  echo "# ------------------------------------ #"
  echo "REGION:" $REGION
  GROUP_IDS=$(aws ec2 describe-security-groups --query 'SecurityGroups[?GroupName==`default`].GroupId' --region $REGION --output text)
  for GROUP_ID in ${GROUP_IDS[@]}; do
    echo "SecurityGroup:" $GROUP_ID
    # Check inbound rules
    inbound_rule=$(aws ec2 describe-security-groups --region $REGION --output json --group-ids $GROUP_ID --query "SecurityGroups[0].IpPermissions" --region $REGION); echo "$inbound_rule" | jq -e 'length > 0' > /dev/null && echo "$inbound_rule" || echo "No inbound rules exist"
    # Check outbound rules
    outbound_rule=$(aws ec2 describe-security-groups --region $REGION --output json --group-ids $GROUP_ID --query "SecurityGroups[0].IpPermissionsEgress" --region $REGION); echo "$outbound_rule" | jq -e 'length > 0' > /dev/null && echo "$outbound_rule" || echo "No outbound rules exist"
  done
done
```

### Result of inbound/outbound rules in the default security group being deleted

```bash
# ------------------------------------ #
... (other regions omitted)
# ------------------------------------ #
REGION: ap-northeast-1
SecurityGroup: sg-xxxxxxxxxxxxxxxxx
No inbound rules exist
No outbound rules exist
SecurityGroup: sg-xxxxxxxxxxxxxxxxx # If there is more than one default security group
No inbound rules exist
No outbound rules exist
# ------------------------------------ #
... (other regions omitted)
```

### If the default security group inbound/outbound rules remain

```bash
# ------------------------------------ #
... (other regions omitted)
# ------------------------------------ #
REGION: ap-northeast-1
SecurityGroup: sg-xxxxxxxxxxxxxxxxx
[
    {
        "IpProtocol": "-1",
        "UserIdGroupPairs": [
            {
                "UserId": "xxxxxxxxxxxx",
                "GroupId": "sg-xxxxxxxxxxxxxxxxx"
            }
        ],
        "IpRanges": [],
        "Ipv6Ranges": [],
        "PrefixListIds": []
    }
]
[
    {
        "IpProtocol": "-1",
        "UserIdGroupPairs": [],
        "IpRanges": [
            {
                "CidrIp": "0.0.0.0/0"
            }
        ],
        "Ipv6Ranges": [],
        "PrefixListIds": []
    }
]
# ------------------------------------ #
... (other regions omitted)
```
