# How to create partitions in Athena

Athena is service offered by AWS which is used to query csv/tsv/json files directly by coverting it into tables. When you have huge data to scan such as daily logs/metrics it is good to spilt the data into partitions and load it into athena. When you load data as partitions you have benefits to query only a selected partitions which will help you to save huge cost (since athena is billed for total data scanned) and obvisouly it will increase the query performance as well as we are query only less data in partitions compared to whole bunch of data.

This is the sample table which I created for reading cloudtrail logs from S3 to Athena. The path of S3 folder is 
bucket_name/AWSLogs/accountid/CloudTrail/region/year/month/date/filename.gz files. CloudTrail will upload .gz files in the mentioned folder structure. So when you need to find the who is the user that has launched instances in us-east-1 region within last 1 week, it is unnecessary to scan the whole data. If you have partitions on region, year, month, date you can just scan only the related data and produce the same result in a cost effective and performance effective way.

Below sample query will create the Athena table with region, year, month and date partitions.

```
CREATE EXTERNAL TABLE `cloudtrail_logs`(
  `eventversion` string COMMENT 'from deserializer', 
  `useridentity` struct<type:string,principalid:string,arn:string,accountid:string,invokedby:string,accesskeyid:string,username:string,sessioncontext:struct<attributes:struct<mfaauthenticated:string,creationdate:string>,sessionissuer:struct<type:string,principalid:string,arn:string,accountid:string,username:string>>> COMMENT 'from deserializer', 
  `eventtime` string COMMENT 'from deserializer', 
  `eventsource` string COMMENT 'from deserializer', 
  `eventname` string COMMENT 'from deserializer', 
  `awsregion` string COMMENT 'from deserializer', 
  `sourceipaddress` string COMMENT 'from deserializer', 
  `useragent` string COMMENT 'from deserializer', 
  `errorcode` string COMMENT 'from deserializer', 
  `errormessage` string COMMENT 'from deserializer', 
  `requestparameters` string COMMENT 'from deserializer', 
  `responseelements` string COMMENT 'from deserializer', 
  `additionaleventdata` string COMMENT 'from deserializer', 
  `requestid` string COMMENT 'from deserializer', 
  `eventid` string COMMENT 'from deserializer', 
  `resources` array<struct<arn:string,accountid:string,type:string>> COMMENT 'from deserializer', 
  `eventtype` string COMMENT 'from deserializer', 
  `apiversion` string COMMENT 'from deserializer', 
  `readonly` string COMMENT 'from deserializer', 
  `recipientaccountid` string COMMENT 'from deserializer', 
  `serviceeventdetails` string COMMENT 'from deserializer', 
  `sharedeventid` string COMMENT 'from deserializer', 
  `vpcendpointid` string COMMENT 'from deserializer')
PARTITIONED BY ( 
  `region` string, 
  `year` string, 
  `month` string, 
  `date` string)
ROW FORMAT SERDE 
  'com.amazon.emr.hive.serde.CloudTrailSerde' 
STORED AS INPUTFORMAT 
  'com.amazon.emr.cloudtrail.CloudTrailInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  's3://your_cloudtrail_bucket_name/AWSLogs/Account_ID'
```

Running the above query will create you the partitioned table named cloudtrail_logs. By default there won't be any data in the table, you have to load each partitions individually so that it will load the S3 files to athena table.

Below is the command to load the partitions to the table.

```
ALTER TABLE sampledb.cloudtrail_logs ADD PARTITION (region = 'us-east-1', year = '2020', month = '01', date = '08') LOCATION 's3://your_cloudtrail_bucket_name/AWSLogs/Account_ID/CloudTrail/us-east-1/2020/01/08/';
```

It is really difficult to load the data manually for all the regions daily. I already have a lambda function created in my 'python_boto3' repository which has python file which will load the partitions automatically whenever you trigger the function.

### How to fetch the data using partitions?

Using where condition you can fetch the data from partitions. 
Once your table is created with partitions you will see all the partitions as an extra column in your tables. When you look at the below table, we never provided any column named region/year/month/date in the above create table statement but we got these 4 new columns. It is created based on our partitions.

![](table.jpg)

### Let's see the actual benefits by running different queries

Running query with just region partition, below query just uses one where condition. i.e. region

![](full_query.jpg)

Data scanned is 814.29 KB.

S3 size is 814.3 KB of folder s3://your_cloudtrail_bucket_name/AWSLogs/Account_ID/CloudTrail/us-east-1/ is also 814.3 KB.

![](s3_size.jpg)

Whereas running query with all partitions region, year, month and date. Data scanned in athena is 409.83 KB.

![](selected_query.jpg)

And the S3 size of the folder s3://your_cloudtrail_bucket_name/AWSLogs/Account_ID/CloudTrail/us-east-1/2020/01/09 is also 409.9 KB.

![](s3_selected_size.jpg)

Which concludes that using partitions we scan lesser data and get the same result in an efficient way.

This is just one way of loading partitions into the table. The above method of loading partition is used when you have th folder structure as 2019/01/03 i.e. without year=2019/month=01/date=03. There is another way of loading partition as well and that is used when your folder structure is like region=us-east-1/year=2020/month=01/date=05 etc.

You can just run the below command and it will load the all the partitions to the table at once.

```
MSCK REPAIR TABLE table_name;
```
