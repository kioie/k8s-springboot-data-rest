#  Deploy a Java Springboot microservice application on Google Cloud.

This tutorial is an attempt to run Ganesh Radhakrishnan ([@ganrad](https://github.com/ganrad)) Spring Purchase-order application that was designed to run off Azure, in a Google Cloud Environment.

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
      #    args: ["build", "-t", "gcr.io/$PROJECT_ID/k8s-springboot-data-rest", "--build-arg=JAR_FILE=target/po-rest-service-1.0.jar", "."]
      #images: ["gcr.io/$PROJECT_ID/k8s-springboot-data-rest"]
    ````

###How To

1. Initialize the Cloud SDK, create an App Engine application, and authorize the Cloud SDK to use GCP APIs in your local environment:

    ```
    gcloud init  
    gcloud app create  
    gcloud auth application-default login
    ```

# Set up Cloud SQL

1. Enable the [Cloud SQL API](https://console.cloud.google.com/flows/enableapi?apiid=sqladmin&_ga=2.97716831.1749283848.1589680102-1322801348.1576371208&_gac=1.250162036.1587192241.CjwKCAjwp-X0BRAFEiwAheRui4GkVAiJEcD-d_dhMaMnTeAmRAMMUBXLV45atuLUiiLinEjPGLLbuhoCzD8QAvD_BwE).  

2. Create a Cloud SQL (MySQL) instance and set the root user password following [these instructions](https://cloud.google.com/sql/docs/mysql/create-instance#create-2nd-gen).

    ````
    gcloud sql instances create test-instance --tier=db-n1-standard-1 --region=us-central1
    ````

3. Set the password for the `root@%` MySQL user

    ````
    gcloud sql users set-password root --host=% --instance test-instance --password [PASSWORD]
    ````

    **_Make sure you replace_** **`[PASSWORD]`** **_with your own password_**

4. Setup `orders` database.

    ````
    gcloud sql databases create orders --instance=test-instance
    ````

5. Get the `connectionName` of the instance in the format `project-id:zone-id:instance-id`:

    ````
    gcloud sql instances describe test-instance | grep connectionName
    ````

     Now you can test out your Cloud Sql Database and see if you are able to use this database. 
     

6. Update your `application-mysql.properties` file by replacing the instance-connection-name, database-name, username and password.  
  
    **_Note: The values you will find when you first open this file have been designed for a secret manager connection which I will discuss about shortly_**

    
# Set up Secret Manager

1.  Enable the [Secret Manager API](https://console.cloud.google.com/flows/enableapi?apiid=secretmanager.googleapis.com&redirect=https://console.cloud.google.com&_ga=2.72503123.1749283848.1589680102-1322801348.1576371208&_gac=1.225110888.1587192241.CjwKCAjwp-X0BRAFEiwAheRui4GkVAiJEcD-d_dhMaMnTeAmRAMMUBXLV45atuLUiiLinEjPGLLbuhoCzD8QAvD_BwE)

    **You will also need to grant the application access**  
    - Go to [IAM & Admin page](https://console.cloud.google.com/iam-admin/iam?_ga=2.101936833.1749283848.1589680102-1322801348.1576371208&_gac=1.224978664.1587192241.CjwKCAjwp-X0BRAFEiwAheRui4GkVAiJEcD-d_dhMaMnTeAmRAMMUBXLV45atuLUiiLinEjPGLLbuhoCzD8QAvD_BwE)  
    - Click the **`Project selector`** drop-down list at the top of the page.  
    - On the **`Select from`** dialog that appears, select the organization for which you want to enable Secret Manager.  
    - On the **`IAM`** page, next to the `app engine service account`, click **`Edit`**.  
    - On the **`Edit permissions`** panel that appears, add the necessary roles.  
    - Click **`Add another role`**. Select **`Secret Manager Admin`**.  
    - Click **`Save`**
  
2. Create new secrets for our datasource configuration file.
  
    ````
    echo -n “gcp-migration-project-278408:us-central1:test-instance” | gcloud secrets create spring_cloud_gcp_sql_instance_connection_name — replication-policy=”automatic” — data-file=-  
   
    echo -n “orders” | gcloud secrets create spring_cloud_gcp_sql_database_name — replication-policy=”automatic” — data-file=-  
   
    echo -n “root” | gcloud secrets create spring_datasource_username — replication-policy=”automatic” — data-file=-  
   
    echo -n “test123” | gcloud secrets create spring_datasource_password — replication-policy=”automatic” — data-file=-
    ````

      **_Note: Remember to use your own credentials here for_** **_`spring_cloud_gcp_sql_instance_connection_name`_** **_and_** **_`spring_datasource_password`_**
  
      Confirm your secrets have been created by running `gcloud secrets list` which should return a list of secrets.
      
      ![](https://github.com/kioie/k8s-springboot-data-rest/blob/master/images/secret_manager_list.png)

3. Replace the values in the `application-mysql.properties` file with the secrets url. For this step, you will need the fully-qualified name of the secret as defined on GCP.

    ````
    gcloud secrets describe spring_cloud_gcp_sql_instance_connection_name | grep name
   
    gcloud secrets describe spring_cloud_gcp_sql_database_name | grep name
    
    gcloud secrets describe spring_datasource_username | grep name
    
    gcloud secrets describe spring_datasource_password | grep name
    ````

    **_You will now use this names to create a secret manager url. The url will use the format `${sm://FULLY-QUALIFIED-NAME}` where `FULLY-QUALIFIED-NAME` is as retrieved above._**

4. Update `src/main/resources/application-mysql.properties:`

    ````
    #CLOUD-SQL-CONFIGURATIONS  
    spring.cloud.appId=gcp-migration-project-278408  
    spring.cloud.gcp.sql.instance-connection-name=${sm://projects/.../secrets/spring_cloud_gcp_sql_instance_connection_name}  
    spring.cloud.gcp.sql.database-name=${sm://projects/.../secrets/spring_cloud_gcp_sql_database_name}  
    ##SQL DB USERNAME/PASSWORD  
    spring.datasource.username=${sm://projects/.../secrets/spring_datasource_username}  
    spring.datasource.password=${sm://projects/.../secrets/spring_datasource_password}
    ````

# Set up GitHub repository with source files

1.  Create a [GitHub account](https://github.com/) if you don’t have one already.

2.  Enable the [Cloud Build API](https://console.cloud.google.com/flows/enableapi?apiid=cloudbuild.googleapis.com) in the target Cloud project.

3. Install the Google Cloud Build App on Github. You can follow this instructions [**here**](https://cloud.google.com/cloud-build/docs/automating-builds/run-builds-on-github#installing_the_google_cloud_build_app).

    **_I can recommend forking this repository, to make your integration easier, or you can create your own fresh repo and push your source files to initialize your repo. Either way, make sure you select your new repo as the repository you are connecting to, otherwise this build will not work._**
    
    ![](https://github.com/kioie/k8s-springboot-data-rest/blob/master/images/github_cloud_build.png)

5. In the Google Cloud Console, open the Cloud Build [Build triggers](https://console.cloud.google.com/cloud-build/triggers?_ga=2.173763299.1749283848.1589680102-1322801348.1576371208&_gac=1.259777016.1587192241.CjwKCAjwp-X0BRAFEiwAheRui4GkVAiJEcD-d_dhMaMnTeAmRAMMUBXLV45atuLUiiLinEjPGLLbuhoCzD8QAvD_BwE) page.

    **_Make sure to delete all triggers created, that may be related to this project before moving to the next step. We are doing this to make sure that no other builds are running other than the single build we have configured._**

6. Select your Google Cloud project and click **`Open`**.

7. Click **`Create Trigger`**.

8. Fill out the options

    -   _Required_. In the **`Name`** field, enter a name
    -   _Optional_. In the **`Description`** field, enter a brief description of how the trigger will work
    -   Under **`Event`**, select **`Push to a branch`**
    -   In the **`Source`** drop-down list, select `kioie/k8s-springboot-data-rest` repository  
    
      **_Note: If this repository does not appear, click the_** **_`Connect New Repository`_** **_button and connect your repo on GitHub, then return to step5._**
      
    -   Under **`Branch`**, enter `^master$`
    -   Under **`Build Configuration`** select **`Cloud Build configuration file (YAML or JSON)`**  
    For the `Cloud Build configuration file location` enter `cloudbuild.yaml`  
    **_Note: Do not add an extra_** **_`/`_**
    -   Click **`Create`**

Under your active triggers, you should now be able to see your newly created trigger.

   ![](https://github.com/kioie/k8s-springboot-data-rest/blob/master/images/cloud_build_triggers.png)

# Set up App Engine

1.  The `pom.xml` file already contains configuration for `projectId` and `version`. Change this to reflect the current project ID.  

    `<deploy.projectId>gcp-migration-project-278408</deploy.projectId>`
    
2.  Enable [App Engine Admin API](https://console.developers.google.com/apis/library/appengine.googleapis.com)

3.  Enable [App Engine Flexible API](https://console.developers.google.com/apis/library/appengineflex.googleapis.com)

4.  Give more permission to the cloud build service account

    **Grant cloudbuild service account, admin access to Secret Manager, App Engine and Cloud Sql**.

    -   Go to [IAM & Admin page](https://console.cloud.google.com/iam-admin/iam?_ga=2.101936833.1749283848.1589680102-1322801348.1576371208&_gac=1.224978664.1587192241.CjwKCAjwp-X0BRAFEiwAheRui4GkVAiJEcD-d_dhMaMnTeAmRAMMUBXLV45atuLUiiLinEjPGLLbuhoCzD8QAvD_BwE)
    
    -   Click the **`Project selector`** drop-down list at the top of the page and select the current project organization.
    
    -   On the **`IAM`** page, next to the `cloud build service account`, (not to be confused with the `cloud build service agent` )click **`Edit`** (or the pencil button).
    
    -   On the **`Edit permissions panel`** that appears, add the necessary roles.
    
    -   Click **`Add another role`** and add these three roles:  
        — App Engine Admin  
        — Cloud SQL Admin  
        — Secret Manager Admin
        
    -   Click **`Save`**.

      The final permission list should look something like this
      
      ![](https://github.com/kioie/k8s-springboot-data-rest/blob/master/images/final_permissions.png)
      
      The final step that triggers a cloud build will require pushing your updated code-base to GitHub.
    
# Push to GitHub and trigger a build

1.  Add your remote GitHub fork repo as your upstream repo

    ````
    git remote add upstream [https://github.com/<YOUR_ACCOUNT_NAME>/k8s-springboot-data-rest](https://github.com/YOUR_ACCOUNT_NAME/k8s-springboot-data-rest)  
    git remote -vv
    ````
  
2. Commit your changes

    ````
    git add .  
    git commit
    ````

3. Push upstream

    ````
    git push upstream master
    ````

4. This should automatically trigger your build on GCP cloud build. You can check the status of your build using the below command

    ````
    gcloud builds list
    ````                                                  
   
   ![](https://github.com/kioie/k8s-springboot-data-rest/blob/master/images/successful_build.png)
   
   A successful build!

5. You can fetch the url of the app with the command

    ````
    gcloud app browse
    ````

6. Now test out your endpoints

    ````
    curl [https://gcp-migration-project-278408.uc.r.appspot.com/orders](https://gcp-migration-project-278408.uc.r.appspot.com/orders)  
    
    curl [https://gcp-migration-project-278408.uc.r.appspot.com/orders/1](https://gcp-migration-project-278408.uc.r.appspot.com/orders/1)  
    
    curl [https://gcp-migration-project-278408.uc.r.appspot.com/orders/2](https://gcp-migration-project-278408.uc.r.appspot.com/orders/2)
    ````
