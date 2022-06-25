## List all buckets ##

aws s3 ls

## List specific bucket ##

aws s3 ls <bucket-name>

## exclude all files matching pattern ##

aws s3 cp <s3-object-key> <path-to-copy-to>

eg: aws s3 cp s3://stock-temple-media/0a9a8e87-8461-485f-8d19-279d3124f215THUMBNAIL.png ./

## delete object or key (aka file or folder) on S3 bucket ##

aws s3 rm <s3-object-key>

eg: aws s3 rm s3://my-bucket/path/MySubdirectory/MyFile3.txt

## sync data on s3 to local path ##

aws s3 sync <source> <target>

eg: aws s3 sync s3://my-bucket .

## sync data on s3 to local path (delete files in target not in source) ##

aws s3 sync <source> <target> --delete

eg: aws s3 sync s3://my-bucket .

