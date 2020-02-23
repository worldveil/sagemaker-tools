## Sagemaker Idle

We want to be able to shutdown sagemaker notebooks automatically, to avoid paying for time we aren't using!

### [Lifecycle configuration script](https://docs.aws.amazon.com/sagemaker/latest/dg/notebook-lifecycle-config.html)

* Runs each time notebook starts
* Limit of 16384 characters
* Limit of 5 minute runtime
* In scripts, `$PATH`=`/usr/local/sbin:/usr/local/bin:/usr/bin:/usr/sbin:/sbin:/bin`
* Can setup connections to other AWS resources as well
* CloudWatch Logs for issues
	- Log group: `/aws/sagemaker/NotebookInstances`
	- Log stream: `[notebook-instance-name]/[LifecycleConfigHook]`

Some example scripts from AWS [can be found here](https://docs.aws.amazon.com/sagemaker/latest/dg/notebook-lifecycle-config.html).