# Testing

I tested every part with real events instead of just checking the configuration. Each test says what I did and what I expected to see.

| # | What I tested | What I did | What I expected | Result |
|---|---|---|---|---|
| 1 | Web server | Opened the EC2 public IP in a browser | Apache page loads | Pass |
| 2 | SSH log collection | Logged in over SSH normally | Events appear in the `ssh-security` log group | Pass |
| 3 | Invalid login detection | Tried to SSH in with a username that doesn't exist | An `Invalid user` entry shows up in CloudWatch Logs | Pass |
| 4 | Alarm | Waited after test 3 | `CloudMonitoring-Invalid-SSH-Alert` moves to ALARM | Pass |
| 5 | Email alert | Checked my inbox after the alarm fired | SNS email arrives | Pass |
| 6 | EC2 stop event | Stopped the instance | EventBridge triggers and an email arrives | Pass |
| 7 | EC2 start event | Started the instance again | EventBridge triggers and an email arrives | Pass |
| 8 | S3 export | Ran an export task on the SSH log group | Log files appear in the S3 bucket | Pass |

## Notes

- Test 3 only works once the agent is running and the log group exists, so I tested log collection (test 2) first.
- Alarm emails arrive after the evaluation period, not instantly, since the alarm uses a 5-minute window.
- The S3 export is a one-time copy of the logs, not a live feed.

Screenshots for each test are in the `screenshots/` folder.
