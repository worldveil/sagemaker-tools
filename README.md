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

Some example scripts from AWS [can be found here](https://docs.aws.amazon.com/sagemaker/latest/dg/notebook-lifecycle-config.html).

Note that if you use the [Sagemaker Knockout](https://github.com/mariokostelac/sagemaker-knockout) starting script, it will fail in the AWS console!

	Uncaught DOMException: Failed to execute 'btoa' on 'Window': The string to be encoded contains characters outside of the Latin1 range.

You have to remove the 🥊 character.

## Adding a Lifecycle Script

You can just add one when you create a notebook.

Otherwise, you need to do a little more work for an existing notebook.

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

#### Steps to update lifecycle instance for existing notebook

We'll be making use of the [`UpdateNotebookInstance`](https://docs.aws.amazon.com/sagemaker/latest/dg/API_UpdateNotebookInstance.html) API endpoint.

1. Go to Sagemaker AWS Console and create a new Lifecycle configuration
1. Associate that configuration with the notebook using the AWS Shell (see below)
1. Restart the notebook!
1. Check the CloudWatch (`/aws/sagemaker/NotebookInstances`) logs to check that the `echo` or whatever statements ran as a sanity check.

Example using the shell to associate the configuration with the (currently not running) notebook:

```shell
aws-shell
aws> .profile myprofile  # activate your profile!
aws> sagemaker update-notebook-instance --lifecycle-config-name my-config --notebook-instance-name mynotebook-name
aws> .exit
```





