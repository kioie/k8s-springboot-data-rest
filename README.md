#  Deploy a Java Springboot microservice application on Google Cloud.

This tutorial is an attempt to run Ganesh Radhakrishnan ([@ganrad](https://github.com/ganrad)) Spring Purchase-order application that was designed to run off Azure, to a Google Cloud Environment.

As [@ganrad](https://github.com/ganrad) has done, we will equally attempt to leverage GCP features to simplify this deployment.

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

1. `Pom.xml`
   We need to update the `pom.xml` file to add `spring-cloud-gcp-starter`, `spring-cloud-gcp-starter-sql-mysql` and `spring-cloud-gcp-starter-secretmanager`
   
   https://github.com/kioie/k8s-springboot-data-rest/blob/4e5869b9782d8e2e3e292a22ca23cb348fc002a1/pom.xml#L65-L76
   
   We will also add the App engine plugin.
   
   https://github.com/kioie/k8s-springboot-data-rest/blob/4e5869b9782d8e2e3e292a22ca23cb348fc002a1/pom.xml#L89-L97
   
2. `application.properties`
   We update to include Cloud Sql configs
   https://github.com/kioie/k8s-springboot-data-rest/blob/ad0a94aaf6102a3256e02ff194bc4f761aeb84a1/src/main/resources/application.properties#L3-L12
   
3. `src/main/java/ocp/s2i/springboot/rest/config/PropertiesConfiguration.java`
    We'll comment out the `@Configuration` properties
    https://github.com/kioie/k8s-springboot-data-rest/blob/ad0a94aaf6102a3256e02ff194bc4f761aeb84a1/src/main/java/ocp/s2i/springboot/rest/config/PropertiesConfiguration.java#L12-L15
    
**Additional Files**

1. `src/main/appengine/app.yaml`
    This is a deployment config file for App engine.
    ````
        env: flex
        runtime: java
        runtime_config:
          jdk: openjdk8
        
        resources:
          cpu: 1
          memory_gb: 1
          disk_size_gb: 10
          volumes:
            - name: ramdisk1
              volume_type: tmpfs
              size_gb: 0.5
        automatic_scaling:
          min_num_instances: 1
          max_num_instances: 3
        
        handlers:
          - url: /.*
            script: this field is required, but ignored
   ````
2. `cloudbuild.yaml`
    This is a cloud build config file for executing the build pipeline.
    ````
    steps:
         - name: maven:3-jdk-8
           entrypoint: mvn
           args: ["test"]
         - name: maven:3-jdk-8
           entrypoint: mvn
           args: ["package", "-Dmaven.test.skip=true","appengine:deploy"]
      #  - name: gcr.io/cloud-builders/docker
      #    args: ["build", "-t", "gcr.io/$PROJECT_ID/inventorymanagement", "--build-arg=JAR_FILE=target/inventorymanagement-1.0.0.0.jar", "."]
      #images: ["gcr.io/$PROJECT_ID/inventorymanagement"]
    ````
 