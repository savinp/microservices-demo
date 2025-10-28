Option A — If You Have Access to STS Directly (with a base user or assumed role)

If your EC2 can use an existing set of credentials (like your login assumed role), you can have a cron job that:

Runs aws sts assume-role

Writes the new credentials into the AWS credentials file

Automatically refreshes before expiry

Example: /usr/local/bin/refresh-creds.sh
#!/bin/bash

ROLE_ARN="arn:aws:iam::<ACCOUNT_ID>:role/S3AccessRole"
PROFILE_NAME="s3-session"
SESSION_NAME="ec2-s3-refresh"
CREDENTIAL_FILE="/home/ec2-user/.aws/credentials"

# Assume the role and fetch credentials
aws sts assume-role \
  --role-arn "$ROLE_ARN" \
  --role-session-name "$SESSION_NAME" \
  --duration-seconds 3600 \
  --output json > /tmp/creds.json

# Parse and write to credentials file
AWS_ACCESS_KEY_ID=$(jq -r '.Credentials.AccessKeyId' /tmp/creds.json)
AWS_SECRET_ACCESS_KEY=$(jq -r '.Credentials.SecretAccessKey' /tmp/creds.json)
AWS_SESSION_TOKEN=$(jq -r '.Credentials.SessionToken' /tmp/creds.json)

aws configure set aws_access_key_id "$AWS_ACCESS_KEY_ID" --profile $PROFILE_NAME
aws configure set aws_secret_access_key "$AWS_SECRET_ACCESS_KEY" --profile $PROFILE_NAME
aws configure set aws_session_token "$AWS_SESSION_TOKEN" --profile $PROFILE_NAME

echo "$(date) : Refreshed AWS credentials for profile $PROFILE_NAME" >> /var/log/aws-token-refresh.log

Add Cron Job

Run every 45 minutes:

crontab -e


Add this line:

*/45 * * * * /usr/local/bin/refresh-creds.sh

Use the Profile in Your App or Scripts

When running commands or SDKs from EC2, just use:

aws s3 ls --profile s3-session


Or set environment variable globally:

export AWS_PROFILE=s3-session
