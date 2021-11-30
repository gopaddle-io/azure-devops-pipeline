# azure-devops-pipeline

we are going to integrate the azure pipelines with gopaddle. for that you need to create a azure pipelines first. while creating the pipelines follow these steps:

	1. select the github repository and branch
	2. select the starter pipeline config 
	3. grant the permission for branches to allow the access to the repo
	
once pipeline created successfully, check your repository to confirm the azure-pipelines.yml file is created. two parts are there to complete the pipeline. one is triggering the build another one is triggering the rolling update. 

for that you need an running cluster and application on the gopaddle. after you created the deploymentTemplates and complete the setup you have to launch the application. if your application is running, you have to set the azure CI-CD pipelines for future update. 

start editing the azure-pipelines.yml to set the pipeline,

	1. trigger: master 

		this is where you are mentioning which branch is going to be build while code pushed.


	2. pool:
           	vmImage: ubuntu-latest

		this is the environment where is the build is going to performed. in our case we are taking ubuntu 

	3. variables:
  		apiToken: <api_token>
  		containerID: <container_id>
  		serviceID: <service_id>
  		applicationID: <application_id> 
  		projectID: <project_id>
  		releaseID: <release_id>
  		distributionID: <distribution_id>
  		endPoint: <gopaddle_endpoint>

		get these details from the gopaddle and use it as a variables to create build and deploy.

	4. steps:
		there are two steps used one is for trigger build and another is create rolling update. use bash script and curl commands to call build and application APIs.

		Trigger Build:
			API:  https://$endPoint/gateway/v1/$(projectID)/build
			Method: POST
			Payload:
				{
				 "serviceID":"$(containerID)",
                                 "releaseID":"$(releaseID)",
                                 "distributionID":"$(distributionID)"
                                }

		to trigger the build use the curl command with this api.

		Get Status Build:
			API: https://$endPoint/gateway/v1/$(projectID)/build/$buildid
			Method: GET

		get the status of the build and if the status moved to created state then create the rolling update

		Rolling Update:
			API: https://$endPoint/gateway/v1/$(projectID)/application/$(applicationID)
			Method: PUT
			Payload: 
				{
				"serviceGroups":[
					{
					"services":[
						{
						"id":"$(containerID)",
						"serviceVersion":"draft",
						"releaseConfig":{
							"buildID":"'"$buildid"'",
							"version":"'"$buildVersion"'"
							}
						}
					],
					"id":"$(serviceID)",
					"name":"eshoponweb",
					"version":"draft",
					"description":"'"$commitMessage"'"
					}],
				"deploymentTemplateVersion":"draft",
				"updateType":"buildUpdate"
				}
			get the build id and version from the build and trigger the build using this payload.


				
		

