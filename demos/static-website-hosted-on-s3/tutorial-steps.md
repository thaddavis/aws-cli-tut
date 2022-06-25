# 1
# Show how to ask cli for help ##
aws <api-name> help
eg: aws s3api help

# 2
## Create a s3 bucket ##
aws s3 mb <bucket-name>
eg: aws s3 mb s3://humblagga.com

# 3
## copy website to bucket ##
aws s3 sync ./ s3://humblagga.com

# 4
## Set bucket policy ##
aws s3api put-bucket-policy --bucket <bucket-name> --policy <file-with-policy>
eg: aws s3api put-bucket-policy --bucket humblagga.com --policy file://json/bucket-policy.json

# 5
## Enable Static Website Hosting ##
aws s3 website <bucket-name> --index-document index.html --error-document error.html
eg: aws s3 website s3://humblagga.com --index-document index.html

# 6
## Test Website URL
website will be: http://<bucket-name>.s3-website.<aws-region>.amazonaws.com
eg: http://humblagga.com.s3-website.us-east-1.amazonaws.com

# 7
## Buy Domain ##
aws route53domains register-domain --region <aws-region> --cli-input-json <path-to-registration-info-json>
eg: aws route53domains register-domain --region us-east-1 --cli-input-json file://json/register-domain.json --profile s3_user

*** Tip: Will need to add the following policies:
*** route53domains:RegisterDomain on resource: *
*** route53:CreateHostedZone on resource: *

# 8
## Get Hosted Zone Id of newly created domain ## 
aws route53 list-hosted-zones | jq '.HostedZones[] | select(.Name == "humblagga.com.") | .Id'

eg: aws route53 list-hosted-zones | jq '.HostedZones[] | select(.Name == "humblagga.com.") | .Id' | cut -d'"' -f 2 | cut -d'/' -f 3

# 9
## Configure Route53 ##

    9a)
        cd json
        sed -i '' 's/RECORD_NAME/.humblagga.com/g' record-set.json

    9b)
        HOSTED_ZONE_ID=$(aws route53 list-hosted-zones | jq '.HostedZones[] | select(.Name == "humblagga.com.") | .Id' | cut -d'"' -f 2 | cut -d'/' -f 3)

        sed -i '' "s/HOSTED_ZONE_ID/$HOSTED_ZONE_ID/g" record-set.json

    9c) 
        S3_BUCKET_URL=humblagga.s3-website.us-east-1.amazonaws.com

        sed -i '' "s/S3_BUCKET_URL/$S3_BUCKET_URL/g" record-set.json

TIP: https://docs.aws.amazon.com/general/latest/gr/s3.html#s3_website_region_endpoints

aws route53 change-resource-record-sets --hosted-zone-id $HOSTED_ZONE_ID --change-batch file://json/record-set.json

TIP: aws route53 change-resource-record-sets --hosted-zone-id $HOSTED_ZONE_ID --change-batch file://update-record-set.json --profile s3_user --region us-east-1

TIP: aws route53 list-resource-record-sets --hosted-zone-id $HOSTED_ZONE_ID

# 10
## create certificate with ACM ##

TIP: aws acm list-certificates --profile s3_user --region us-east-1

IDEMPOTENCY_TOKEN=$(uuidgen | tr -d '\n-' | tr '[:upper:]' '[:lower:]' | cut -c1-6)
aws acm request-certificate --domain-name humblagga.com --validation-method DNS --idempotency-token IDEMPOTENCY_TOKEN --subject-alternative-names www.humblagga.com --profile s3_user --region us-east-1

# 11
## create prerequisite

OAI_CALLER_REFERENCE_TOKEN=$(uuidgen | tr -d '\n-' | tr '[:upper:]' '[:lower:]' | cut -c1-8)
# -> sed needed
aws cloudfront create-cloud-front-origin-access-identity --cloud-front-origin-access-identity-config file://json/OAI-config.json --profile s3_user --region us-east-1


# 12
## create cloudfront distribution

a) CALLER_REFERENCE_TOKEN=$(uuidgen | tr -d '\n-' | tr '[:upper:]' '[:lower:]' | cut -c1-8)

aws cloudfront create-distribution --distribution-config file://dist-config.json

TIP: https://docs.aws.amazon.com/cloudfront/latest/APIReference/API_DistributionConfig.html

# 13
## add Route53 A record for Cloudfront

aws route53 change-resource-record-sets --hosted-zone-id $HOSTED_ZONE_ID --change-batch file://change-resource-record-sets.json --profile s3_user --region us-east-1

# 14
## test the final URL ##

CONGRATS!!!