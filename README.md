# AWS Secure Cloud Log Monitoring & Security Alerting

I built this project to get hands-on with how security monitoring actually works in AWS. It's a small setup: one web server, a handful of AWS services, and a clear goal. If someone tries to log in over SSH with a username that doesn't exist, or if the server is started or stopped, I want an email about it.

## What it does

- Runs an Apache web server on an EC2 instance inside its own VPC.
- Sends SSH and Apache logs to CloudWatch Logs.
- Spots invalid SSH login attempts with a metric filter and raises a CloudWatch alarm.
- Watches EC2 start/stop/terminate events with EventBridge.
- Sends all alerts to my inbox through SNS.
- Exports logs to an S3 bucket for archiving.

## How it fits together

```text
Invalid SSH login ─► CloudWatch Logs ─► Metric Filter ─► Alarm ─┐
                                                                ├─► SNS ─► Email
EC2 start / stop ─► EventBridge rule ───────────────────────────┘

CloudWatch Logs ─► Export task ─► S3 (archive)
```

## Services used

VPC, Internet Gateway, EC2, Security Groups, IAM, CloudWatch Logs / Metric Filters / Alarms, SNS, EventBridge, S3.

## How I tested it

I used real events rather than assuming it worked:

- Tried to SSH in with a made-up username, and the alarm fired and the email arrived.
- Stopped and started the instance, and EventBridge sent an email each time.
- Exported the SSH logs to S3 and confirmed the files were there.

The details are in [`documentation/testing.md`](documentation/testing.md).

## Security choices

- SSH is open only to my own IP; HTTP is open because it's a web server.
- The instance uses an IAM role, so there are no access keys on the box.
- The S3 bucket has encryption on and Block Public Access enabled.

## What I'd add next

CloudTrail, GuardDuty, WAF, and a Lambda function to respond automatically instead of just notifying me.

## More detail

- [Architecture](documentation/architecture.md)
- [Deployment](documentation/deployment.md)
- [Security](documentation/security.md)
- [Testing](documentation/testing.md)

*This is a learning project, not a replacement for a real SOC setup.*
