# 🛫 Hands-on tutorial:  Display and search SAP Flight Model data in a Microsoft Power App. 

## 👩‍💻 Introduction 
The scope of this hands-on tutorial is a detailed description about the creation of a low-code canvas app for displaying tabular data from an SAP NetWeaver system. 

To stay consistent with the recent demo-scenarios in this GitHub repository, the [SAP Flight Model](https://help.sap.com/doc/saphelp_nw70/7.0.31/en-US/cf/21f304446011d189700000e8322d00/content.htm?no_cache=true) sample application was selected. 
This tutorial is built on top of the existing blog of Sameer Chabungbam and is intended to give a more detailed step-by-step description for new users of Power Apps. 


It is recommended to read the blog before starting the implementation of this tutorial. 

[Introducing the SAP ERP connector | Microsoft Power Apps](https://powerapps.microsoft.com/en-us/blog/introducing-the-sap-erp-connector/)

The result of this hands-on tutorial will be a canvas app which displays flight data and provides the option to filter by an airline using a text field. 

The SAP data will be fetched by calling the SAP Business Application Programming Interface (BAPI) "BAPI_FLIGHT_GETLIST". 

A detailed description of the BAPI can be found here:
[BAPI_FLIGHT_GETLIST SAP ABAP Function Module - Find list of flights (se80.co.uk)](https://www.se80.co.uk/sapfms/b/bapi/bapi_flight_getlist.htm)

To implement the scenario three major steps have to be implemented: 
*  Preparation of demo environment  
*  Install and configure on-prem gateway 
*  Create instant cloud flow 
*  Create canvas app 

![Overview development tasks](https://github.com/ROBROICH/Display-and-search-SAP-Flight-Model-data-in-a-Microsoft-Power-App/blob/main/img/Intro_Creation_Tasks.png?raw=true)

From an architectural perspective four components are required for the implementing the scenario:

* SAP Netweaver system
* On-premise gateway
* Power Automate Flow 
* Power App canvas app 

![Overview architecture](https://github.com/ROBROICH/Display-and-search-SAP-Flight-Model-data-in-a-Microsoft-Power-App/blob/main/img/Intro_Architekture.png?raw=true)



## 👩‍💻 Prerequisites and demo environment setup 

Typically, productive customer environment applies policies and rules for the configuration of the Power App and on-premise data gateway. 

To limit the dependencies to productive environments, this example describes how to setup a dedicated demo environment, which includes the usage a dedicated SAP demo system as well. 

The availability of a demo SAP system is the prerequisite for this scenario. 
Typically the fastest and most convenient way to provision SAP demo systems is the usage of [SAP Cloud Appliance Library images](https://cal.sap.com/catalog#/solutions). 

These steps are required to configure the Power App demo environment: 
* Joint the M365 developer program and write down the administrator user. 
    * [Microsoft 365 dev center](https://developer.microsoft.com/en-us/microsoft-365)
* Create a Power Platform trail environment using the M365 administrator user 
    * [About trial environments - Power Platform | Microsoft Docs](https://docs.microsoft.com/en-us/power-platform/admin/trial-environments) 
    * The deployed trail environment should look as displayed here in the Power Platform Admin center: 
        ![Admin center](https://github.com/ROBROICH/Display-and-search-SAP-Flight-Model-data-in-a-Microsoft-Power-App/blob/main/img/Configuratin_AdminCenter.png?raw=true)

* Install an on premise data gateway on a virtual machine with network access to the SAP system (see prerequisite described here [Introducing the SAP ERP connector | Microsoft Power Apps](https://powerapps.microsoft.com/en-us/blog/introducing-the-sap-erp-connector/))
    * Please use the M365 developer program administrator user for registration of the gateway 
    * Choose the same Azure region as the Power App tenant is deployed at 
    * [What is an on-premises data gateway? | Microsoft Docs](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-onprem)
    * [Download and install SAP .NET Connector 3.0 SDK from SAP (SAP user required)](https://support.sap.com/en/product/connectors/msnet.html)
    
After successful installation and registration, the on-premise data gateway should be able to connect to the Power Apps and Power Automate services and be visible in the Power Apps portal: 
![on-premises data gateway 1](https://github.com/ROBROICH/Display-and-search-SAP-Flight-Model-data-in-a-Microsoft-Power-App/blob/main/img/Configuration_DataGateway.png?raw=true)
![on-premises data gateway 2](https://github.com/ROBROICH/Display-and-search-SAP-Flight-Model-data-in-a-Microsoft-Power-App/blob/main/img/Configuration_DataGateway_PowerApp.png?raw=true)

## Instant cloud flow implementation
The first implementation step is the creation of an instant cloud flow in the Power Apps portal. 
This instant cloud flow will create the connection to the SAP Netweaver system and fetch the data by calling the BAPI. 

* Select Flows -> Instant cloud flow 
![Instant Cloud Flow](https://github.com/ROBROICH/Display-and-search-SAP-Flight-Model-data-in-a-Microsoft-Power-App/blob/main/img/Flow_InstantCloudFlow.png?raw=true)

* As trigger select “Power Apps”  
![Select Power App](https://github.com/ROBROICH/Display-and-search-SAP-Flight-Model-data-in-a-Microsoft-Power-App/blob/main/img/Flow_InstantCloudFlow_SelectPoweApp.png?raw=true)

* Click on “New step”, search for “SAP” and select “Call SAP function”.   
![Select Call SAP function](https://github.com/ROBROICH/Display-and-search-SAP-Flight-Model-data-in-a-Microsoft-Power-App/blob/main/img/Flow_InstantCloudFlow_SelectSAPFunction.png?raw=true)

* The next step is to maintain the SAP connection parameters: 
    * Your AS Host
    * Client ID
    * System number 
    * The specific SAP function name which is __BAPI_FLIGHT_GETLIST__ for this demo scenario.

* To maintain the BAPI, the function specific parameters will be available for selection and maintenance. Please Click in “Airline” text field and select “Ask in PowerApps” to pass the Airline search parameter from the canvas app to the BAPI. 
* Limit the maximum rows to 100 by maintaining the corresponding field max_rows in the BAPIs parameters. 
![Maintain Call SAP function](https://github.com/ROBROICH/Display-and-search-SAP-Flight-Model-data-in-a-Microsoft-Power-App/blob/main/img/Flow_InstantCloudFlow_MaintainSAPFunction.png?raw=true)

* After maintaining the parameters, the next steps is to the test and execute the function by clicking on “Test” in the right top corner. The test is primarily required to collect the schema information of the BAPIs result set. 
![Test Call SAP function](https://github.com/ROBROICH/Display-and-search-SAP-Flight-Model-data-in-a-Microsoft-Power-App/blob/main/img/Flow_InstantCloudFlow_TestSAPFunction.png?raw=true)

* After successfully executing the flow and calling the BAPI, please copy and save the content of the text box “FLIGHT_LIST” as template for the following schema maintenance. 
![Test Call SAP function result](https://github.com/ROBROICH/Display-and-search-SAP-Flight-Model-data-in-a-Microsoft-Power-App/blob/main/img/Flow_InstantCloudFlow_TestSAPFunction_Result.png?raw=true)


🙏 Many thanks for your time and reading the document until here! 🙏












