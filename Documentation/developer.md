## APPLICATION DEVELOPMENT by DEVELOPER


Test environment is ready, CI/CD pipeline is also ready, waiting developers to develop the application. Developers' responsibilities are developing the application, coding unit tests, prepraring configuration files and also working with devops engineer to prepare the build and deployment scripts that will execute on the pipeline.

We will execute the pipeline with simple code, that is developed by developer.


## Let's do it together..

1. Login to AWS Web console as developer. I use the AWS Identity Access Management user named as _developer_ in this sample. Note: If we are role-playing both devopsman and developer. We need to use different browsers to stay logged in at same time.

2. Check that you are in AWS Region US-WEST-2 (Oregon)
![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/21-aws-dashboard-dev.png)

3. Switch to _AWS Cloudformation_ service dashboard. Select "ProjectQ-CICD" Stack, and click _Outputs_ tab. Copy the "CodeRepoCloneURLHttp" key, HTTP URL of the AWS CodeCommit repository, that is needed to clone the repository.

```
https://git-codecommit.us-west-2.amazonaws.com/v1/repos/ProjectQRepo
```

3. Switch to _AWS Cloud9_ service dashboard.
 
 ![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/2-Cloud9-dashboard.png)
 
4. Create an AWS Cloud9 environment for development related activities for developer.
  * Step 1
    * Name: **ProjectQ-Dev**
    * Desciption **Cloud9 for Project Q Developer**  
  * Step 2 (Choose all default values)
  * Step 3 (Review and Create Environment)
  
  AWS Cloud9 IDE Environment will be created in a couple of minutes
  
  ![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/22-Cloud9-IDE-Created-dev.png)
  
5. In AWS Cloud9, run the commands in _terminal_ view.
 
 ```console
 #Delete the README.md file, that comes with Cloud9 IDE
 developer:~/environment $ rm README.md 
 
 
 #Clone the AWS CodeCommit Repository to your AWS Cloud9 Environment
 #You will need Git credentials to connect to AWS CodeCommit repository. 
 #You can create your Git Credentials from AWS IAM - https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html
 developer:~/environment $ git clone https://git-codecommit.us-west-2.amazonaws.com/v1/repos/ProjectQRepo
 
 #Git configurations - email, username, caching credential etc.
 developer:~/environment $ git config --global user.email developerQ@amazon.com
 developer:~/environment $ git config --global user.name "developer"
 developer:~/environment $ cd ProjectQRepo/
 ddeveloper:~/environment/ProjectQRepo $ git config credential.helper cache --timeout=3000
 
 ```
 
 6. Our goal in this lab is to show how to automate CI/CD pipeline resources creation, not to show how to develop a web application. So instead of coding everything, we will use a sample application to show the how Ci/CD pipeline will work. Sample application archive file will be downloaded from Amazon S3.
 
 ```
 #Download archive.tar.gz a project 
 developer:~/environment/ProjectQRepo $ aws s3 sync  s3://hb-cfn-templates/projectzip/archive.tar.gz .
 
 #Unzip the archive file, you will see the folders and files
 developer:~/environment/ProjectQRepo (master) $ tar -xvfz archive.tar.gz
 
 #Delete the archive file. 
 developer:~/environment/ProjectQRepo (master) $ rm archive.tar.gz 
 
 ```
 
 7. You will see the **buildspec.yml** and **appspec.yml** files on the root of your source directory. **buildspec.yml** file will be used in the Build stage of the pipeline and **appspec.yml** will be used in the Deploy stage of the pipeline. You can see the details in ![buildspec](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html) and ![appspec](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html)
 
 
8. We developed our codes in our development environment (AWS Cloud9 IDE) and now it's time to add them in version control (AWS CodeCommit) that we alredy cloned the empty repository.

 ```
 #check the status of the files in your environment
 developer:~/environment/ProjectQRepo (master) $ git status
 
 #You will see list of files and folders that are not under source control yet. Add them to the git. 
 developer:~/environment/ProjectQRepo (master) $ git add .

 #Check the status of the files in your environment
 developer:~/environment/ProjectQRepo (master) $ git status
 
 # Commit the source codes to the git with comment
 developer:~/environment/ProjectQRepo (master) $ git commit -m "initial codes"
 
 #Send your committed changes to the shared repository your user/pass credentials are required.
 developer:~/environment/ProjectQRepo (master) $ git push origin master
 
 ```

9. Open AWS Codecommit service dashboard and click on the _ProjectQRepo_ on the list. You will see all the source codes that you pushed from the AWS Cloud9 IDE.

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/23-codecommit-files-cicd.png)

10. As you can remember, CI/CD pipeline is triggered by events that are generated by AWS CloudWatch. From that point of view, we committed new changes to the AWSCodeCommit and it must be trigger the AWS CodePipeline. Let's go to the AWS Codepipeline dashboard.

11. Open AWS Codepipeline service dashboard and the click the pipeline whose name starts with "ProjectQ".

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/24-Codepipeline%20list%20cicd.png)

12. Pipeline diagram shows the status of the each Stage and each stage's actions. Our pipeline will execute (if all steps are successful of course) till _DeployProd_ stages _ApproveToProd_ action. Because that action is "manual approval" waiting Product Owner (or DevOps Engineer) approval.

<img src="https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/25-waitingProd-cicd.png" width="450" height="500">

We deployed to the TEST environment but not PROD environment yet. Let's check if we really did.

13. We need URL address of the Application Load Balancer (ALB) both for TEST and PROD. We can find these information in Amazon EC2 console, but we also set that as output of the AWS Cloudformation templates. 

Open AWS CloudFormation dashboard. You will see all the executed templates.

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/26-CFN-Stacks.png)

14. Click on the _Test-ProjectQ_ stack on the list. In the new view, click on the "Outputs" tab. You will see the URL key's value. 

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/27-TestStack-ALB-URL.png)

15. Open the link in a new browser tab. You will see the deployed page..

16. Click on the _Prod-ProjectQ_ stack on the list. In the new view, click on the "Outputs" tab. You will see the URL key's value. 

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/28-ProdStack-ALB-URL.png)

17. Open the link in a new browser tab. You will still see "hello world"..

18. We saw that there is a bug in the page, we forgot to add the "Welcome to Re:invent 2019!"

19. Open AWS Cloud9 IDE and make the changes (below) to the _public/index.html_ file.

 ```
     
  <div class="text">
      <h1>Congratulations!</h1>
      <h1>Welcome to Re:Invent 2019!</h1>
      <h2>You just created a Node.js web application.</h2>
  </div>

 ```
 
 20. Save the index.html file than commit and push changes to AWS CodeCommit repository.
 
 ```
 #Check if you saved the index.html
 developer:~/environment/ProjectQRepo (master) $ git status
 
 #Commit the changes
 developer:~/environment/ProjectQRepo (master) $ git commit -a -m "bug fix- index.html updated"
 
 #Push the changes to AWS Codecommit Repository
 developer:~/environment/ProjectQRepo (master) $ git push origin master
 
 ```
 
 21. Open AWS Codepipeline and follow the execution of the pipeline stages. You can click on the _details_ link of each build and deploy actions, that will open AWS Codebuild or AWS Codedeploy execution logs. 
 
22. If _DeployTest_ stage is successful, refresh the test page. 

![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/29-Test-Page.png)

23. Back to AWS Codepipeline (ProjectQ-CICD...). Click the "Review" button on the _DeployProd_ stage's _ApproveToProd_ action. Click 'Approve' button. Pipeline execution will continue with _DeploytoProd_ action.

24. If _DeployProd_ stage is successful, refresh the prod page.


Congrats..You have successfully deployed the applications..
![alt text](https://github.com/halilbahadir/pipelinefactory/blob/master/Documentation/29-Test-Page.png)



