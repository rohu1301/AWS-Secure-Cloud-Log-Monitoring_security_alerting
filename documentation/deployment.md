# Deployment

This is the order I built things in, and the settings I used.

## 1. Network

I created a VPC called `cloud-monitoring-vpc` with the CIDR `10.0.0.0/16`, added a public subnet, and attached an Internet Gateway. The route table has two entries:

| Destination | Target |
|---|---|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |

## 2. Security group

`cloud-monitoring-sg` allows two things:

| Port | Source | Why |
|---|---|---|
| TCP 22 (SSH) | My IP only | Admin access |
| TCP 80 (HTTP) | 0.0.0.0/0 | Public web page |

## 3. IAM role

I made a role called `EC2-CloudWatch-Log-Role` with the `CloudWatchAgentServerPolicy` managed policy, then attached it to the instance. That lets the agent send logs to CloudWatch without any stored credentials.

## 4. EC2 and Apache

I launched an Amazon Linux 2023 instance in the public subnet with the security group and role above. Then I installed Apache, enabled it, and started it. Opening the public IP in a browser showed my test page, which confirmed the server worked.

## 5. CloudWatch Agent

I installed the agent and configured it to collect Apache and SSH logs into these log groups:

```text
/aws/ec2/cloud-monitoring/apache-access
/aws/ec2/cloud-monitoring/apache-error
/aws/ec2/cloud-monitoring/ssh-security
```

## 6. Metric filter and alarm

On `ssh-security` I added a metric filter:

- **Pattern:** `"Invalid user"`
- **Namespace:** `CloudMonitoring/Security`
- **Metric:** `InvalidSSHLogin`, value 1

Then I created the alarm `CloudMonitoring-Invalid-SSH-Alert`:

- **Statistic:** Sum
- **Period:** 5 minutes
- **Threshold:** greater than or equal to 1
- **Evaluation:** 1 out of 1
- **Action:** notify the SNS topic

## 7. SNS

I created a topic and subscribed my email address. I had to confirm the subscription from the email AWS sent before alerts would arrive.

## 8. EventBridge

The rule `CloudMonitoring-EC2-StateChange-Rule` matches `EC2 Instance State-change Notification` and targets the SNS topic.

## 9. S3 archive

I used an existing bucket with encryption and Block Public Access turned on, and exported the SSH log group to it with a CloudWatch Logs export task. The bucket policy needs to allow CloudWatch Logs to write to it, or the export will fail.
