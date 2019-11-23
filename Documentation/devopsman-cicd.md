## CI/CD PIPELINE GENERATOR by DEVOPS ENGINEER


As a DevOps Engineer, your responsibilities also include managing CI/CD pipeline and services in that pipeline. These pipeline services are source control with AWS CodeCommit, build the code (compile, unit test, static code analysis, security code analysis etc.) with AWS CodeBuild, deploy to different environments (Test, Prod etc.) with AWS CodeDeploy, automating the test. 

Designing and configuring all these services in a pipeline with AWS CodePipeline is not adding value to the business, so it is better to automate the pipeline generation as well.

We will prepare an AWS CloudFormation Template, that will create all resources required for the CI/CD pipeline. In this AWS CloudFormation template, we will use NESTED templates that will help us to reuse the sub-templates, and design easier if any change required. CI/CD pipeline will be as the diagra below.

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/20-Pipeline-diagram-merged.png)


##Let's do it together

1. Open Amazon S3 Dashboard and find the object "master-formation.yaml" URL under hb-projectq bucket, CICDScripts folder or you can know from the S3 URL template
 
 ```
   https://<<BUCKET-NAME>>.s3-<<AWS REGION CODE>>.amazonaws.com/<<FOLDER NAME/<<FILE NAME>>
 ```
 So here is the URL

```
https://hb-projectq.s3-us-west-2.amazonaws.com/CICDScripts/master-formation.yaml

```

2. Open AWS CloudFormation Dashboard

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/13-CFN-Dashboard-CICD.png)

3. Create our CI/CD Pipeline stack. Click _Create Stack / With New Resources (standard)_

  * Step 1
    * Our template is ready, Template Source is Amazon S3 and the URL is above. Paste the URL and NEXT.
  * Step 2
    * StackName: **ProjectQ-CICD**
    * RepositoryName: **ProjectQRepo**
    * RepositoryDescription: **ProjectQRepo for CodeCommit Repository**
    * BranchName: **master**
    * TriggerName: **masterTrigger**
    * BucketName: **hb-projectq**
    * ApplicationName: **ProjectQ**
    * DeploymentStackName: **Test-ProjectQ**
    * EmailAddress: **halilb@amazon.com**
  * Step 3
  * Step 4 
    * Check the **I acknowledge that AWS CloudFormation might create IAM resources.** in capabilities.
    * Check the **I acknowledge that AWS CloudFormation might require the following capability: CAPABILITY_AUTO_EXPAND** in capabilities.
    
      
   Click _Create Stack_
   
   ![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/14-CFN-cicd-master.png)
 
 4. Because the _master-formation.yaml_ template is designed in nested way. Each stack will call another AWS Cloudformation template
 
  * CodeCommitStack (codecommit-formation.yaml)
  * CodeBuildStack (codebuild-formation.yaml)
  * CodeDeployStack (codedeploy-formation.yaml)
  * CodePipelineStack (codepipeline-formation.yaml)
  
  and each nested template will execute one by one sequentially because they are dependent to each other.
  
  * CodeCommitStack (codecommit-formation.yaml) 
  
   ![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/15-CFN-cicd-commit.png)
 
  * CodeBuildStack (codebuild-formation.yaml)
  
   ![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/16-CFN-cicd-build.png)
   
  * CodeDeployStack (codedeploy-formation.yaml)
 
   ![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/17-CFN-cicd-deploy.png)
  
  * CodePipelineStack (codepipeline-formation.yaml)
 
   ![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/18-CFN-cicd-pipeline.png)
   
   Wait until **ProjectQ-CICD** stack's status is "CREATE_COMPLETE". That means all nested stack's statuses are "CREATE_COMPLETE".
   
5. Open AWS Codepipeline Dashboard and click the pipeline thats name start with "ProjectQ".

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/19-codepipeline-cicd.png)

6. You will see the recently created pipeline stacks and its actions in a diagram. 





IT's [DEVELOPER](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/developer.md) TIME..

