Amazon Media Manager Jenkins Cloud Deployment Pipeline
======================

This repository is a collection of Ruby scripts, Cloud Formation Templates and Chef cookbooks used to set up a Jenkins server that will be used to deploy and manage the Cloud Deployment Amazon Media Manager (AMM) application Pipeline.

Most of these cookbooks are just copies of open source cookbooks. They were retrieved using berkshelf, since it makes everything way easier. If you need to update the open source cookbooks, it's simple enough; just add the new dependency to Berksfile, and then run these commands:

```
gem install berkshelf
berks install --path temp
cp -R temp/* .
rm -rf temp
```

(berkshelf seemed to nuke the directory that you pass in with --path, so don't just try passing in the current directory. That took me a little while to figure out...)

---

The custom cookbooks are as follows:
* jenkins-configuration: cookbooks to configure Jenkins jobs, views, etc.
* rvm: the one berkshelf pulls in is wicked old, and the override wasn't working. I just downloaded it manually.

how to use this repository
======================

This repository is designed to be used as the custom Chef cookbooks repository for a Jenkins stack built using Amazon's OpsWorks service.

We've designed the infrastructure for the Jenkins server to be run in a VPC. A CloudFormation template is provided to set up the VPC.

Also included in the repository is CloudFormation template that will handle building the appropriate IAM roles and OpsWorks stack. To run either template, you have a couple options.

The easier one is probably to clone this repository and run the Ruby script inside that will spin up the VPC, and then Jenkins server inside of it. To do this, [Ruby](https://www.ruby-lang.org/en/) needs to be installed on your system. You should probably install it with [RVM](http://rvm.io/) though. You'll also need the [AWS SDK for Ruby 2.0](https://github.com/aws/aws-sdk-core-ruby) which can be installed with `gem install aws-sdk-core --pre`.

Once both of those are installed, you can run this command to set everything up (the VPC and Jenkins server:

    ``ruby create_vpc_and_jenkins.rb --region aws-region-to-build-in --keyname your-ec2-keypair-name``

The parameters are:

* **keyname**: the name of an EC2 keypair that exists in that region. It will be linked to the NAT and Bastion host boxes that the VPC template creates.
* **region**: The AWS region you want to run everything in. Defaults to US-West-2, Oregon.

 Your other option is to run the template manually yourself (2 Cloud Formation scripts, 1 for the VPC, and 1 for the Jenkins server). You will need the [AWS CLI tool installed and configured](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html). Then, just pull down the repo and run these commands:

    aws cloudformation create-stack --stack-name "AMM-VPC" --template-body "`cat vpc.template`" --region your-region --parameters ParameterKey=KeyName,ParameterValue=your-ec2-keypair

    aws cloudformation create-stack --output json --disable-rollback --stack-name "AMM-Jenkins" --template-body "`cat jenkins.template`" --region your-region --capabilities="CAPABILITY_IAM" --parameters  ParameterKey=adminEmailAddress,ParameterValue="you@example.com"   ParameterKey=vpc,ParameterValue=the-vpc-id   ParameterKey=publicSubnet,ParameterValue=the-public-subnet-id   ParameterKey=privateSubnetA,ParameterValue=the-private-subnet-A-id ParameterKey=privateSubnetB,ParameterValue=the-private-subnet-A-id

The parameters for those templates are as follows:

* **region**: The AWS region you want to run everything in.
* **keyname**: the name of an EC2 keypair that exists in that region. It will be linked to the NAT and Bastion host boxes that the VPC template creates.
* **vpc**, **publicSubnet**, **privateSubnetA**, and **privateSubnetB** are outputs from the VPC template and must be inputted into the Jenkins template, so it knows where to build the instance and load balancer. It also saves this information for building Honolulu Answers application servers.

The Jenkins template also supports two other optional parameters: _repository_ and _branch_. If you'd like to specify a github repository other than the AMM app, you can pass in a parameter. The URL must be a github repository, and it must be a public repo. You can also specify a branch if you need one.

    --parameters ParameterKey=repository,ParameterValue=https://github.com/yourgithubrepo.git
    --parameters ParameterKey=branch,ParameterValue=your_branch_name
    
**Note**: When your Jenkins server comes up, it will have security turned on. The username / password will be admin / admin, though you'll likely want to change that.

* Log in to Jenkins as admin
* Click the "People" link on the right
* Click the "admin" link
* Click "configure"
* Punch in your new password in the password fields and click save.

how to update jenkins configuration:
====

If you've made changes to the Jenkins server configuration, it will not be persisted if the server goes down. If you'd like to commit that configuration to a source control repo, fork this repo and look in the jenkins-configuration cookbook. In there you will find various ERB template files, each full of XML. These are the raw Jenkins configuration files. You can find this XML by configuring the jobs on the Jenkins server, and then changing the URL. The Jenkins job configure URL will end in /jobname/configure; if you go to /jobname/config.xml you'll see the pure XML. 

The templates don't do much templating (only the source control repo URL) so you can just copy the XML and paste it into the template file.

questions?
====
If you have any issues, feel free to open an issue or make a pull request. Alternatively, you can reach out on twitter: @jonathansywulak

:books: 

## LICENSE

Copyright (c) 2014 Stelligent Systems LLC

MIT LICENSE

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
