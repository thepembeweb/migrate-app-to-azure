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

| Azure Resource   | Service Tier | Monthly Cost |
| ---------------- | ------------ | -------------- |
| Azure PostgreSQL |   Basic      |   $43.55       |
| Azure Service Bus|   Basic      |   $0.01        |
| Azure App Service|   Basic (B1) |   $21.12       |
| Azure Storage    |   Basic      |   $0.10        |

## Architecture Explanation

### Azure Web App
Using App Service was ideal for this app since the main criteria was cost. With the basic tier being $13.14 per instance for 730 hours, this means we can have multiple instances to get high availability and scalability for less than the price of the single VM (standard tier). A VM would have been a more expensive option. Going the Azure Web App route was also great because this app is relatively less complex, and is not expected to handle a vast increase in the number of users, with separate, dedicated servers which would have been suitable for a VM. Also, because the app is built with Python, using App Services is a great choice since Python is fully supported.

### Azure Function
Azure Functions were a great choice for this app in terms of cost as well as having full language support for Python. In addition, Azure Functions can run directly in an App Service and scale with the App Service. Also, by using Consumption Plan hosting, we only pay for the computing resources based on the number of executions, time of execution, and memory used. Using the Consumption Plan also ensures that the Azure Function instances are automatically allocated to handle the scale of the requests. Azure Functions fit nicely in this application in that when a user creates a notification, a new message is queued, and then a Service Bus Queue trigger activates the Azure function to send emails to attendees as well as update the notification status. This solution prevents HTTP timeout exceptions which would have occurred in long running tasks as a result of looping through all attendees. 

## Built With

* [Azure](https://portal.azure.com/) - Cloud service provider used
* [PostgreSQL](https://www.postgresql.org//) - Database used
* [Python](https://www.python.org/) - The programming language used

## Authors

* **[Pemberai Sweto](https://github.com/thepembeweb)** - *Initial work* - [TechConf Registration Website](https://github.com/thepembeweb/migrate-app-to-azure)

## License

[![License](http://img.shields.io/:license-mit-green.svg?style=flat-square)](http://badges.mit-license.org)

- This project is licensed under the MIT License - see the [LICENSE](LICENSE.md) file for details
- Copyright 2021 © [Pemberai Sweto](https://github.com/thepembeweb).

