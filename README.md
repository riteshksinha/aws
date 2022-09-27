# Table of Contents
[Redshift serverless snapshot automation](#Redshift-serverless-snapshot-automation)
- [Policy to be created](#Policy-to-be-created)
- [Role to be created](#Role-to-be-created)
- [Lambda function](#Lambda-function)
	- [Attach role](#Attach-role)
	- [Add BOTO3 Layer](#Add-BOTO3-Layer)
	- [Environment variables](#Environment-variables)
- [Event Bridge](#Event-Bridge)
- [Cloud Formation](#Cloud-Formation)

## Redshift serverless snapshot automation
This process can be used to automate the Redshift serverless snapshot automation. 

Restoring a snapshot to a serverless namespace replaces the database with the database in the snapshot. You can also restore recovery points, which are created every 30 minutes and kept for 24 hours, to your serverless namespace.

This process helps in automating the process and giving flexibility of storing snapshot more than 24 hours.

## Policy to be created 
Name - redshift-serverless-snapshot-automation-policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "redshift:*",
                "redshift-serverless:*"
            ],
            "Effect": "Allow",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "arn:aws:iam::*:role/aws-service-role/redshift.amazonaws.com/AWSServiceRoleForRedshift",
            "Condition": {
                "StringLike": {
                    "iam:AWSServiceName": "redshift.amazonaws.com"
                }
            }
        },
        {
            "Sid": "DataAPIPermissions",
            "Action": [
                "redshift-data:ExecuteStatement",
                "redshift-data:CancelStatement",
                "redshift-data:ListStatements",
                "redshift-data:GetStatementResult",
                "redshift-data:DescribeStatement",
                "redshift-data:ListDatabases",
                "redshift-data:ListSchemas",
                "redshift-data:ListTables",
                "redshift-data:DescribeTable"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ]
}
```

## Role to be created
Name - redshift-serverless-snapshot-automation-role 
```
Attached the above created policy to this role
```

## Lambda function
```
import json
import boto3
import time
import os
import logging
logger = logging.getLogger(__name__)

def lambda_handler(event, context):
    
    serverless_client = boto3.client('redshift-serverless')
    namespace = os.getenv("namespace")
    retention_period = int(os.getenv("retention_period"))

    namespace_list = [namespace_list_temp.strip() for namespace_list_temp in namespace.split(",") if namespace_list_temp]

    for i_namespace in namespace_list:
        snapshot_timestamp = time.strftime("%Y-%m-%d-%H-%M-%S", time.localtime(time.time()))
        snapshot_name = i_namespace + '-' + 'snapshot' + '-' + snapshot_timestamp
        try:   
            response = serverless_client.create_snapshot(namespaceName=i_namespace,retentionPeriod=retention_period,snapshotName=snapshot_name)
            status = 'Snapshot creation initiated for ' + i_namespace
            
        except Exception as e:
            msg = e.response['Error']['Code']
            if msg == 'ConflictException':
                status = 'Snapshot creation is already in progress for ' + i_namespace
                logger.error(status) 
                continue
            elif msg == 'ResourceNotFoundException':
                status = i_namespace + ' namespace does not exist'
                logger.error(status) 
                continue
            else:
                raise
    return status
```

## Attach role
```
Attach above created role to the lambda function created redshift-serverless-snapshot-automation-role
```

## Add BOTO3 Layer
```
Attach latest boto3 package as layer
```

## Environment variables
![image](https://user-images.githubusercontent.com/28712961/192637887-78e7a50a-0b49-421f-95cd-4c78845e850d.png)
 
## Event Bridge
You can start an automation by specifying a runbook as the target of an Amazon EventBridge event. You can start automations according to a schedule, or when a specific AWS system event occurs.


Click on below **Launch Button** to launch the Cloud Formation:
https://docs.amazonaws.cn/en_us/eventbridge/latest/userguide/eb-run-lambda-schedule.html#eb-schedule-create-rule

[<img src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png" target=\”_blank\”>](https://github.com/riteshksinha/aws/blob/main/redshiftServerlessSnapshotAutomation.YAML
)
