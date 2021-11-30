![](https://img.shields.io/github/downloads/gopaddle-io/azure-devops-pipeline/total) ![](https://img.shields.io/github/issues/gopaddle-io/azure-devops-pipeline) ![](https://img.shields.io/twitter/follow/gopaddleio?style=flat-square)

# Sample pipeline template for Azure DevOps

Sample pipeline template to be used with Azure DevOps to trigger an automated container build and a rolling update for applications managed by gopaddle. 

 ## Pre-requisite

As a pre-requisite, an application must be deployed in gopaddle. Below flow chart gives the step by step process to be followed before creating a Azure DevOps pipeline.

![](https://gopaddle-marketing.s3.ap-southeast-2.amazonaws.com/azure-devops-pipeline-prerequisite.png)

Since we are building a pipeline for an application deployed in gopaddle, we must first initialize and deploy an application in gopaddle before we move on to creating the pipeline in Azure DevOps.

+ Subscribe to gopaddle - If you do not have a gopaddle subscription yet, subscribe to the [gopaddle portal](https://portal.gopaddle.io/signUp)
+ [Provision K8s in gopaddle](https://help.gopaddle.io/en/articles/3942973-registering-a-cloud-account)
+ [Add a Container Registry](https://help.gopaddle.io/en/articles/3942974-adding-a-docker-registry) - Add a Container registry to gopaddle, to push or pull Docker images
+ Clone the project locally - Clone the GitHub project to be containerized. 
+ Initialize and deploy the project using gopaddle
    + [Download and install gpctl](https://help.gopaddle.io/en/articles/5116592-installing-and-configuring-gopaddle-command-line-utility) - Now, from your local desktop, download and install gpctl command line utility.
	+ [Perform gpctl init](https://help.gopaddle.io/en/articles/5056807-initializing-a-microservice-from-scratch) - Auto-generate the Dockerfile and Kubernetes YAML, build docker images, and deploy the application.
	+ capture the .gp file with the resource IDs - Once the application is onboarded using gopaddle, gpctl init creates a .gp file in the project folder which contains the ***apiToken***, ***containerID***, ***serviceID***, ***applicationID***, ***projectID***, ***releaseID*** and the ***distributionID***. Make a note of these IDs, as we will be using these in the Azure DevOps pipeline script.

 

## Getting started

Create a pipeline in Azure DevOps.

  1. Select the GitHub repository and branch used while initializing the gopaddle application
  
  ![](https://gopaddle-marketing.s3.ap-southeast-2.amazonaws.com/azure-github-configure.png)
  
  2. Select the starter pipeline config
  
  ![](https://gopaddle-marketing.s3.ap-southeast-2.amazonaws.com/azure-starter-pipeline.png)
  
  3. Grant the permission for branches to allow the access to the GitHub reposiory
  4. Once the pipeline is created, Azure DevOps creates a ***azure-pipelines.yml*** file in the project root folder in GitHub repository. Replace this default template with the contents from the ***azure-pipelines.yml*** from this sample template.
  5. Edit the contents of the ***azure-pipelines.yml*** file and replace the IDs with the IDs gathered during the gpctl init process.

 ## Pipeline Script explained
 
- trigger: master
Branch to be monitored for code commits. Any change that is committed to this branch will trigger the Azure DevOps pipeline.

- pool
```
vmImage: ubuntu-latest
```
Build environment where the container build has to be triggered.

- variables
The set of variables used by the pipeline. The values of these variables are derived from the .gp file generated during the gptcl init process.

| Variable  | Purpose |
| ------------- | ------------- |
| apiToken  | API token to access the gopaddle account  |
| containerID  | Container ID of the project onboarded in gopaddle  |
| serviceID  | Service ID under which the container is added  |
| applicationID  |  ID of the application launched in gopaddle  |
| projectID  | Project ID under which the resources are onboarded. If no project ID is specified during the gpctl init process, the ***default*** project will be assumed. |
| releaseID  | Release ID under which the container is onboarded. If no release ID is specified during the gpctl init process, the ***default*** release will be assumed.  |
| distributionID  | Distribution ID under which the container image is built. If no distribution ID is specified during the gpctl init process, the ***default*** distribution will be assumed.  |
| endPoint  | gopaddle access URL. If no endPoint is specified during the gpctl init process, the endpoint ***portal.gopaddle.io*** will be assumed by default. In case of gopaddle Enterprise Edition, this endpoint can configured to point to the on-premise gopaddle access URL  |

- steps

Defines the different stages of the DevOps pipeline. The template has two steps:

  + **Step-1:** Leverages gopaddle APIs to initiate the Container build within gopaddle as soon as the code is committed. Below is the API request that is used to initiate a build in gopaddle.

  **API**:  https://$endPoint/gateway/v1/$(projectID)/build
  
  **Method**: POST
  
  **Payload**: 
  
  ```json
{
   "serviceID":"$(containerID)",
   "releaseID":"$(releaseID)",
   "distributionID":"$(distributionID)"
}
```

  Once the build is triggered, the pipeline script polls for the build status and waits until the build completes. Below is the API call to get the build information based on build ID.
  
  **API**: https://$endPoint/gateway/v1/$(projectID)/build/$buildid
  
  **Method**: GET
 
  ![](https://gopaddle-marketing.s3.ap-southeast-2.amazonaws.com/azure-pipelinerun.png)
 
  + **Step-2 :** Leverages gopaddle APIs to update the application managed by gopaddle. Below is the API request that is used to initiate a build in gopaddle.
  
  **API**: https://$endPoint/gateway/v1/$(projectID)/application/$(applicationID)
  
  **Method**: PUT
  
  **Payload**: 
  
  ```json
	  {
   "serviceGroups":[
      {
         "services":[
            {
               "id":"$(containerID)",
               "serviceVersion":"draft",
               "releaseConfig":{
                  "buildID":"'""$buildid""'",
                  "version":"'""$buildVersion""'"
               }
            }
         ],
         "id":"$(serviceID)",
         "name":"eshoponweb",
         "version":"draft",
         "description":"'""$commitMessage""'"
      }
   ],
   "deploymentTemplateVersion":"draft",
   "updateType":"buildUpdate"
}
```

## Sequence of pipeline steps

![](https://gopaddle-marketing.s3.ap-southeast-2.amazonaws.com/azure-devops-pipeline-sequence.png)

## Maintainers
This sample template is maintained by the <img src="https://i0.wp.com/blog.gopaddle.io/wp-content/uploads/2020/08/cropped-gopaddle.png?fit=512%2C512&ssl=1" width="17" height="17"> [gopaddle.io](https://gopaddle.io) team.
