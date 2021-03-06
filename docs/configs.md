---
title: 'Custom Configurations'
layout: page
---

# Contents
  * [Configuring a `staphb-tk` application](#configuring-a-staphb-tk-application)
    - [`staphb-tk` example: ivar](#staphb-tk-example:-ivar)
  * [Configuring a `staph-wf` workflow](#configuring-a-staph-wf-workflow)
    - [Tredegar Example Scenario](#tredegar-example-scenario)

# Configuring a `staphb-tk` application
Configuring Custom Docker Images and Parameters for the `staph-tk` command. By default the StaPH-B ToolKit `staphb-tk` utilizes the docker images listed [here](/staphb_toolkit/default_images).

To modify the images used by `staphb-tk`:
1. Create a copy of the Docker image configuration template file using the `$ staphb-tk --get_docker_config` command.
2. Edit the Docker configuration template file.
3. Specifying your custom configuration file (i.e. the edited Docker configuration template file) when executing the `staphb-tk` command.

## `staphb-tk` example: ivar

The steps to run the `ivar` application using the `andersonlabapps/ivar:1.1` Docker image rather than the default (`staphb/ivar:1.2.1`) image are:

#### 1.  Start by creating a copy of the Docker configuration template file
```
$ staphb-tk --get_docker_config
```
Running this command will save a Docker configuration template file to your current working directory as `<date>_docker_config.json`--in this example, `<date>` = `20-05-12`

```
$ ls
20-05-12_docker_config.json
```

Settings for each application in the Docker configuration template file are stored as JSON objects in the following format:

```
"<application>":{
        "image": "<Docker Image>",
        "tag": "<Docker Image Tag>",
        "params": "<Specified parameters>"
```  
<br>
#### 2. Edit the `20-05-12_docker_config.json` file to specify the appropriate image and tag for the `ivar` application.
In the example, the `ivar` JSON Object would be modified from:

```
"ivar":{
        "image": "staphb/ivar",
        "tag": "1.2.1",
        "params": ""
```

to:

```
"ivar":{
        "image": "andersenlabapps/ivar",
        "tag": "1.1",
        "params": ""
```
<br>
#### 3. Specifying your custom configuration file when executing the `staphb-tk` command:

```
$ staphb-tk -c 20-05-12_docker_config.json ivar version
iVar version 1.1
Please raise issues and bug reports at https://github.com/andersen-lab/ivar/
```
The `-c <custom_config_file>` flag must be made before specifying the application.


# Configuring a `staph-wf` workflow
Workflows within the StaPH-B ToolKit are written with default docker and singularity Nextflow profiles that can assigned with the `--profile` tag during execution. These profiles, along with workflow-specific configuration files, instruct the toolkit to run the workflow using local compute resources and designate the default docker images and parameters incorporated into each workflow. These settings are outlined for each workflow [here](/staphb_toolkit/workflows).

These default settings can be modified by:  
1. Creating a copy of the workflow configuration template file using the `$ staphb-wf <workflow> --get_config` command
2. Editing the workflow configuration template file
3. Specifying your custom configuration file (i.e. the edited workflow configuration template file) when executing the `staphb-wf <workflow>` command

## Tredegar Example Scenario
Adjusting the `tredegar` workflow to use 1) an AWS Batch environment rather than local compute resources and 2) a Trimmomatic quality score trimming threshold of Q30 rather than the default Q20, the following steps would apply:  

#### 1.  Start by creating a copy of the `tredegar` configuration template file

```
$ staphb-wf tredegar --get_config
```
Running this command will save a `tredegar` configuration template file to your current working directory as `<date>_docker_config.json`--in this example, `<date>` = `20-05-12`

```
$ ls
20-05-12_tredegar.config
```

Workflow configurations are stored in a Nextflow-readable format to specify the run environment (Docker, Singularity,AWS Batch,etc ...), Docker Container information, pipeline parameters, and Nextflow process parameters. To read more about the Nextflow configuration visit the latest [documentation](https://www.nextflow.io/docs/latest/config.html).

#### 2. Edit the `20-05-12_tredegar.config` to specify an AWS Batch environment and adjust the `Trimmomatic` quality-score trimming threshold to Q30

To specify an AWS Batch environment, the `##AWS Batch Params##` section must be uncommented and filled in accordance to the user's AWS Batch environment. In this example, the `##AWS Batch Params##` section would be modified from:<br /><br />
```
//####################
//##AWS Batch Params##
//####################
//aws.batch.cliPath = '/home/ec2-user/miniconda/bin/aws'
//aws.region = 'us-east-1'
//workDir = 's3://'
```
to:
```
//####################
//##AWS Batch Params##
//####################
aws.batch.cliPath = '<path_to_aws_cli>'
aws.region = '<aws_region>'
workDir = 's3://<Nextflow_work_directory>/'
```

The awsbatch executor and queue must also be defined under the `process` section. In this example, the `process` section would be modified from:

```
process {
      //stageInMode = "link" //uncomment for singularity\
      //executor = 'awsbatch' //uncomment for AWS Batch
      //queue = '' //uncomment for AWS Batch
```
to:
```
process {
    //stageInMode = "link" //uncomment for singularity
    executor = 'awsbatch' //uncomment for AWS Batch
    queue = '<name_of_aws_queue>' //uncomment for AWS Batch
```

To adjust the `Trimmomatic` quality-score trimming threshold to Q30, the `###Pipeline Params###` section would be modified from:

```
//#####################
//###Pipeline Params###
//#####################
[...]
//Trimming
params.minlength=100
params.windowsize=10
params.qualitytrimscore=20
threads=4
```
to:
```
//#####################
//###Pipeline Params###
//#####################
[...]
//Trimming
params.minlength=100
params.windowsize=10
params.qualitytrimscore=30
threads=4
```
<br>
#### 3. Specify the custom configuration file (i.e. the edited workflow configuration template file) when executing the `staphb-wf tredegar` command

```
$ staphb-wf tredegar <reads_dir> -c 20-05-12_tredegar.config
```
