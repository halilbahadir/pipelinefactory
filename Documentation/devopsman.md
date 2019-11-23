#DEVOPS ENGINEER


As a devops engineeer, you are responsible for creating and managing the TEST / PROD Stages resources in an automated way. To do so you will be designing a pipeline using AWS CodePipeline services. As you can see below, 

<img src="https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/8-Merged-Test-Prod.png" width="450" height="500">

 1. adimlari acikla...
 
 
 
 ## Let's do it together..
 
 1. Login to AWS Web console as devops engineer. I use the AWS Identity Access Management user named as _devopsman_ in this sample.
 
 2. Check that you are in AWS Region US-WEST-2 (Oregon)
 
  ![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/1-aws-dashboard.png)
 
 3. Switch to _AWS Cloud9_ service dashboard.
 
 ![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/2-Cloud9-dashboard.png)
 
 4. Create an AWS Cloud9 environment for DevOps related activities for devops engineer.
  * Step 1
    * Name: **ProjectQ**
    * Desciption **Cloud9 for Project Q**  
  * Step 2 (Choose all default values)
  * Step 3 (Review and Create Environment)
  
  AWS Cloud9 IDE Environment will be created in a couple of minutes
  
  ![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/3-Cloud9-IDE-Created.png)
  
  5. As a company policy and DevOps best practices, every source code, configuration file, documentation etc. are under version control, version controling can be done by AWS CodeCommit or Amazon S3 depending on the needs. We have AWS Cloudformation Templates repository stored and versioned with AWS CodeCommit. For the ProjectQ needs I will reuse existing templates and configure according to the ProjectQ architecture.
  We will clone the AWS Cloudformation Templates repository to our AWS Cloud9 environment.
  
 6. In AWS Cloud9, run the commands in _terminal_ view.
 
 ```console
 #Delete the README.md file, that comes with Cloud9 IDE
 devopsman:~/environment $ rm README.md 
 
 
 #Clone the AWS CodeCommit Repository to your AWS Cloud9 Environment
 #You will need Git credentials to connect to AWS CodeCommit repository. 
 #You can create your Git Credentials from AWS IAM - https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html
 devopsman:~/environment $ git clone https://git-codecommit.us-west-2.amazonaws.com/v1/repos/CFN-Templates-Repo
 
 #Git configurations - email, username, caching credential etc.
 devopsman:~/environment $ git config --global user.email devopsman@amazon.com
 devopsman:~/environment $ git config --global user.name "devopsman"
 devopsman:~/environment $ cd CFN-Templates-Repo/
 devopsman:~/environment $ git config credential.helper cache --timeout=3000
 
 ```
 
 7. First we will create the TEST stage's resources as defined in architecture diagram above. In AWS Cloud9 IDE, ProjectQ/CFN-Templates-Repo/EnvScripts folder stores the AWS CloudFormation template and configuration files. The main AWS CloudFormation template file is **StackPipeline.yml**
 _StackPipeline.yml_ will create the pipeline, using AWS CodePipeline Service. As you can see in figure below, this pipeline will have XXX actions
 
 <img src="https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/8-Merged-Test-Prod.png" width="450" height="500">
 
 
 8. You can do some parameter changes for the architecture in **test-stack-configuration.yaml** and **prod-stack-configuration** files, according to our project needs.
 
 For example: You can change the VPCIPRange in TEST environment, if you want your CIDR range "10.40.0.0/16" than set  "VPCIpRange": "10.40". 
 
 
 ```json
 {
  "Parameters" : {
    "VPCIpRange": "10.32",
    "KeyName" : "EnvKeyPair-Oregon",
    "InstanceType" : "t3.small",
    "SSHLocation" : "0.0.0.0/0",
    "DBClass" : "db.t2.medium",
    "DBName" : "TestDB",
    "DBUser" : "TestDBuser",   
    "DBPassword" : "TestDBPassword",
    "MultiAZDatabase" : "false",
    "WebServerCapacity" : "1",
    "DBAllocatedStorage" : "10",
    "EnvironmentStageTag" : "TEST"
  }
}
```
 
 9. Save all the files that you have made changes in AWS Cloud9.
 
 10. Run the commands below to commit the changes to your git repository.
 
 ```console
 #Check the status of the files in git
 devopsman:~/environment/CFN-Templates-Repo (master) $ git status
 
 #Commit the changes to git
 devopsman:~/environment/CFN-Templates-Repo (master) $ git commit -a -m "updated for ProjectQ"
 
 #Push the changes to the origin in AWS CodeCommit Repository.
 devopsman:~/environment/CFN-Templates-Repo (master) $ git push origin master 
 
 ```
 
 11. We will execute the template in AWS CloudFormation to generate the resources that e defined. AWS CloudFormation supports uploading template files from S3 Bucket or Local file system (like your laptop). Because of that, we need to upload our AWS CloudFormation template files (yaml) and configuration files (json) to an Amazon S3 Bucket.   
 
 ```console
 #Create an Amazon S3 bucket and upload your files to S3 bucket..You cannot use CAPITAL letter in S3 bucket names.
 devopsman:~/environment/CFN-Templates-Repo (master) $ aws s3api create-bucket --bucket hb-projectq --region us-west-2 --create-bucket-configuration LocationConstraint=us-west-2  
 
 #Check if the projectq bucket is created. You must see the 'projectq' in the list of command output.
 devopsman:~/environment/CFN-Templates-Repo (master) $ aws s3 ls
 
 #Upload your all files in AWS Cloud9 to recently created 'projectq' Amazon S3 bucket.
 devopsman:~/environment/CFN-Templates-Repo (master) $ aws s3 sync . s3://hb-projectq
 
 #Check if the files are uploaded. You must see all files in your AWS Cloud9 environment's folder CFN-Templates-Repo/CICDScripts/ in the command output
 devopsman:~/environment/CFN-Templates-Repo (master) $ aws s3 ls s3://hb-projectq/CICDScripts/
 
 ```
 
 12. We need **stack-pipeline.yml** file's URL address for using in AWS CloudFormation. You can get URL from Amazon S3 Console,the **stack-pipeline.yml** object's detail page or you can know from the S3 URL template
 
 ```
   https://<<BUCKET-NAME>>.s3-<<AWS REGION CODE>>.amazonaws.com/<<FOLDER NAME/<<FILE NAME>>
 ```
 So here is the URL
 
 ```
 https://hb-projectq.s3-us-west-2.amazonaws.com/EnvScripts/stack-pipeline.yml
 ```
 
 13. Open AWS CloudFormation Service Dashboard.
 
 ![Atlt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/5-CFN-Dashboard.png)
 
 
 14. It's time to create our project environments stack.
   
   * Step 1
     * Click _Create Stack / With New Resources (standard)_
     * Our template is ready, Template Source is Amazon S3 and the URL is above. Paste the URL and NEXT.
   * Step 2
     * StackName: **ProjectQEnvStack**
     * PipelineName: **PL-ProjectQ**
     * RepositoryName: **CFN-Templates-Repo**
     * Email: **halilb@amazon.com**
     * TemplateFileName: **EnvScripts/environment-stack-formation.yaml**  
     * TestStackName: **Test-ProjectQ**
     * TestStackConfig: **EnvScripts/test-stack-configuration.json**
     * ChangeSetName: **UpdatePreview-ProjectQ** 
     * ProdStackName: **Prod-ProjectQ**
     * ProdStackConfig: **EnvScripts/prod-stack-configuration.json**
   * Step 3
   * Step 4 
     * Check the **I acknowledge that AWS CloudFormation might create IAM resources.** in capabilities.
   
   Click _Create Stack_
 
    
