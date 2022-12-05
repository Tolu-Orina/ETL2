# ETL2: INGESTING DATA FROM A MYSQL RDS DATABASE INTO S3-BASED DATA LAKE USING AWS DMS (Database Migration Service)

### AWS Data Database Migration Service (AWS DMS)
AWS Database Migration Service (AWS DMS) is a managed migration and replication service that helps move your database and analytics workloads to AWS quickly, securely, and with minimal downtime and zero data loss. It is is primarily intended for doing either a one-off ingestion of historical data 
from a database, or for replicating data from a relational database on an ongoing basis. 
When using AWS DMS, the target is either a different database engine or an Amazon S3-based data lake.

DMS provisions one or more EC2 servers as replication instances. These replication instances connect to the source database, read data from the source, prepare the data for loading into the target, and then connect to the target and write the data.

#### PROCEDURE:
- Create a MySQl database instance using Amazon Relational Database Service (RDS): etl2db in order to host our test database
- Load Demo Data (Sakila) using an ec2 instance into our RDS MySQL DB
- Create IAM policy and role for DMS: The IAM policy and role will allow DMS
to write to our target s3 bucket
- Create an S3 VPC gateway endpoint
- Create a DMS replication instance and other required DMS resources

**User Data script**
```
#!/bin/bash
yum install -y mariadb
curl https://downloads.mysql.com/docs/sakila-db.zip -o sakila.zip
unzip sakila.zip
cd sakila-db
mysql --host=<myrdsendpoint> --user=admin --password=<mypass> -f < sakila-schema.sql
mysql --host=<myrdsendpoint> --user=admin --password=<mypass> -f < sakila-data.sql
```
The Bash Script:
- Installs MariaDB (which comes with a mysql client to connect to our MySQL server)
- Downloads a demo database named sakila
- Connects to the mysql server twice to inject schema and data

**IAM Policy**: ETLS3BucketPolicy (permissive can be more fine-grained)
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:*"
                ],
            "Resource": [
                "arn:aws:s3:::etl-trex-landing-zone/*"
                ]
        }
        ]
}
```

**CONFIGURING THE Database Migration Task**
- DMS Replication Instance connects to the source endpoint,
 retrieves data, and writes to the target endpoint
- Configure Source and Target Endpoints: Source(etl2-db), Target(etl2-etl-trex-csv)
- Create Database Migration task (using the pre-provisioned configuration settings): etl2-mysql-s3

**GET MIGRATED FILE**
```
aws s3api get-object --bucket etl-trex-landing-zone --key etl2-db/sakila/actor/LOAD00000001.csv received.csv
```


https://user-images.githubusercontent.com/78595009/205676950-e1887624-10e3-4f00-88dc-7218de152d29.mp4

