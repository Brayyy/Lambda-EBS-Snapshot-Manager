# Lambda EBS Snapshot Manager
---
A simple Lambda script to schedule creation and clearing of EBS snapshots.

- This function can create and manage many volumes and snapshots.
- To tag an EBS volume for backup, add a tag key "Backup" with the rate in which to snapshot.
- Optionally, add the key "Retention" with the number of days to override the default amount in the function.
- Old snapshots are automatically purged once the "DeleteAfter" timestamp has passed.
- Snapshots will try to inherit the EBS volume tagged Name, but will always at least indicate the source EBS volume ID and time of snapshot in the snapshot name and decription.

Valid values for "Backup" key: true, daily, 1/day, 2/day, 4/day, 6/day, 8/day, 12/day, 24/day, hourly

![EBS Volume tagging example](./example-tagged-volume.png)

Lambda config:

- Runtime: Python 2.7
- Handler: lambda-ebs-snapshot-manager.lambda_handler
- Role: [role as specified below]
- Memory: 128
- Timeout: 5 sec
- Add a trigger using "CloudWatch Events - Schedule", set for "rate(1 hour)"
- Environment variables
  - **BACKUP_KEY**: Optional. Override EBS Tag key for configuring backup. Default is "Backup".
  - **RETENTION_KEY**: Optional. Override EBS Tag key for configuring retention. Default is "Retention".
  - **RETENTION_DEFAULT**: Optional. Number of days to retain snapshots. Default is 7 days.
  - **TIME_ZONE**: Optional. Python formatted timezone. Default is "US/Eastern".
  - **AWS_REGION**: Optional. AWS region where backup is taking place. Default "us-east-1"

---

IAM Lambda Role:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:*"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVolumes",
                "ec2:DescribeSnapshots",
                "ec2:CreateSnapshot",
                "ec2:DeleteSnapshot",
                "ec2:CreateTags",
                "ec2:ModifySnapshotAttribute",
                "ec2:ResetSnapshotAttribute"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

This project is a fork of [powerupcloud/EBSSnapshotsLambda](https://github.com/powerupcloud/EBSSnapshotsLambda), but I believe the initial work was done by [serverlesscode.com](https://serverlesscode.com/post/lambda-schedule-ebs-snapshot-backups/).
