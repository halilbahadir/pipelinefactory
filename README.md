# Pipeline Factory
Empower Lean Teams with Centrally Built CI/CD Pipelines Using AWS CodePipeline &amp; AWS CloudFormation

In this workshop,  you build a pipeline product that your lean teams can use to bring their application code, tests, and infrastructure to be vetted, executed, approved, and eventually published with AWS CloudFormation.


## Scenario

There is a new demand from your business team. They want to have a new web site for the incoming marketing event in December. Demand is accepted and prioritized. Scrum team initiated and as part of Scrum team, devops engineer (devopsman) will responsible for provisioning the TEST and PROD environment according to the architecture design and also responsible for designing the CI / CD pipeline. 

As a DevOps engineer,  you are big fan of automation and using your existing templates, you will automate creating the AWS resources both in environment and also CI/CD pipeline.

Here are the requirements: 

1. Web Application will be running on AWS EC2 instances. We are expecting increasing traffic to the web application while reaching to the event date so it is important to scale according to the load. 

2.  There will be TEST and PROD environments mainly, but if we need another stage like UAT, it must be created quickly same configuration as PROD.

3. We have limited bugdet so cost is critical for us.

4. We will be using release pipeline using AWS Developer Tools. AWS CodeCommit for source control management, AWS CodeBuild for building (compile, unit test, etc.) the application, AWS CodeDeploy to deploy the application to the instances and AWS CodePipeline to design the release pipeline. 

5. We will be using continuous delivery but not continuous deployment yet, "Product Owner" has to approve the deployment for production environment.

## ARchitecture Design

The team designed the initial architecture as below. Considering the cost, security, performance, high-availability, scalability etc. 

