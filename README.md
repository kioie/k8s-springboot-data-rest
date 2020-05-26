#  Deploy a Java Springboot microservice application on Google Cloud.

This tutorial is an attempt to run Ganesh Radhakrishnan (@ganrad) Spring Purchase-order application that was designed to run off Azure, to a Google Cloud Environment.

As @ganrad has done, we will equally attempt to leverage GCP features to simplify this deployment.

Our goal here is to:

1.  Define a **Build Pipeline** in **Google Cloud Build**.  Execute the build pipeline to create Springboot Java Microservice Application jar (**po-service 1.0**) and push it to App Engine.  This task focuses on the **Continuous Integration** aspect of the DevOps process.
2.  Leverage on **Cloud Sql** as a replacement of the mysql service container.
3.  Leverage on **Secret Manager** to manage application secrets.

This Springboot application demonstrates how to build and deploy a *Purchase Order* microservice (`po-service`) as an application on GCP App Engine Service on GCP. The deployed microservice supports all CRUD operations on purchase orders.

**Prerequisites:**
1. Create a project in the [Google Cloud Platform Console](https://console.cloud.google.com/).  
2. Enable billing for your project.  
3. Install the [Google Cloud SDK](https://github.com/GoogleCloudPlatform/community/blob/master/sdk).  
4. Install [Maven](https://maven.apache.org/install.html) 
5. Create a [GitHub account](https://github.com/)

We have forked this project from https://github.com/ganrad/k8s-springboot-data-rest.git

**Adjustments**

1. ***Pom.xml***
   We need to update the `pom.xml` file to add `spring-cloud-gcp-starter`, `spring-cloud-gcp-starter-sql-mysql` and `spring-cloud-gcp-starter-secretmanager`
   
   https://github.com/kioie/k8s-springboot-data-rest/blob/4e5869b9782d8e2e3e292a22ca23cb348fc002a1/pom.xml#L65-L76
   
   We will also add the App engine plugin.
   
   https://github.com/kioie/k8s-springboot-data-rest/blob/4e5869b9782d8e2e3e292a22ca23cb348fc002a1/pom.xml#L89-L97
   
    