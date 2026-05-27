# Week 0 — Billing and Architecture

## Required tasks

### 1. IAM User credentials
  1 Created admin group "Admin" with `AdministratorAccess` policy attached.
  2. Created IAM User within Admin group with `Enable console access` and generated access keys for AWS CLI.

### 2. AWS CLI installation
  1. Installation of AWS CLI when Ona environment launches.
  2. Set AWS CLI to use partial autoprompt to easily debug CLI commands.
  3. Bash commands come from the AWS CLI instructions: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
  4. Updated `.gitpod.yml` to include the following task:
     ```sh
     tasks:
     - name: aws-cli
       env:
         AWS_CLI_AUTO_PROMPT: on-partial
       init: |
         cd /workspace
         curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
         unzip awscliv2.zip
         sudo ./aws/install
         cd $THEIA_WORKSPACE_ROOT
     ```
  5. It's important to mention that the bash commands need to be run manually once first.

### 3. Set environment variables
  1. Added Access Key ID, Secrey Access Key and Default Region inside `~/.bashrc` with:
     ```sh
     export AWS_ACCESS_KEY_ID="access_key"
     export AWS_SECRET_ACCESS_KEY="secret_key"
     export AWS_DEFAULT_REGION="us-east-1"
     ```
2. Then did `source ~/.bashrc`
#### 3.1. Checked that the AWS CLI is working and you're the expected user
```sh
aws sts get-caller-identity
```
If everything works fine, then a json with UserId, Account and Arn is echoed.

### 4. Set AWS CloudWatch Billing alarm
  1. Enabled `Receive Billing Alerts` under `Billing Preferences` from the Billing Page.
  2. Then created an SNS topic with the following command and copied the TopicARN:
     ```sh
     aws sns create-topic --name billing-alarm
     ```
  3. Created a subscription to the SNS topic:
     ```sh
     aws sns subscribe \
       --topic-arn TopicARN \
       --protocol email \
       --notification-endpoint email@email.com
     ```
  4. Added a configuration JSON script `alarm-config.json` to the aws/json folder and ran the command:
     ```sh
     aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm-config.json
     ```
     It's done like this because the --metric parameter requires multiple expressions and using a JSON file is easier.

### 5. AWS Budget creation
  1. Got my AWS account ID from:
     ```sh
     aws sts get-caller-identity --query Account --output text
     ```
  2. Then, used the following command wit the required information:
     ```sh
     aws budgets create-budget \
     --acount-id AccountId \
     --budget file://aws/json/budget.json \
     --notification-with-subscribers file://aws/json/budget-notification-with-subscribers.json
     ```
     The contents of `budget.json` and `budget-notification-with-subscribers.json` were taken from: https://docs.aws.amazon.com/cli/latest/reference/budgets/create-budget.html#examples

## Homework challenges

### 1. Health alerts from services
  1. Created a SNS topic named `health-alerts` and suscribed with an email.
  2. Created EventBridge rule with `Health Event` for Triggering Events and `SNS Topic` for Targets. Then I let it create a new IAM role for the resource.

## Extra tasks

### Remove sensitive data

#### 1. Installation of Homebrew
  1. Installed Homebrew from: https://brew.sh/
  ```sh
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```
  2. Installed `bfg` package then followed the remaining commands:
  ```sh
  echo >> /home/vscode/.bashrc
  echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv bash)"' >> /home/vscode/.bashrc
  eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv bash)"
  ```
  3. Created `replacements.txt` for the sensitive data and moved it backward with `mv replacements.txt ..`.

#### 2. Other branches and tags
The following steps were done for other branches and tags. For `main`, it's just following the from the second step below.
  1. Cloned a mirror of the repository to be able to push the changes in all branches and tags.
  ```sh
  cd ..
  git clone --mirror HTTPS-repository-URL.git <folder_name>
  cd <folder_name>
  ```
  2. Pasted the following commands:
  ```sh
  bfg --replace-text ../replacements.txt
  ```
  3. It prompted to do:
  ```sh
  git reflog expire --expire=now --all && git gc --prune=now --aggressive
  ```
  4. Finally, did a `git push`.