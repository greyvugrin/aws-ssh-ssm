# Intro
SSH into AWS EC2 instances securely, using MFA + AWS temporary credentials.
- ✅ No hardcoded IP addresses
- ✅ No shared PEM files. Uses temporary SSH keys
- ✅ Access controlled in AWS

# Requirements

## Packages
- OpenSSH > 7.3p1. Check with `ssh -V`
- [AWS CLI v2 installed](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [AWS CLI configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config)
- [SSM AWS CLI plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html#install-plugin-macos-signed) installed
- `pip` installed.
  ```
  curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
  python3 get-pip.py
  ```
- AWS MFA pip package installed. `pip install aws-mfa`

## AWS

- Each user must have a MFA device assigned. LINK
- Instances
  - Each instance must have a policy attached to its role that enables SSM
  - The role must be tagged with the correct user for SSM to run (ex: `ubuntu` for Ubuntu AMIs)
  - The SSM agent must be running (AWS Linux & Ubuntu instances have this by default)

# Installation
Get the location of this folder with `pwd`. Copy and replace it in the commands below where you see *_FOLDER_LOCATION_*.
Ex: /Users/gvugrin/Desktop/code/aws-ssh-ssm

## Add to PATH
Add this folder to your $PATH. This will let you run the shell scripts here from any location.
Add this line to the end of your ~/.zshrc or ~/.bashrc, depending on which shell you use.

```
PATH=_FOLDER_LOCATION_:$PATH
```
*Note: Add `export PATH` below that line if it's not already included.*

## Import ssh config

This is the ✨ magic ✨. It defines the aliases for your instance hostnames (autocomplete) and runs a script lets you use SSM to ssh into them.

Insert these into your `~/.ssh/config` as **the first lines**.

```
Include _FOLDER_LOCATION_/ssh_config_base
Include _FOLDER_LOCATION_/ssh_config_hosts
```

# Configure

## Create your configuration file & fill it out
```
cp config.example config
```

Fill out the config file variables.

| Variable | Description | Required |
| --- | --- | --- |
| duration | Number of seconds the credentials will be valid for. Default is 3600 (1 hour). | No |
| username | The username of your IAM user.  | Yes |

## Create your SSH hosts file & add hosts

```
cp ssh_config_hosts.example ssh_config_hosts
```

Your ssh_config_hosts file is where you define the hostnames you'll use to ssh into your instances.
It's not version controlled, so you can add/remove hosts as you need without worrying about conflicts with upstream.

For each one, you only need to provide:
- a memorable host identifier (starting with the aa_ prefix)
- the instance ID.


# Usage

## 1 - Create/refresh credentials:
```
iamwhoiam
```
You will be asked to enter your MFA code. A temporary set of AWS credentials will be created and stored in your `~/.aws/credentials` file. They are valid for the duration you specified in the config file.

As long as they're valid you'll be able to interact with the instance.

## 2 - SSH/SCP/whatever into instance name.

This leverages normal ssh & scp behaviors, so this all stock.
Hostnames will autocomplete. See ssh config file for full list.

```
ssh aa_your_instance_name

# copy file from instance to local
# Note that scp remote path autocomplete suggestions can be slow. Give it time
scp aa_your_instance_name:test-html/index.html .
```

# Troubleshooting

**Instance is offline**
- Check SSM panel in AWS to see the list of running instances
- You may need to restart the instance

# TODO

- See about AWS_REGION in SSH proxy command
- Improve AWS requirements portion for instance, IAM setup
