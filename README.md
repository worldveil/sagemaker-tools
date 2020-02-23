## Sagemaker Idle

We want to be able to shutdown sagemaker notebooks automatically, to avoid paying for time we aren't using!

### [Lifecycle configuration script](https://docs.aws.amazon.com/sagemaker/latest/dg/notebook-lifecycle-config.html)

* Runs each time notebook starts
* Script details:
	* Runs as `root`
		- Notebooks run as `ec2-user`, so to get that user in shell: `sudo -u ec2-user`
	* Limit of 16384 characters
	* Limit of 5 minute runtime
	* In scripts, `$PATH`=`/usr/local/sbin:/usr/local/bin:/usr/bin:/usr/sbin:/sbin:/bin`
* Can setup connections to other AWS resources as well
* CloudWatch Logs for issues
	- Log group: `/aws/sagemaker/NotebookInstances`
	- Log stream: `[notebook-instance-name]/[LifecycleConfigHook]`

Tons more details for iterating through [Conda environments](https://docs.aws.amazon.com/sagemaker/latest/dg/notebook-lifecycle-config.html).

Some example scripts from AWS [can be found here](https://github.com/aws-samples/amazon-sagemaker-notebook-instance-lifecycle-config-samples).

### Recommended Setup

1. Turn off notebook
1. Create new lifecycle configuration with the below script
1. Click "Edit" notebook in AWS console, associate lifecycle with notebook
1. Restart notebook and check CloudWatch logs to make sure it worked

```shell
#!/bin/bash

set -e

# OVERVIEW
# This script stops a SageMaker notebook once it's idle for more than 1 hour (default time)
# You can change the idle time for stop using the environment variable below.
# If you want the notebook the stop only if no browsers are open, remove the --ignore-connections flag
#
# Note that this script will fail if either condition is not met
#   1. Ensure the Notebook Instance has internet connectivity to fetch the example config
#   2. Ensure the Notebook Instance execution role permissions to SageMaker:StopNotebookInstance to stop the notebook 
#       and SageMaker:DescribeNotebookInstance to describe the notebook.
#

# PARAMETERS
IDLE_TIME=10800

echo "Fetching the autostop script"
wget https://raw.githubusercontent.com/aws-samples/amazon-sagemaker-notebook-instance-lifecycle-config-samples/master/scripts/auto-stop-idle/autostop.py

echo "Starting the SageMaker autostop script in cron"

(crontab -l 2>/dev/null; echo "5 * * * * /usr/bin/python $PWD/autostop.py --time $IDLE_TIME --ignore-connections") | crontab -
```

## Sagemaker Knockout

Seems a promising way to manage and use CPU/GPU load.

Note that if you use the [Sagemaker Knockout](https://github.com/mariokostelac/sagemaker-knockout) starting script, it will fail in the AWS console!

	Uncaught DOMException: Failed to execute 'btoa' on 'Window': The string to be encoded contains characters outside of the Latin1 range.

You have to remove the 🥊 character.

I haven't been able to make this work yet, pending [this pidfile issue](https://github.com/mariokostelac/sagemaker-knockout/issues/5).

## Using [AWS Shell](https://github.com/awslabs/aws-shell) to manage

If you want to [update your lifecycle configuration for existing notebook](https://aws.amazon.com/blogs/machine-learning/lifecycle-configuration-update-for-amazon-sagemaker-notebook-instances/):

```shell
pip3 install -r requirements.txt
aws-shell  # will take a bit to create autocomplete index...
```

Create an IAM user with `SagemakerFullAccess` policy. This lets it control sagemaker along with a few other adjacent things, like particular S3 buckets and keys.

I usually add these to `~/.aws/credentials` file in the format:

```
[myprofile]
output = json
region = us-east-1
aws_access_key_id = xxxxxx
aws_secret_access_key = xxxxxxx
```

Then in the shell:

```shell
aws-shell
aws> .profile myprofile  # activate your profile!
aws> sagemaker update-notebook-instance --lifecycle-config-name my-config --notebook-instance-name mynotebook-name
aws> .exit
```

If you prefer, you can also do this with [`boto3`](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sagemaker.html#SageMaker.Client.update_notebook_instance).
