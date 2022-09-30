# Federated Query (RDS Postgres) - Same VPC and Different Subnet

## Pre-requisites
- Create Redshift Cluster in the same VPC and different subnet
- Create RDS instance also in the same VPC and different subnet
- S3 bucket to hold the source file

### Policy/Role
#### Glue Crawler Policy/Role
Use the below AWS managed policy 
```
AWSGlueServiceRole
```

#### Create secret
Follow the below link and create a secret for RDS instance

#### Redshift
Make sure that Redshift has access to Glue and S3, and attach below policy to Redshift

https://docs.aws.amazon.com/secretsmanager/latest/userguide/create_database_secret.html

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AccessSecret",
            "Effect": "Allow",
            "Action": [
                "secretsmanager:GetResourcePolicy",
                "secretsmanager:GetSecretValue",
                "secretsmanager:DescribeSecret",
                "secretsmanager:ListSecretVersionIds",
                "secretsmanager:GetRandomPassword",
                "secretsmanager:ListSecrets"
            ],
            "Resource": [
                "arn:aws:secretsmanager:us-east-2:[ACCOUNT_ID]:secret:RedshiftPostgreSQLSecret",
                "arn:aws:secretsmanager:us-east-2:[ACCOUNT_ID]:secret:same-rds-subnet"
            ]
        }
    ]
}
```

### Step - 1: Upload file S3

### Step - 2: Crawl file using Glue crawler
Follow the below link to crawl the s3 file
https://docs.aws.amazon.com/glue/latest/ug/tutorial-add-crawler.html

### Step - 3: Create external schema in Redshift
```
DROP SCHEMA aurora_pg_ext_1;

CREATE EXTERNAL SCHEMA aurora_pg_ext_1
FROM POSTGRES
DATABASE 'postgres'
SCHEMA 'private'
URI '[federated_database_name]'
IAM_ROLE '[IAM_ROLE_NAME]'
SECRET_ARN '[USE THE SECRET CREATED ABOVE]';

select * from aurora_pg_ext_1.customer;
```

Note - If the RDS and Redshift are in public subnet then disable enhanced VPC routing.
