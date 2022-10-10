# DHC vRA - SNOW ITSM PoC

# Table of Contents

- [Changelog](#Changelog)
- [Introduction](#Introduction)
- [Purpose](#Purpose)
- [Audience](#Audience)
- [Scope](#Scope)


# Changelog
  
| Version | Date       | Description              | Author       |
| ------- | ---------- | ------------------------ | --------------- |
| 0.1     | 07/10/2022 | First version | Maikal Kumar |


# Introduction


# Purpose

This document describes the process and design for cretaion  and management of service requests in SNOW for VM reboot action in vRA 

# Audience

DHC Deployment Engineers, Dev-Sec-Ops team

# Scope

The PoC for vRA- SNOW service request management only covers reboot action in vRA which is  "Change virtual server power state" catalog in SNOW.


## Requirements

 Integration User : - This user will have access to request servicenow api from vra.
	
 Access to SNOW environemnt :- User should have access to Service now.
	
 Access to vRA Cloud :- User should have VRo access.
	

   
## SNOW Endpoint/ RestHost Configuration

  1. we need to configure RestApi so that api will be called by workflow. to cnfigure resthost follow below steps.

  2. Add a resthost by Navigating Library -> Workflow -> Library -> Http-Rest -> Configurtion -> Add a Resthost.
	
	![](SnowVraIntegration/snowimages/Add-resthost.png)
	
	3. Click on add Resthost.
	
	4. Configure the Resthost properties such as Host-Properties, Host- Authentication and proxy.	
	
	![](SnowVraIntegration/snowimages/Host-Properties.JPG)
	
	NOTE:  while adding Host URL,add only baseUrl. for eg: https://atosglobaldev.service-now.com/ is the base url.

	![](SnowVraIntegration/snowimages/Host-Authentication.JPG)
	
	![](SnowVraIntegration/snowimages/proxy-setting.JPG)	
		
	5. Click on Run.
		
	![](SnowVraIntegration/snowimages/Run.JPG)

   
        
## Create vRO workflow 

 * To create a workflow, We need to follow bewlow Steps.
  
 * This workflow will be called after an event will be trigger from VRA. 
	
 * Login to Respected Environment VRO.
	
 ![](SnowVraIntegration/snowimages/Vro-Login.JPG)   
     
 * To create workflow Navigate to Workflow, under workflow, select a folder or create a new folder for eg DHC then create a workflow under that folder using create New workflow.
    
![](SnowVraIntegration/snowimages/create-new-workflow.JPG)

* Give a name for workflow and click on create button.
    
![](SnowVraIntegration/snowimages/new-workflow2.JPG)

* Define variable which will be used in workflow for eg Rest-Host.

 ![](SnowVraIntegration/snowimages/new-workflow-interface.JPG)
    
* To define new variable, select variable tab and click on new button. After clicking on new button,new dialogue button will open.
    
* Give the varible name for eg "snowHost".
    
* Select the type of variable for eg: "RestHost".
    
![](SnowVraIntegration/snowimages/create-variable.JPG)

* Select the rest host that we have added using add restHost Properties.

* Click on create button.

* Navigate to Schema Section, drag a scriptable task object from left menu in the middle of start and end workflow object.

* Under general table go to input/output section and select defined variable as a input. to select this click on plus ***'+'*** button. and then click on select variable button then select that variable.

![](SnowVraIntegration/snowimages/select-reshostPcker.JPG)

* Go to under script section then write required code under this tab.

* To Create a RITM In Service Now, we have used below api.

### API call structure and details
  
 > ***URL:-*** https://atosglobaldev.service-now.com/api/x_aits2_appintinbo/order/create
 	
	> ***Request Method:-*** POST

	> Header: - 
		
        content Type: Application/json
        Authoraization: Basic
        
    > ***Request Body:-*** 
  
  		{  
        "transactionID":"S0760.001.01",
        "externalid":custDeploymentId,
        "organisation":"DHC Test1",
        "short_description":"VM Reboot",
        "CRM": "DHC-AUT-D",
        "orderDetail":{
        "vm_owner":"DHC Test001",
        "vm_site":"dhc.test001@dhctest001.com",
        "virtual_machine" :"dhctestnpapp01",
        "power_action":"Restart",
        "business_justification":"Test_BR01",
        "description":"VM Has rebooted for Some reason"
          }
        }

        
  ### Description of JSON Attribute
   
   | Attempt | Type    | Required |Description   |
   | :---:   | :---: | :---: | :---: |
   | Transaction Id | String   | Yes   | This is catalog mapping id, it varries based on catalog. for eg: S0760.001.01 is the mapping to restart virtul Machine.
   | ExternalId | String   | Yes   | This can be be anything in string.
   | Organisation | String   | Yes   | This is fixed "DHC Test1".
   | Short Description | String   | Yes | Short description for request.
   | CRM | String   |  Yes | This is fixed String "DHC-AUT-D".
   | VM Owner | String   | Yes   | This is a fixed String "DHC Test001".
   | VM Site | String   | Yes   | This is fix String "dhc.test001@dhctest001.com".
   | Virtual_Machine | String   | Yes   | This is virtual machine name on which action has performed.
   | Power_Action | String   | Yes   | This is action that will be performed on VM for eg: "Reboot", "Power Off".
   | Business_Justification | String   | Yes   | This can be anything meaningful which will state the justification of doing action.
   | Description | String   | Yes   | This can be used to define the detailed reason of performing action.


* Once code is written then select click on ***"RUN".***

![](SnowVraIntegration/snowimages/Run-work-final-workflow.png)
   
   
   
## Workflow configuration in VRA

   * To Configure workflow in VRA, create a Subscription and then attach in vro workflow in it.
	
   * Navigate to VRA ***Cloud Assembly*** -> ***Extensiblity*** -> ***Subscription*** -> ***New Subscription.***
   
![](SnowVraIntegration/snowimages/subscription-home-page.JPG)

 * Give the subscription name, select the event topic, select the workflow that we have created in VRO.
   
 * if we need to filter any event the we can add condtion as well. to add condition we can use metadata. for eg ***event.data.operation == "Reboot"*** will make this workflow execute only when we will reboot any VMs.
    
![](SnowVraIntegration/snowimages/Subscription-creation.JPG)

   * ***NOTE:*** If we make this subscription is blocking, While executing this workflow other workflow will not be executed until it finishes.
    
## Testing

   * To test the workflow we need to restart the any vm then we will see that event gets trigged and workflow executed succcessfully.
   
   ![](SnowVraIntegration/snowimages/restart-vm.png)
   
   * To see the worflow runs, navigate to Activity -> workflow Runs. then click on the workflow run id to see the execution details.
   
   ![](SnowVraIntegration/snowimages/subscription-workflow-runs-execution-details.JPG)
	
   * Also, we can verify in SNOW portal using RITM that we have got in the log of workflow. 
