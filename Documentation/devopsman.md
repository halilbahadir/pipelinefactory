## DEVOPS ENGINEER


As a devops engineeer, you are responsible for creating and managing the TEST / PROD Stages resources in an automated way. To do so you will be designing a pipeline using AWS CodePipeline services. As you can see below, 

<img src="https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/8-Merged-Test-Prod.png" width="450" height="500">
 
 
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
 _StackPipeline.yml_ will create the pipeline, using AWS CodePipeline Service. As you can see in figure below, 
 
 <img src="https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/8-Merged-Test-Prod.png" width="450" height="500">
 
 There are 3 stages in pipeline
 
  * _StackSource_ is the first stage. This stack has 1 source action (_TemplateSource_) provided by AWS CodeCommit that listens the events from AWS CodeCommit and pipeline is triggered by the each new commits. 
  * _TestStack_ is the second stage. This stack has 2 actions (_CreateStack_ and _ApproveTestStack_). _CreateStack_ action is deploy action, provided by AWS CloudFormation. We configure the action by setting the AWS CloudFormation Template file (_environment-stack-formation.yaml_) and configuration file (_test-stack-configuration.json_) as input. _ApproveTestStack_ is a manual approval action, waits manual approve/refect input to continue or stop the pipeline execution. 
  * _ProdStack_ is the third stage. This stack has 3 actions (_CreateChangeSet_, _ApproveChangeSet_ and _ExecuteChangeSet_). _CreateChangeSet_  action is deploy action, provided by AWS CloudFormation. When you need to update a stack, understanding how your changes will affect running resources before you implement them can help you update stacks with confidence. AWS CloudFormation makes the changes to your stack only when you decide to execute the change set, allowing you to decide whether to proceed with your proposed changes or explore other changes by creating another change set. Actually we are not changing anything in Prod environment initially, but for future changes, it is important to knw what will change in PROD environment, before executing. _ApproveChangeSet_ is a manual approval action, waits manual approve/refect input to continue or stop the pipeline execution. This step is needed to get approval for the PROD changes. _ExecuteChangeSet_ action is deploy action, provided by AWS CloudFormation. This action executes the change sets that are defined in _CreateChangeSet_ action.
 
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
 
    
15. It will quickly create all AWS CodePipeline resource on AWS. In AWS CloudFormation Dashboard, check if _ProjectQEnvStack_ status is **CREATE_COMPLETE**.  

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/11-OnlyProjectStackPipeline.png)


16. Open AWS CodePipeline and click on the _PL-ProjectQ_ pipeline in the list. You will see the pipeline stacks and its actions in a diagram. 

17. After the pipeline is created, the pipeline will start execution starting with the first stage (_StackSource_). After the first stage is successfull than second stage (_TestStage_) will start and as first action of the stack is deploying a AWS CloudFormation template. 

18. Go back to the AWS CloudFormation Dashboard and you will see another stack execution is started (CREATE_IN_PROGRESS). This stack is the one that triggered by the AWS CodePipeline's _CreateStack_ action.

19. It will take some time to create all TEST environment resources on AWS. In AWS CloudFormation Dashboard, check if _Test-ProjectQ_ status is **CREATE_COMPLETE**. 

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/10-TestStack.png)

20. As you can see above in the figure, the outputs tab has the URL key, that is the Application Load Balancer URL. Open link in a new browser tab. While creating the Amazon EC2 instance, we added (by user data) a basic page, to see if the instance is running. This is the page that you see in browser saying "Hello World!"

21.  Open AWS CodePipeline and click on the _PL-ProjectQ_ pipeline in the list. You will see the pipeline stacks and _ApproveTestStack_ action is waiting your manual approval to continue to execute the pipeline. 

22. Click on "Approve" button to approve the TEST environment is successfully created.

23. Pipeline will start the _ProdStage_ stage, with _CreateChangeSet_ action. Click on the "Details" link in the action frame.

24. It will open a browser tab for list of the changes. Because there is no resource in the PROD stage so all AWS resources  are tagged as **ADD**.

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/12-prodChangeSets.png)

AWS CloudFormation did not create any AWS Resources for the PROD stage yet, it only showed the  resources changes if you execute.


Congrats...You have successfully created the TEST environment.

Let's stop in here..Because we don't need the PROD environment now.

IT's [DEVELOPER](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/developer.md) TIME..

