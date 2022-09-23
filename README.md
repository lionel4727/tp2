# Public Cloud - TP2

## Instructions
During this tutorial, we will implement the platform-as-a-service and Serverless concepts we have seen during the course. To do so, we will deploy the PetClinic application on Elastic Beanstalk in the first part of this lab, and on AWS Lambda in the second part of the lab.

A brief reminder of the application's architecture
![Architecture](https://pbs.twimg.com/media/CxsvUT8WIAAgUcA?format=jpg "Architecture")

The tutorial is divided into two parts: the following `README.md` file, and an `ANSWER.md` file which contains 5 questions to answer.

The principle of the tutorial is to answer the questions in the `ANSWER.md` file and push it to your repository. The rest is explained as the tutorial progresses.

## 1: Git
### 1.0: Fork the lab repo
Follow the link on the board to fork this repository into your personal GitHub Classroom repository. If you have access to the repository, it means that the manipulation worked well.

### 1.1 : Clone the repository
Clone the newly copied repo on your computer 
> From now on, **you will only work in your copy**. You don't have to come back to [the parent project](https://github.com/cours-hei/tp2).

## 2: Elastic Beanstalk
First, we will deploy the PetClinic project on Elastic Beanstalk.
As the virtual machines where you compiled the Petclinic project no longer exist, you will have to clone the repository of `Petclinic` on your local machine and recompile the application first.

> I remind you the URL of the `Petclinic` repository: `https://github.com/spring-projects/spring-petclinic/`, and the command to generate the `./mvnw package`.

### 2.1: Deploying Elastic Beanstalk
As seen during the course, to deploy an Elastic Beanstalk, go to the AWS portal and click on the Elastic Beanstalk menu (if you can't find the menu, it's also possible to use AWS search).
In the Elastic Beanstalk menu, create an application. This application must be of type :
- Platform: Java
- Platform branch : Java 8 running on 64bit Amazon Linux
- Platform version: 2.11.11 (Recommended)

> Remember to name your application `petclinic-<firstname>-<name>` so that the application can be created.

In order to deploy the PetClinic application, it is necessary to upload the compiled application, using the menu: Upload your code. It is therefore probably necessary to recompile the application. The compiled application will then be in the folder: `target`

**WARNING**: Create an issue called `2.1` with the content of the commands you have done

### 2.2: Verify that the application works
Once the application has been deployed, you now need to check that it works. Click on the exposed URL of the application.
You should then get a nginx proxy error. This is because by default Elastic Beanstalk expects the application to listen on port 5000. You will need to find a way to configure the proxy.

Some help can be found here: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-nginx.html

> ⚠️ **WARNING**: Create an issue called `2.2` with the commands you have made as the content

### 2.3: Connect to a database
Now that the PetClinic application is running, it is necessary to add a database. Luckily, Elastic Beanstalk makes it easy to add a database. In the Elastic Beanstalk configuration, add an RDS database by configuring it as follows:

```
- Engine: mysql
- Engine version: 8.0.23
- Instance class: db.t2.micro
- Storage : 5
- Username : petclinic
- Password : petclinic
- Retention : Delete
- Availability : Low (one AZ)
```

This command will create a MySQL RDS managed database. Once the database is created, AWS will return the endpoint of the RDS database.

Two new variables need to be created in Elastic Beanstalk in order to hot reconfigure the website.

The first variable is the `SPRING_PROFILES_ACTIVE` variable which indicates the profile to use when the application starts. If you remember from TP1, you will know the value to put in this variable.

The second variable is the `MYSQL_URL` variable which contains the database connection string. The value to put in this variable is the database endpoint in the form `jdbc:mysql://<endpoint-rds>/ebdb`

> ⚠️ **WARNING**: Create an issue called `2.3` with the contents of the commands you performed

### 2.4: Enable blue/green deployment
Once the PetClinic application has been modified to use the RDS database, we will modify the source code to make a visible change to the application. However, in order not to impact connected users, we will have to use the blue/green deployment technique.

Clone the current environment (the one with the RDS database) in order to create a pre-production environment resembling production in every way. This environment will be used as a test environment to test the changes made.

Modify the code to make a visible change. For example, modify the `welcome.html` file in the `src/main/resources/templates/welcome.html` folder.

Recompile the application and redeploy it to the new environment. Check with the new URL that the changes made are visible.

Once the changes made are satisfactory, switch the production URL to the new version.

> ⚠️ **WARNING**: Create an issue called `2.4` with the commands you made as content

## 3: AWS S3 and AWS Lambda
We will now deploy the application using AWS S3 and AWS Lambda

### 3.1: Deploying the website
We will use a new version of the web application developed in Angular. This new version has the advantage of being able to be compiled as a static site and thus to be deployed on a static site hosting, like AWS S3.

Clone the following new repo: `https://github.com/spring-petclinic/spring-petclinic-angular`

Follow the installation and build documentation for the application. At the end of the build, you should have a `dist` folder containing the compiled website.

> Create an issue called `3.1` containing the commands you have made

### 3.2: Creating the S3 bucket
The site will now need to be hosted on a public S3 bucket.

Go to the S3 menu and click on the **Create bucket** button. In the new menu, choose a unique DNS name, then in the permissions, uncheck the **Block *all* public access** box, then check the *I acknowledge that the current settings may result in this bucket and the objects within becoming public* box

Once the bucket has been created, upload **all** of the contents of the *dist* folder (including subfolders) into the S3 bucket. Then select all the files and sub-folders in the bucket, click on the **Actions** button and select **Make public**.

Then you need to activate the AWS S3 static hosting solution. To do this, go to the Properties tab, click on the Static Web Hosting box and activate the *Use this bucket to host a website* option. In the field, **Index document**, indicate the file **index.html**.

Finally click on **Save**.

You should be able to access the website via a URL of the form: `http://<bucket-name>.s3-website-<region>.amazonaws.com`

The site should not work perfectly, as the API calls are now missing. In the next part of the tutorial, we will deploy the application on AWS Lambda.

> Create an issue called `3.2` with the content of the commands you have made

### 3.3: Deploying the application on AWS Lambda
We will now deploy the PetClinic application on AWS Lambda. In order to use PetClinic on AWS Lambda, we will use a new project exposing REST APIs.

Clone the repo: `https://github.com/spring-petclinic/spring-petclinic-rest`. Before being able to deploy the application on AWS Lambda, it is necessary to make some modifications so that AWS Lambda accepts the application. All explanations are available here: https://github.com/awslabs/aws-serverless-java-container/wiki/Quick-start---Spring-Boot2#manual-setup--converting-existing-projects

During step **3. Packaging the application**, it is also necessary to remove the following part of the `pom.xml`.

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <!-- Spring Boot Actuator displays build-related information
                if a META-INF/build-info.properties file is present -->
            <goals>
                <goal>build-info</goal>
            </goals>
            <configuration>
                <additionalProperties>
                    <encoding.source>${project.build.sourceEncoding}</encoding.source>
                    <encoding.reporting>${project.reporting.outputEncoding}</encoding.reporting>
                    <java.source>${maven.compiler.source}</java.source>
                    <java.target>${maven.compiler.target}</java.target>
                </additionalProperties>
            </configuration>
        </execution>
    </executions>
</plugin>
```

You also need to change the version of the `maven-shade-plugin`. Change the line `<version>2.3</version>` to version `3.2.4`.

Once you have done this, you need to recompile the application and deploy the `.jar` in the `target` folder to a new AWS S3 Bucket. This new bucket will be used to deploy the application on AWS Lambda.

> ⚠️ **WARNING**: Create an issue called `3.3` with the contents of the commands you made

### 3.4: AWS Lambda and API Gateway
We will now create an AWS Lambda to retrieve various information from the PetClinic application. Go to the AWS Lambda menu, and click on the Create function button. Give your function a name in the form `petclinic-rest-<firstname>-<lastname>`, choose the runtime **Java 8 on Amazon Linux 2** and click Create function.

On the new page, in the **Function code** section, select **Actions** and **Upload a file from Amazon S3**. Then copy/paste the path to the S3 bucket containing the PetClinic binary.

Once the S3 bucket is configured, we now need to change the handler that will respond to our requests. Go to the Basic Settings section further down the page. Change the handler field to your handler (in my case `org.springframework.samples.petclinic.StreamLambdaHandler::handleRequest`). Also change the timeout to 2 minutes and the memory allocation to 1536 MB.

Once these changes are made, return to the top of the page and click on Add trigger. On the new page, select **API Gateway**, then **Create a new API**, then **HTTP API** and finally **Open** from the **Security** menu. Click on **Add**. Your Lambda will then be exposed via the URL `https://<random>.execute-api.<region>.amazonaws.com/default/<lambda-name>`.

**WARNING**: Create an issue called `3.4` with the commands you have made as its content

### 3.5: Changing the API URL
Now that the Lambda is exposed to the internet, it is necessary to change the API call URL in the website, and redeploy the website.

Go to the `src/environment.ts` file in the `spring-petclinic-angular` project and change the **REST_API_URL** to the new Gateway API URL (of the form `https://<random>.execute-api.<region>.amazonaws.com/default/<lambda-name>`)

Then recompile the `spring-petclinic-angular` application and redeploy it to the S3 bucket.

The new PetClinic page is now fully operational.

> Create an issue called `3.5` with the contents of the commands you made
