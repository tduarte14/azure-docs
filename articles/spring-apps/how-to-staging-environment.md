---
title: Set up a staging environment in Azure Spring Apps | Microsoft Docs
description: Learn how to use blue-green deployment with Azure Spring Apps
author: karlerickson
ms.service: spring-apps
ms.topic: conceptual
ms.date: 01/14/2021
ms.author: karler
ms.custom: devx-track-java, devx-track-azurecli, event-tier1-build-2022
---

# Set up a staging environment in Azure Spring Apps

> [!NOTE]
> Azure Spring Apps is the new name for the Azure Spring Cloud service. Although the service has a new name, you'll see the old name in some places for a while as we work to update assets such as screenshots, videos, and diagrams.

**This article applies to:** ✔️ Java ❌ C#

**This article applies to:** ✔️ Basic/Standard tier ✔️ Enterprise tier

This article explains how to set up a staging deployment by using the blue-green deployment pattern in Azure Spring Apps. Blue-green deployment is an Azure DevOps continuous delivery pattern that relies on keeping an existing (blue) version live while a new (green) one is deployed. This article shows you how to put that staging deployment into production without changing the production deployment.

## Prerequisites

* Azure Spring Apps instance on a Standard pricing tier
* [Azure Spring Apps extension](/cli/azure/azure-cli-extensions-overview) for the Azure CLI

This article uses an application built from Spring Initializr. If you want to use a different application for this example, you'll need to make a simple change in a public-facing portion of the application to differentiate your staging deployment from production.

> [!TIP]
> [Azure Cloud Shell](https://shell.azure.com) is a free interactive shell that you can use to run the instructions in this article.  It has common, preinstalled Azure tools, including the latest versions of Git, JDK, Maven, and the Azure CLI. If you're signed in to your Azure subscription, start your Cloud Shell instance. To learn more, see [Overview of Azure Cloud Shell](../cloud-shell/overview.md).

To set up blue-green deployment in Azure Spring Apps, follow the instructions in the next sections.

## Install the Azure CLI extension

Install the Azure Spring Apps extension for the Azure CLI by using the following command:

```azurecli
az extension add --name spring
```

## Prepare the app and deployments

To build the application, follow these steps:

1. Generate the code for the sample app by using Spring Initializr with [this configuration](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.3.4.RELEASE&packaging=jar&jvmVersion=1.8&groupId=com.example&artifactId=hellospring&name=hellospring&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.hellospring&dependencies=web,cloud-eureka,actuator,cloud-starter-sleuth,cloud-starter-zipkin,cloud-config-client).

2. Download the code.
3. Add the following *HelloController.java* source file to the folder *\src\main\java\com\example\hellospring\*:

   ```java
   package com.example.hellospring;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.bind.annotation.RequestMapping;

   @RestController

   public class HelloController {

   @RequestMapping("/")

     public String index() {
         return "Greetings from Azure Spring Apps!";
     }

   }
   ```

4. Build the *.jar* file:

   ```azurecli
   mvn clean package -DskipTests
   ```

5. Create the app in your Azure Spring Apps instance:

   ```azurecli
   az spring app create -n demo -g <resourceGroup> -s <Azure Spring Apps instance> --assign-endpoint
   ```

6. Deploy the app to Azure Spring Apps:

   ```azurecli
   az spring app deploy -n demo -g <resourceGroup> -s <Azure Spring Apps instance> --jar-path target\hellospring-0.0.1-SNAPSHOT.jar
   ```

7. Modify the code for your staging deployment:

   ```java
   package com.example.hellospring;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.bind.annotation.RequestMapping;

   @RestController

   public class HelloController {

   @RequestMapping("/")

     public String index() {
         return "Greetings from Azure Spring Apps! THIS IS THE GREEN DEPLOYMENT";
     }

   }
   ```

8. Rebuild the *.jar* file:

   ```azurecli
   mvn clean package -DskipTests
   ```

9. Create the green deployment:

   ```azurecli
   az spring app deployment create -n green --app demo -g <resourceGroup> -s <Azure Spring Apps instance> --jar-path target\hellospring-0.0.1-SNAPSHOT.jar
   ```

## View apps and deployments

View deployed apps by using the following procedure:

1. Go to your Azure Spring Apps instance in the Azure portal.

1. From the left pane, open the **Apps** pane to view apps for your service instance.

   ![Screenshot of the open Apps pane.](media/spring-cloud-blue-green-staging/app-dashboard.png)

1. You can select an app and view details.

   ![Screenshot of details for an app.](media/spring-cloud-blue-green-staging/app-overview.png)

1. Open **Deployments** to see all deployments of the app. The grid shows both production and staging deployments.

   ![Screenshot that shows listed app deployments.](media/spring-cloud-blue-green-staging/deployments-dashboard.png)

1. Select the URL to open the currently deployed application.

   ![Screenshot that shows the U R L for the deployed application.](media/spring-cloud-blue-green-staging/running-blue-app.png)

1. Select **Production** in the **State** column to see the default app.

   ![Screenshot that shows the U R L for the default app.](media/spring-cloud-blue-green-staging/running-default-app.png)

1. Select **Staging** in the **State** column to see the staging app.

   ![Screenshot that shows the U R L for the staging app.](media/spring-cloud-blue-green-staging/running-staging-app.png)

>[!TIP]
> * Confirm that your test endpoint ends with a slash (/) to ensure that the CSS file is loaded correctly.
> * If your browser requires you to enter login credentials to view the page, use [URL decode](https://www.urldecoder.org/) to decode your test endpoint. URL decode returns a URL in the format *https://\<username>:\<password>@\<cluster-name>.test.azureapps.io/gateway/green*. Use this format to access your endpoint.

>[!NOTE]
> Configuration server settings apply to both your staging environment and your production environment. For example, if you set the context path (*server.servlet.context-path*) for your app gateway in the configuration server as *somepath*, the path to your green deployment changes to *https://\<username>:\<password>@\<cluster-name>.test.azureapps.io/gateway/green/somepath/...*.

If you visit your public-facing app gateway at this point, you should see the old page without your new change.

## Set the green deployment as the production environment

1. After you've verified your change in your staging environment, you can push it to production. On the **Apps** > **Deployments** page, select the application currently in **Production**.

1. Select the ellipsis after **Registration status** of the green deployment, and then select **Set as production**.

   ![Screenshot that shows selections for setting the staging build to production.](media/spring-cloud-blue-green-staging/set-staging-deployment.png)

1. Confirm that the URL of the app displays your changes.

   ![Screenshot that shows the U R L of the app now in production.](media/spring-cloud-blue-green-staging/new-production-deployment.png)

>[!NOTE]
> After you've set the green deployment as the production environment, the previous deployment becomes the staging deployment.

## Modify the staging deployment

If you're not satisfied with your change, you can modify your application code, build a new .jar package, and upload it to your green deployment by using the Azure CLI:

```azurecli
az spring app deploy  -g <resource-group-name> -s <service-instance-name> -n gateway -d green --jar-path gateway.jar
```

## Delete the staging deployment

To delete your staging deployment from the Azure portal, go to the page for your staging deployment and select the **Delete** button.

Alternatively, delete your staging deployment from the Azure CLI by running the following command:

```azurecli
az spring app deployment delete -n <staging-deployment-name> -g <resource-group-name> -s <service-instance-name> --app gateway
```

## Next steps

* [CI/CD for Azure Spring Apps](./how-to-cicd.md?pivots=programming-language-java)
