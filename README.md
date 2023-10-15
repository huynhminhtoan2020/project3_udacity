# TechConf Registration Website

## Project Overview
The TechConf website allows attendees to register for an upcoming conference. Administrators can also view the list of attendees and notify all attendees via a personalized email message.

The application is currently working but the following pain points have triggered the need for migration to Azure:
 - The web application is not scalable to handle user load at peak
 - When the admin sends out notifications, it's currently taking a long time because it's looping through all attendees, resulting in some HTTP timeout exceptions
 - The current architecture is not cost-effective 

In this project, you are tasked to do the following:
- Migrate and deploy the pre-existing web app to an Azure App Service
- Migrate a PostgreSQL database backup to an Azure Postgres database instance
- Refactor the notification logic to an Azure Function via a service bus queue message

## Dependencies

You will need to install the following locally:
- [Postgres](https://www.postgresql.org/download/)
- [Visual Studio Code](https://code.visualstudio.com/download)
- [Azure Function tools V3](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#install-the-azure-functions-core-tools)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Tools for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack)

## Project Instructions

### Part 1: Create Azure Resources and Deploy Web App
1. Create a Resource group
2. Create an Azure Postgres Database single server
   - Add a new database `techconfdb`
   - Allow all IPs to connect to database server
   - Restore the database with the backup located in the data folder
3. Create a Service Bus resource with a `notificationqueue` that will be used to communicate between the web and the function
   - Open the web folder and update the following in the `config.py` file
      - `POSTGRES_URL`
      - `POSTGRES_USER`
      - `POSTGRES_PW`
      - `POSTGRES_DB`
      - `SERVICE_BUS_CONNECTION_STRING`
4. Create App Service plan
5. Create a storage account
6. Deploy the web app

### Part 2: Create and Publish Azure Function
1. Create an Azure Function in the `function` folder that is triggered by the service bus queue created in Part 1.

      **Note**: Skeleton code has been provided in the **README** file located in the `function` folder. You will need to copy/paste this code into the `__init.py__` file in the `function` folder.
      - The Azure Function should do the following:
         - Process the message which is the `notification_id`
         - Query the database using `psycopg2` library for the given notification to retrieve the subject and message
         - Query the database to retrieve a list of attendees (**email** and **first name**)
         - Loop through each attendee and send a personalized subject message
         - After the notification, update the notification status with the total number of attendees notified
2. Publish the Azure Function

### Part 3: Refactor `routes.py`
1. Refactor the post logic in `web/app/routes.py -> notification()` using servicebus `queue_client`:
   - The notification method on POST should save the notification object and queue the notification id for the function to pick it up
2. Re-deploy the web app to publish changes

## Monthly Cost Analysis
Complete a month cost analysis of each Azure resource to give an estimate total cost using the table below:

| Azure Resource | Service Tier | Monthly Cost |
| ------------ | ------------ | ------------ |
| *Azure Postgres Database* | Single Server| $77.672 |
| *Azure Service Bus* | Standard tier | $9.782|
  *Azure Functions* | Consumption Tier| $0.000016/GB-s |
| *Storage Accounts* | StorageV2 (general purpose v2) Premium| $0.15 per GB| 
| *App Service* | F1 | free |
|
Reference: https://azure.microsoft.com/en-us/pricing/#product-pricing

## Architecture Explanation

* Overview:

The TechConf website registration architecture includes:
Azure Web App to host the web application
Azure Function to process notifications via a service queue
Azure Postgres Database It allows you to run, manage, and scale PostgreSQL databases in the cloud without having to worry about the underlying infrastructure.
Service Bus, It is designed for building scalable, decoupled, and event-driven systems.
Azure Storage Accounts are containers for storing a wide variety of data types, including files, blobs, tables, queues, and disks.

* Deployment:
The web app is deployed on an Azure App Service, 
Azure App Service is a fully managed platform for building, deploying and scaling web apps and APIs. 
It is a platform-as-a-service (PaaS) from Microsoft Azure, 
designed to simplify hosting and managing web apps, mobile backends and RESTful APIs.

* Price:
The Azure resources chosen are cost-effective for the TechConf registration website
After reviewing the website https://azure.microsoft.com/en-us/pricing/#product-pricing, 
I found that the above selections are suitable for a hands-on project.

* Risks:
One potential risk is the hidden costs of the Azure Postgres Database, 
which could increase if the database grows significantly in both size and volume.

* In the Future:
The chosen architecture for the TechConf registration website, including Azure Web App and Azure Function along with 
other supporting Azure resources, provides a scalable, cost-effective and flexible solution for hosting and managing 
the registration process, notifications, and other website functions. 
It allows for easy deployment, cost optimization, and future scaling, while also minimizing potential risks.
