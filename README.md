# Prerequisites

1) Installing aws-cli
eg: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

# 10
## add CNAME records to Route53 ##
<!-- aws route53 change-resource-record-sets --hosted-zone-id $HOSTED_ZONE_ID --change-batch file://change-resource-record-sets.json --profile s3_user --region us-east-1 -->