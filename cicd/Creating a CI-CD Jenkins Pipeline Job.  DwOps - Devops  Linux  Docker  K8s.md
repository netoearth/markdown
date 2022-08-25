Creating a CI-CD Jenkins Pipeline Job.

Here we will learn how to create a demo Continuous Integration (CI) and Continuous Delivery (CD) Jenkins Pipeline job. We also implement manual steps for the deployment to a staging environment. The procedure will only give you an idea of how CI-CD works.

###### **Prerequisites:**

**✔  Maven:** Install Maven on your Jenkins server.

**✔  SSH Publisher Plugin:** Install ‘SSH Publisher’ or ‘Publish over SSH’ plugin in Jenkins.

**✔  Staging server:** a remote server on which we will deploy the artifact. In this, the AWS EC2 instance is being used.

**✔  Java Web Application:** Get the demo project from **[here](https://github.com/dwops-git/springDemo).**

#### **Step 0. Install and Configure the SSH Publisher Plugin**

**✔**  Install the plugin by going to **Manage Jenkins > Manage** Plugin > Available Tab > Search SSH publisher > Install the plugin.

**✔**  Configure the plugin by going to Manage Jenkins > Configure System > Find Publish over SSH.

![SSH-Publisher-plugin](https://dwops.com/wp-content/uploads/2021/05/Selection_073.png)

**✔**  Fill the required by fields as follows:

    **✔**  Passphrase: If has any. (In this case, we don’t have any)

    **✔**  Key: Paste the .pem file or private key of the staging server.    

    **✔  SSH Servers:** Click ‘Add’        

        **✔  Name:** Give an appropriate name        

        **✔  Hostname:** IP address of the staging server.        

        **✔  Username:** username used to log in to the staging server.        

        **✔  Remote Directory:** Directory path where you want to deploy the artifact.

**✔**  After that test the configuration by clicking the Test button.

![ssh-publisher-add-server](https://dwops.com/wp-content/uploads/2021/05/image_2021_05_23T10_24_23_990Z.png)

#### **Step 1. Create a Pipeline Job.**

**✔**  Click on **New Items**.

**✔**  Give the project a suitable name and choose the Pipeline project type.

#### **Step 2. Configure the CI Part.**

**✔**  Go to the Build Triggers section.

**✔**  Select Poll SCM and schedule to check for any changes for every minute using the following cron expression: \* \* \* \* \*

**✔**  Under Pipeline write the basic syntax of the pipeline such as:

```
Pipeline {
     agent any
     stages{
        stage(‘’){
        }
     }
 }
```

**✔**  Open the **Pipeline Syntax** page in a new tab to generate pipeline syntax.

###### **Stage 1: ‘Git Clone’**

**✔**  Write your first stage to clone the code from GitHub by generating the syntax.

###### **Stage 2: ‘Clean Package’**

**✔**  We will be using **Maven** installed on the Jenkins server for the next 2 stages.

**✔**  In the **steps** section, write the maven command:

```
stage('Clean Package'){
    steps{
        sh ‘mvn clean package’
    }
}
```

**✔  Clean** will clean the target directory and the **package** will build and packages the project.

###### **Stage 3: ‘Test the code’**

**✔**  Next, we will use maven to perform unit tests on the project.

**✔**  In the **steps** section, write the below command: sh ‘mvn test’

```
stage('Test the Code'){
    steps{
        sh ‘mvn test’
    }
}
```

#### **Step 3. Adding the CD Part in Pipeline**

Now, we will add the last stages of the pipeline.

###### **Stage 4. ‘Manual Approval’**

**✔**  This stage will be used to get the manual approval from another Jenkins User. This will prevent the automatic deployment of artifacts (.war file) on the staging server.

**✔**  For simplicity, we will add a basic ‘Proceed or Abort’ option here.

**✔**  You can get the code from the Pipeline Syntax page by choosing the ‘input’ option.

```
input ‘Deploying to stage server’
```

###### **Stage 5. ‘Deploy’**

**✔**  In this stage, we will deploy the artifact (.war file) to the staging server using the SSH Publisher plugin.

![ssh-publisher-plugin](https://dwops.com/wp-content/uploads/2021/05/image_2021_05_23T07_08_54_495Z.png)

**✔**  Just select ‘sshPublisher’ in the dropdown of Sample Step and fill the required fields:

    **✔**  Name: Select the name of the server configured in Step 0.

    **✔**  Source files: give the path of the artifact using a pattern. For eg. \*\*/\*.war

    **✔**  Remove Prefix: If need to remove some folder from the path.

    **✔**  Remote directory: This is the same field configured in

**Step 0**. So, we can leave this blank.    

**✔  Exec Command:** To execute any command on the server.

**✔**  Next, Generate the Script and paste it into the **steps** section of this **stage**.

![ssh-publisher](https://dwops.com/wp-content/uploads/2021/05/Selection_076-1024x573.png)

**✔**  Save the configuration.

**Note:** You can see the Code and the build of the code on the Jenkins server at /var/lib/jenkins/<name\_of\_job>. This will help you to know the directory structure of the job and how to provide the path of the .war file in the SSH Publisher plugin configuration.

![jenkins-project-path](https://dwops.com/wp-content/uploads/2021/05/image_2021_05_23T07_30_30_303Z.png)