# AWS Resource Listing Automation Using Bash


---


A beginner-friendly DevOps shell-scripting project that uses the **AWS
Command Line Interface (AWS CLI)** to list active AWS resources in a
selected AWS Region.

> **Project goal:** A user supplies an AWS Region and an AWS service
> name. The script validates the inputs, checks that AWS CLI is
> installed and configured, and runs the matching AWS CLI command.

------------------------------------------------------------------------

## 1. What does this project do?

DevOps teams need to know which cloud resources are active in their AWS
accounts. This helps teams review their environment and support
cost-optimization activities.

This project creates a Bash script named `aws_resource_list.sh`.

The user runs the script with two command-line arguments:

1.  AWS Region
2.  AWS service name

For example:

``` bash
./aws_resource_list.sh ap-south-1 ec2
```

The script then lists the EC2 instances in the `ap-south-1` Region.

For another service, change the second argument:

``` bash
./aws_resource_list.sh ap-south-1 s3
```

This lists the S3 buckets visible to the configured AWS identity. Note
that S3 buckets are a global service; the Region argument is still part
of this project's common input format, but it does not make S3 buckets
regional.

------------------------------------------------------------------------

## 2. Why is this project useful?

This is an example of a common type of DevOps automation:

-   Helps teams inspect AWS resources.
-   Reduces the need to run separate commands manually.
-   Demonstrates Bash scripting and command-line argument handling.
-   Uses AWS CLI instead of manually building AWS API requests.
-   Can be extended to support more AWS services.
-   Can be scheduled with cron as an optional improvement.

The tutorial describes this type of report as useful to managers,
developers, and others who need visibility into active AWS resources. It
also explains that organizations should choose the services they
actually use rather than trying to support every AWS service.

------------------------------------------------------------------------

## 3. Technologies used

  Technology               Purpose
  ------------------------ --------------------------------------------
  Bash / Shell scripting   Automates the workflow
  AWS CLI                  Sends commands to AWS
  AWS account              Provides access to AWS resources
  Linux / Ubuntu           Environment for running the script
  Visual Studio Code       Editing the script
  EC2 (optional)           A Linux machine on which to run the script
  cron (optional)          Schedules the script to run automatically

------------------------------------------------------------------------

## 4. Prerequisites

Before starting, make sure you have:

-   An AWS account.
-   A Linux machine, or an Ubuntu EC2 instance.
-   AWS CLI installed.
-   An AWS identity with permission to list the resources you want to
    inspect.
-   A terminal.
-   Basic familiarity with Linux commands.

**Important:** Use an IAM user or role with only the permissions needed
for this task. Avoid using your AWS root account for everyday work.

### AWS credentials and security

The tutorial demonstrates configuring AWS CLI with an access key and
secret access key. For real projects, prefer temporary credentials or an
attached IAM role when possible.

-   Never write access keys or secret keys inside the script.
-   Never upload credentials, private keys, or your AWS credentials file
    to GitHub.
-   Do not share your secret access key.
-   Remove or rotate access keys that are no longer needed.
-   Give the identity only the permissions required to list the selected
    resources.

------------------------------------------------------------------------

## 5. Create the project folder

Open your terminal and run:

``` bash
mkdir aws-resource-list
cd aws-resource-list
```

Create the script file:

``` bash
touch aws_resource_list.sh
```

Open the folder in Visual Studio Code:

``` bash
code .
```

If the `code` command is not available, open Visual Studio Code manually
and choose **File → Open Folder**.

------------------------------------------------------------------------

## 6. Plan the script before writing code

The script should follow this order:

1.  Add a Bash shebang and script information.
2.  Check that exactly two arguments were supplied.
3.  Store the Region and service name in variables.
4.  Check that AWS CLI is installed.
5.  Check that AWS CLI is configured and credentials are available.
6.  Validate that the requested service is supported.
7.  Use a `case` statement to run the correct AWS CLI command.
8.  Display a useful error if the service is not supported.

This order makes it easier for a user to understand why the script
cannot run.

------------------------------------------------------------------------

## 7. Add the script header and documentation

At the top of `aws_resource_list.sh`, add:

``` bash
#!/bin/bash

#############################################
# Script Name: aws_resource_list.sh
# Description: Lists selected AWS resources
#              in a specified AWS Region.
# Author: Your Name or DevOps Team
# Version: 0.1.0
#
# Usage:
#   ./aws_resource_list.sh <region> <service>
#
# Example:
#   ./aws_resource_list.sh ap-south-1 ec2
#############################################
```

Replace `Your Name or DevOps Team` with your own name or team name.

Update the version when you make meaningful changes to the script. Start
with a development version such as `0.1.0`; use a stable version such as
`1.0.0` when the script is ready for regular use.

------------------------------------------------------------------------

## 8. Understand command-line arguments

The script expects two arguments:

``` bash
./aws_resource_list.sh ap-south-1 ec2
```

In Bash:

-   `$0` is the script name.
-   `$1` is the first argument (`ap-south-1`).
-   `$2` is the second argument (`ec2`).
-   `$#` is the number of arguments supplied.

The script should stop if the user does not supply exactly two
arguments. It should print the correct usage and return a non-zero exit
status.

Example validation:

``` bash
if [ "$#" -ne 2 ]; then
    echo "Usage: $0 <region> <service>"
    echo "Example: $0 ap-south-1 ec2"
    exit 1
fi

REGION="$1"
SERVICE="$2"
```

**Why validate first?** If a required argument is missing, the script
should explain the correct format instead of continuing with incomplete
information.

------------------------------------------------------------------------

## 9. Check whether AWS CLI is installed

The script uses AWS CLI to communicate with AWS. It does not build AWS
API requests itself.

Check whether the `aws` command is available:

``` bash
if ! command -v aws >/dev/null 2>&1; then
    echo "Error: AWS CLI is not installed."
    exit 1
fi
```

You can also check the installed version manually:

``` bash
aws --version
```

If AWS CLI is missing, install it by following the official AWS
instructions:

[Install or update the AWS
CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

For Ubuntu, follow the current Linux installation instructions in the
AWS documentation rather than relying on an old copied installation
command.

------------------------------------------------------------------------

## 10. Configure AWS CLI

AWS CLI must have credentials and a default configuration, unless you
use another supported credential method such as an attached IAM role.

For local learning, you can configure a profile interactively:

``` bash
aws configure
```

The CLI prompts for:

-   AWS Access Key ID
-   AWS Secret Access Key
-   Default region name
-   Default output format

For example, the Region could be `ap-south-1`, and the output format
could be `json`.

Check that the CLI can identify the configured AWS identity:

``` bash
aws sts get-caller-identity
```

If this command fails, check your credentials, profile, network
connection, and IAM permissions.

**Security warning:** Do not paste credentials into this README, a shell
script, screenshots, or a public GitHub repository. Do not create
long-lived access keys just for convenience if an IAM role or temporary
credentials are available.

------------------------------------------------------------------------

## 11. Check AWS CLI configuration in the script

A simple check is to run an AWS CLI command that requires valid
credentials:

``` bash
if ! aws sts get-caller-identity >/dev/null 2>&1; then
    echo "Error: AWS CLI credentials are not configured or are not working."
    exit 1
fi
```

This is more reliable than checking only whether a local configuration
directory exists. A configuration file can exist while its credentials
are invalid or expired.

------------------------------------------------------------------------

## 12. Run the command for the selected service

Use a Bash `case` statement to choose a command based on the service
argument.

The tutorial explains this pattern:

``` bash
case "$SERVICE" in
    ec2)
        aws ec2 describe-instances --region "$REGION"
        ;;
    s3)
        aws s3 ls
        ;;
    *)
        echo "Error: Unsupported service: $SERVICE"
        exit 1
        ;;
esac
```

In this example:

-   `ec2)` runs the EC2 command.
-   `s3)` runs the S3 command.
-   `*)` handles any other input.
-   `esac` ends the `case` statement.

The tutorial discusses adding other services such as EBS, RDS, DynamoDB,
ELB, VPC, CloudFormation, and IAM. Add a service only after checking the
correct AWS CLI command and its permissions in the official
documentation.

AWS CLI command reference:

<https://docs.aws.amazon.com/cli/latest/reference/>

**Note:** The example above is a small learning example, not a complete
production script. For example, the S3 listing command lists buckets
available to the identity and does not use the supplied Region in the
same way as the EC2 command.

------------------------------------------------------------------------

## 13. Make the script executable

From the project folder, run:

``` bash
chmod +x aws_resource_list.sh
```

Now you can execute it using:

``` bash
./aws_resource_list.sh ap-south-1 ec2
```

If you do not want to set the executable bit, you can run it with Bash:

``` bash
bash aws_resource_list.sh ap-south-1 ec2
```

------------------------------------------------------------------------

## 14. Test the script

Test both successful and unsuccessful inputs.

### Test 1: Missing arguments

``` bash
./aws_resource_list.sh
```

Expected result: the script prints the usage instructions and exits.

### Test 2: Only one argument

``` bash
./aws_resource_list.sh ec2
```

Expected result: the script prints the usage instructions and exits.

### Test 3: EC2 resource listing

``` bash
./aws_resource_list.sh ap-south-1 ec2
```

Expected result: AWS CLI returns EC2 instance information that the
configured identity is allowed to view in that Region.

### Test 4: S3 bucket listing

``` bash
./aws_resource_list.sh ap-south-1 s3
```

Expected result: AWS CLI lists the S3 buckets visible to the configured
identity.

### Test 5: Unsupported service

``` bash
./aws_resource_list.sh ap-south-1 fsx
```

Expected result: the script prints an unsupported-service error and
exits.

### Test 6: AWS CLI unavailable or credentials not working

Try the script in an environment where AWS CLI is not installed, or
where valid credentials are not available.

Expected result: the script prints a clear error and stops.

------------------------------------------------------------------------

## 15. Optional: Run the script on an EC2 instance

The tutorial demonstrates editing the script in Visual Studio Code and
running it on an Ubuntu EC2 instance.

High-level steps:

1.  Launch an Ubuntu EC2 instance in your chosen Region.
2.  Choose an appropriate key pair and store the private key securely.
3.  Connect to the instance using SSH.
4.  Install AWS CLI by following the official AWS installation guide.
5.  Configure access using a suitable IAM role attached to the instance,
    preferably, or another approved credential method.
6.  Copy the script to the instance.
7.  Make it executable with `chmod +x`.
8.  Run it with a Region and service name.

Example SSH command (replace the key filename and public IP):

``` bash
ssh -i /path/to/key.pem ubuntu@<EC2-PUBLIC-IP>
```

Do not make an EC2 instance publicly accessible unless required.
Restrict SSH access and review AWS costs. Stop or terminate resources
when you no longer need them, while remembering that some attached
resources may continue to incur charges.

------------------------------------------------------------------------

## 16. Optional: Schedule the script with cron

The tutorial suggests extending the project by scheduling it to run
every day at 9:00 p.m.

Cron is a Linux job scheduler. You can edit your user's crontab with:

``` bash
crontab -e
```

A schedule for 9:00 p.m. every day has this format:

``` cron
0 21 * * * /absolute/path/to/aws_resource_list.sh ap-south-1 ec2
```

Replace the example path and arguments with your actual values.

For a real scheduled report, consider writing output to a log file,
handling failures, and arranging an approved reporting or notification
method. The tutorial does not implement email reporting.

------------------------------------------------------------------------

## 17. File permissions and security

The tutorial explains Linux permission numbers:

-   `4` = read
-   `2` = write
-   `1` = execute

For example:

``` bash
chmod 771 aws_resource_list.sh
```

This gives the owner and group read, write, and execute permissions,
while others get execute permission only.

Use permissions that match your team's needs. Do not assume that
execute-only access is automatically the safest choice for every
environment. Also remember that anyone who can execute a script may be
able to observe its behavior or its output.

Never put AWS secrets in the script. Use IAM permissions to control what
AWS actions the script can perform.

------------------------------------------------------------------------

## 18. Common errors and troubleshooting

  -----------------------------------------------------------------------
  Problem                             What to check
  ----------------------------------- -----------------------------------
  `AWS CLI is not installed`          Install AWS CLI and check with
                                      `aws --version`.

  Credentials are not working         Run `aws sts get-caller-identity`
                                      and review the selected profile or
                                      IAM role.

  Access denied                       Ask an administrator to review the
                                      IAM permissions required for the
                                      listing command.

  No EC2 instances shown              Check the Region, account, filters,
                                      and whether any instances exist
                                      there.

  Unsupported service                 Check the supported service names
                                      in the script's `case` statement.

  Script cannot be executed           Run `chmod +x aws_resource_list.sh`
                                      or use
                                      `bash aws_resource_list.sh ...`.

  Invalid Region                      Check the Region spelling against
                                      AWS's available Regions.
  -----------------------------------------------------------------------

------------------------------------------------------------------------
------------------------------------------------------------------------

## 19. Useful official documentation

-   [AWS CLI
    installation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
-   [AWS CLI command
    reference](https://docs.aws.amazon.com/cli/latest/reference/)
-   [AWS CLI
    configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
-   [AWS IAM best
    practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
-   [Bash manual](https://www.gnu.org/software/bash/manual/)

------------------------------------------------------------------------

## 20. Project completion checklist

-   [ ] Create the project folder.
-   [ ] Create `aws_resource_list.sh`.
-   [ ] Add the script header and usage comments.
-   [ ] Validate that exactly two arguments are supplied.
-   [ ] Check that AWS CLI is installed.
-   [ ] Check that AWS credentials are working.
-   [ ] Add the supported service commands.
-   [ ] Handle unsupported service names.
-   [ ] Make the script executable.
-   [ ] Test valid and invalid inputs.
-   [ ] Review IAM permissions and credential safety.
-   [ ] Add the files to GitHub without committing secrets.
-   [ ] Try the optional cron assignment.


