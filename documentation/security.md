# Security

## What I put in place

**Network.** The project sits in its own VPC instead of the default one. The security group opens SSH only to my IP, so the rest of the internet can't even reach port 22. Port 80 is open to everyone because that's the whole point of a web server.

**Credentials.** The instance uses an IAM role. There are no access keys or secrets on the server, so nothing to leak if someone gets onto it.

**Detection.** Invalid SSH usernames are logged, matched by a metric filter, and turned into an alarm. Instance start/stop/terminate events are caught by EventBridge. Both end in an email.

**Storage.** The S3 archive bucket has encryption enabled and Block Public Access on.

## Keeping secrets out of GitHub

Before pushing, I made sure the repo has no access keys, secret keys, passwords, `.pem` or SSH private keys, MFA recovery codes, or personal details. My `.gitignore` includes:

```text
*.pem
*.key
.env
.env.*
.aws/
.ssh/
credentials
```

I also checked screenshots for account IDs, IP addresses, and email addresses before adding them.

## Limitations

This is a learning project, and I know it's basic:

- The metric filter only catches one pattern (`Invalid user`). It won't catch brute-force attempts against valid usernames.
- The alarm uses a 5-minute window, so alerts aren't instant.
- HTTP is open to the world and not behind a WAF.
- The instance is in a public subnet. A production setup would use a private subnet and a bastion or Session Manager.

## What I'd do next

CloudTrail for API activity, GuardDuty for threat detection, WAF in front of the web server, and Lambda for automatic responses such as isolating an instance.
