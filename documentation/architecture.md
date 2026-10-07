# Architecture

The project has two separate monitoring paths that end up in the same place, my email inbox. One watches what happens *inside* the server (logs), and the other watches what happens *to* the server (state changes).

## Network layout

```text
Internet ─► Internet Gateway ─► VPC (10.0.0.0/16) ─► Public Subnet ─► EC2 (Apache)
```

I put the EC2 instance in a public subnet so I could reach the web page and SSH into it. The route table sends internet-bound traffic (`0.0.0.0/0`) through the Internet Gateway, and everything inside the VPC stays local.

## Path 1: log-based detection

```text
EC2 ─► CloudWatch Agent ─► CloudWatch Logs ─► Metric Filter ─► InvalidSSHLogin metric ─► Alarm ─► SNS ─► Email
```

The CloudWatch Agent on the instance ships three logs to their own log groups: Apache access, Apache error, and SSH security events.

On the SSH log group I added a metric filter that looks for the text `"Invalid user"`. Every match adds 1 to a custom metric, `InvalidSSHLogin`. An alarm watches that metric and notifies SNS as soon as it sees one.

## Path 2: event-based detection

```text
EC2 state change ─► EventBridge rule ─► SNS ─► Email
```

EC2 emits an event whenever its state changes. An EventBridge rule picks up those events and forwards them to the same SNS topic. This path doesn't depend on logs at all, so it still works if the agent is down.

## Log archiving

```text
CloudWatch Logs ─► Export task ─► S3
```

I used a CloudWatch Logs **export task** to copy the SSH logs into S3. This is a one-off export, not a live stream. If I wanted continuous delivery I'd need a subscription filter with Firehose, which I left out to keep things simple.
