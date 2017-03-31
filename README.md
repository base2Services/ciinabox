# Ciinabox

## Intro

Ciinabox is set of tools that was created in order to leverage creation and
management of large numbers of CI/CD environments, allowing user to treat
set of CI tools and automation tasks as code. These tools are responsible for
following

- Provisioning of underlying infrastructure for CI Tools
- Provisioning of CI/CD tools themselves
- Provisioning of automation tasks in repeatable manner. Storing these tasks
  in a human readbale format, and treating them as a code

## Repositories

- [ciinabox-ecs](https://github.com/base2Services/ciinabox-ecs) is responsible for provisioning infrastructure and CI tools on AWS.
Any automation servers are provisioned as [Docker](https://www.docker.com) containers. In it's default setup
ciinabox-ecs provisions [Jenkins](https://jenkins.io) automation server containers automation server,
however it is possible to rpvoision any automation server/tool that can be run as
a Docker container

- [ciinabox-containers](https://github.com/base2Services/ciinabox-containers) is set of docker container definition for automation tools. Most of
tooling is focused around Jenkins ecosystem. Pull requests are welcome :)

- [ciinabox-jenkins](https://github.com/base2Services/ciinabox-jenkins) is command line utility for provisioning Jenkins tasks from human readbale
YAML files. These YAML files can than be stored as code in repository and versioned,
treating your automation tasks as a code.


- [ciinabox-pipelines](https://github.com/base2Services/ciinabox-pipelines) is [Jenkins pipelines library](https://jenkins.io/doc/book/pipeline/shared-libraries/) aiding in orchestration of existing Jenkins.
This library also contains methods that should enable it's users to build new pipeline jobs
utilising methods such as `dockerBuild` for building docker images, or `withECR`
for executing Groovy closure while logged into Amazon's Elastic Container Service in case your pipeline is running

## Getting started

### Create Ciinabox environment on AWS

Before commencing to steps below, be aware that you will incur charges on
your AWS account (1 NAT Gateway, 1 t2.large instance and 1 ELB). Also note
that instance type can be tweaked using configuration files. Once environment
has been torn down, it will leave EBS snapshots behind, as deletion policies
are set to `Snapshot` for data drives.

#### Requirements

- AWS Account
- Route53 hosted zone for your tools (`tools.aws.example.net` in example below)
- S3 Bucket for Cloud Formation templates (optional, can be automatically created)

#### Creating ciinabox

Ciinabox creation consists of following steps

1. Generation of configuration files
2. (Optional) creation and upload of server certificates
3. (Optional) creation of ciinabox key
4. (Optional) creation of s3 bucket with Cloud Formation templates

If you have previously deployed ciinabox to your AWS account, you won't need
steps 2 and 3, as step 4 can be done either manually or trough setup script.
Each of this steps can be executed in isolation, as separate rake task.
Fully guided install is available trough `ciinabox:full_install` rake task.
Type `rake -t` for more info.

- Clone the repo and install required gems

```
$ git clone https://github.com/base2Services/ciinabox-ecs.git

ciinabox-ecs $ bundle install
Using rake 12.0.0
Using cfndsl 0.12.4
Using bundler 1.13.3
Bundle complete! 2 Gemfile dependencies, 3 gems now installed.
Use  \`bundle show [gemname]\` to see where a bundled gem is installed.
```

- `ciinabox:full_install` rake task will guide you trough whole process of generating
  configuration files new toolset, and provisioning all resources in your
  AWSS account. Only 2 mandatory parameters are name of your ciinabox installation,
  determining local path for configuration files, and Route53 hosted zone for newly
  setup tools. Full install will wait for completion of stack creation before
  exiting

```
ciinabox-ecs $ bundle exec rake ciinabox:full_install
Enter the name of your ciinabox:
demo
Enter the AWS region to create your ciinabox [us-east-1]:

Using us-east-1 as AWS region
Enter the name of the S3 bucket to deploy ciinabox to [ciinabox-deployment-0bce48b2-8eda-405e-a26f-8653882d3956]:

Enter top level domain (e.g tools.example.com), must exist in Route53 in the same AWS account:
tools.aws.example.net
Enter AWS profile you wish to use for provisioning (empty for default):

Using AWS Account 111111111111
Enter the name of created Cloud Formation stack [ciinabox]:
ciinabox-demo
Using user-provided services.yml File
# Enable active ciinabox by executing or override ciinaboxes base directory:
export CIINABOXES_DIR="ciinaboxes"
export CIINABOX="demo"
# or run
# eval "$(rake ciinabox:active[demo])"
Create source bucket (y/n)? [y]
y
make_bucket: s3://ciinabox-deployment-0bce48b2-8eda-405e-a26f-8653882d3956/
Successfully created S3 source deployment bucket ciinabox-deployment-0bce48b2-8eda-405e-a26f-8653882d3956
Create and upload server certificate (y/n)? [y]
n
Create and upload ciinabox key (y/n)? [y]
n
......
..... # Output of template generation
......
Successfully uploaded rendered templates to S3 bucket ciinabox-deployment-0bce48b2-8eda-405e-a26f-8653882d3956
{
    "StackId": "arn:aws:cloudformation:us-east-1:111111111111:stack/ciinabox-demo/xxxxxxx"
}
Starting creation of ciinabox environment
Waiting for Cloud Formation stack creation completion ...
```

### Making changes to ciinabox environment


### Publishing jobs to Jenkins

Jenkins jobs can be provisioned by using [ciinabox-jenkins](https://github.com/base2Services/ciinabox-jenkins)
CLI utility.
Please see repo's [README](https://github.com/base2Services/ciinabox-jenkins/blob/master/README.md) for reference on job dsl, and yaml file format.
Once you check out the repo, there will be some example jobs within `ciinaboxes.example/example/jenkins/dsl-reference-jobs.yml` file.
To provision these sample jobs to your newly created ciinabox environment, use command below.
Example assumes that default username/password hasn't been changed.

```
ciinabox-jenkins $ ./gradlew jenkins -Dusername=ciinabox -Dpassword=ciinabox -Durl=http://jenkins.aws.example.net -Dciinaboxes=ciinaboxes.example -Dciinabox=example

```

Info output on all jobs created should be written to console

```
....
processing job: MyJobWithParameters
Processing provided DSL script
dsl-doc - updated
dsl-doc/MyJobWithParameters - created

processing job: MyJobWithDescription
Processing provided DSL script
dsl-doc - updated
dsl-doc/MyJobWithDescription - created

BUILD SUCCESSFUL

Total time: 20.255 secs

```
