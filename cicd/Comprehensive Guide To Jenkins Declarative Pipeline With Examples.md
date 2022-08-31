Jenkins Pipeline is an automation solution that lets you create simple or complex (template) pipelines via the DSL used in each pipeline. Jenkins provides two ways of developing a pipeline- Scripted and Declarative. Traditionally, Jenkins jobs were created using Jenkins UI called FreeStyle jobs. In Jenkins 2.0, Jenkins introduced a new way to create jobs using the technique called pipeline as code. In **pipeline as code** technique, jobs are created using a script file that contains the steps to be executed by the job. In Jenkins, that scripted file is called **Jenkinsfile**. In this [Jenkins tutorial](https://www.lambdatest.com/learning-hub/jenkins), we will deep dive into Jenkins Declarative Pipeline with the help of Jenkins declarative pipeline examples.

Let’s get started with the basics.

## What is Jenkinsfile?

**Jenkinsfile** is just a text file, usually checked in along with the project’s source code in Git repo. Ideally, every application will have its own Jenkinsfile.

Jenkinsfile can be written in two aspects – **Scripted pipeline syntax & Declarative pipeline syntax**

## What is Jenkins Scripted Pipeline?

Jenkins pipelines are traditionally written as scripted pipelines. Ideally, the scripted pipeline is stored in Jenkins webUI as a Jenkins file. The end-to-end scripted pipeline script is written in Groovy.

-   It requires knowledge of Groovy programming as a prerequisite.
-   Jenkinsfile starts with the word **node**.
-   Can contain standard programming constructs like if-else block, try-catch block, etc.

**Sample Scripted Pipeline**

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>node</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Stage 1'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>'hello'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

## What is Jenkins Declarative Pipeline?

The Declarative Pipeline subsystem in Jenkins Pipeline is relatively new, and provides a simplified, opinionated syntax on top of the Pipeline subsystems.

-   The latest addition in Jenkins pipeline job creation technique.
-   Jenkins declarative pipeline needs to use the predefined constructs to create pipelines. Hence, it is not flexible as a scripted pipeline.
-   Jenkinsfile starts with the word **pipeline**.

Jenkins declarative pipeline should be the preferred way to create a Jenkins job as they offer a rich set of features, come with less learning curve & no prerequisite to learn a programming language like Groovy just for the sake of writing pipeline code.

We can also validate the syntax of the Declarative pipeline code before running the job. It helps to avoid a lot of runtime issues with the build script.

### Our First Declarative Pipeline

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Welcome Step'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>'Welcome to LambdaTest'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

We recommend VS Code IDE for writing Jenkins pipeline scripts, especially when creating Jenkins Declarative pipeline examples.

## Running Your First Declarative Pipeline

Now that you are well-acquainted with the Jenkins pipeline’s basics, it’s time to dive deeper. In this section, we will learn how to run a Jenkins declarative pipeline. You can refer to our [Jenkins Pipeline Tutorial](https://www.lambdatest.com/blog/jenkins-pipeline-tutorial/) for more.

Let us run our Jenkins declarative pipeline step by step.

**Step 1:** Open Jenkins home page (`http://localhost:8080` in local) & click on **New Item** from the left side menu.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image3-1.png)

**Step 2:** Enter **Jenkins job name** & choose the style as **Pipeline** & click **OK**.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image5-1.png)

**Step 3:** Scroll down to the **Pipeline** section & copy-paste your first Declarative style Pipeline code from below to the script textbox.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image4-1.png)

**Step 4:** Click on the Save button & Click on **Build Now** from the left side menu.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image2-1.png)

We can see the build running stage by stage in **Stage** View.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image1-1.png)

**Step 5:** To check logs from Build Job, click on any stage & click on the **check logs** button. Or you can use the **Console Output** from the left side menu to see the logs for the build.

### Console Output

![Console Output](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Console-Output-1024x383.png)  
[![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/CTA-1.png)](https://accounts.lambdatest.com/register?utm_source=Blog&utm_medium=LambdaTest%20App&utm_campaign=blogCTA&utm_term=CTA)

## Jenkins Declarative Pipeline Syntax

In this section, we will look at the most commonly used Jenkins declarative pipeline examples or syntax. Typically, declarative pipelines contain one or more declarative steps or directives, as explained below.

**pipeline**

Entire Declarative pipeline script should be written inside the **pipeline** block. It’s a **mandatory** block.

**agent**  
Specify where the Jenkins build job should run. agent can be at pipeline level or stage level. It’s mandatory to define an agent.

**Possible values-**

1.  **any** – Run Job or Stage on any available agent.
2.  **none** – Don’t allocate any agent globally for the pipeline. Every stage should specify their own agent to run.
3.  **label** – Run the job in agent which matches the label given here. Remember [Jenkins CI/CD](https://www.lambdatest.com/blog/what-is-jenkins/) can work on Master/Agent architecture. Master nodes can delegate the jobs to run in Agent nodes. Nodes on creation given a name & label to identify them later.
    
    E.g All linux nodes can be labeled as linux-machine
    

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>agent</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>label</span><span> </span><span>'linux-machine'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

5.  **docker** – Run the job in given Docker container
    
    A sample can be found here at [GitHub](https://github.com/iamvickyav/jenkins-declarative-pipeline-notes/blob/master/declarative-syntax-samples.md#agent).
    

## stages

**stages** block constitutes different executable stage blocks. At least one stage block is mandatory inside stages block.

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>agent</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>label</span><span> </span><span>'linux-machine'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

## stage

stage block contains the actual execution steps. Stage block has to be defined within stages block. It’s mandatory to have at least one stage block inside the stage block. Also its mandatory to name each stage block & this name will be shown in the **Stage View** after we run the job.

In below example, the stage is named as “**build step**”

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>agent</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>label</span><span> </span><span>'linux-machine'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stage</span><span>(</span><span>'build step'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

A sample can be found here at [GitHub](https://github.com/iamvickyav/jenkins-declarative-pipeline-notes/blob/master/declarative-syntax-samples.md#stage).

## steps

**steps** block contains the actual build step. It’s mandatory to have at least one step block inside a stage block.

Depending on the Agent’s operating system (where Jenkins job runs), we can use shell, bat, etc., inside the steps command.

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stage</span><span>(</span><span>'build step'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>echo</span><span> </span><span>"Build stage is running"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

Scripted Pipeline in Jenkins job

Sample can be found here at [GitHub](https://github.com/iamvickyav/jenkins-declarative-pipeline-notes/blob/master/declarative-syntax-samples.md#steps).

## parameters

The parameters directive provides a way for Jenkins job to interact with Jenkins CI/CD users during the running of the build job.

parameter can be of the following types – **string, text, booleanParam, choice** & **password**

**string** – Accepts a value of String type from Jenkins user.

E.g.  
**string(name: ‘NAME’, description: ‘Please tell me your name?’)**

**text** – Accepts multi line value from Jenkins user  
E.g.  
**text(name: ‘DESC’, description: ‘Describe about the job details’)**

**booleanParam** – Accepts a true/false value from Jenkins user  
E.g.  
**booleanParam(name: ‘SKIP\_TEST’, description: ‘Want to skip running Test cases?’)**

**choice** – Jenkins user can choose one among the choices of value provided  
E.g.  
**choice(name: ‘BRANCH’, choices: \[‘Master’, ‘Dev’\], description: ‘Choose the branch’)**

**password** – Accepts a secret like password from Jenkins user  
E.g.  
**password(name: ‘SONAR\_SERVER\_PWD’, description: ‘Enter SONAR password’)**

Let’s look at a sample on how to use parameters directive-

<table><tbody><tr><td data-settings="show"><div><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p></div></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>parameters</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>string</span><span>(</span><span>name</span><span>:</span><span> </span><span>'NAME'</span><span>,</span><span> </span><span>description</span><span>:</span><span> </span><span>'Please tell me your name?'</span><span>)</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>text</span><span>(</span><span>name</span><span>:</span><span> </span><span>'DESC'</span><span>,</span><span> </span><span>description</span><span>:</span><span> </span><span>'Describe about the job details'</span><span>)</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>booleanParam</span><span>(</span><span>name</span><span>:</span><span> </span><span>'SKIP_TEST'</span><span>,</span><span> </span><span>description</span><span>:</span><span> </span><span>'Want to skip running Test cases?'</span><span>)</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>choice</span><span>(</span><span>name</span><span>:</span><span> </span><span>'BRANCH'</span><span>,</span><span> </span><span>choices</span><span>:</span><span> </span><span>[</span><span>'Master'</span><span>,</span><span> </span><span>'Dev'</span><span>]</span><span>,</span><span> </span><span>description</span><span>:</span><span> </span><span>'Choose branch'</span><span>)</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>password</span><span>(</span><span>name</span><span>:</span><span> </span><span>'SONAR_SERVER_PWD'</span><span>,</span><span> </span><span>description</span><span>:</span><span> </span><span>'Enter SONAR password'</span><span>)</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Printing Parameters'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"Hello ${params.NAME}"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"Job Details: ${params.DESC}"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"Skip Running Test case ?: ${params.SKIP_TEST}"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"Branch Choice: ${params.BRANCH}"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"SONAR Password: ${params.SONAR_SERVER_PWD}"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-9.06.10-AM.png)

If a parameters directive is used in the pipeline, Jenkins CI/CD can sense that it needs to accept the user’s input while running the job. Hence Jenkins will change the **Build now** link in the left side menu to **Build with parameters** link.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-01-31-at-2.13.34-PM.png)

When we click on the **Build with Parameters** link, Jenkins CI/CD will let us pass values for the parameters we configured in the declarative pipeline.

![Build with Parameters](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Build-with-Parameters-1024x465.png)

Once we enter each parameter’s values, we can hit the **build** button to run the job.

## script

script block helps us run **Groovy** code inside the Jenkins declarative pipeline.

<table><tbody><tr><td data-settings="show"><div><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p></div></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>parameters</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>string</span><span>(</span><span>name</span><span>:</span><span> </span><span>'NAME'</span><span>,</span><span> </span><span>description</span><span>:</span><span> </span><span>'Please tell me your name'</span><span>)</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>choice</span><span>(</span><span>name</span><span>:</span><span> </span><span>'GENDER'</span><span>,</span><span> </span><span>choices</span><span>:</span><span> </span><span>[</span><span>'Male'</span><span>,</span><span> </span><span>'Female'</span><span>]</span><span>,</span><span> </span><span>description</span><span>:</span><span> </span><span>'Choose Gender'</span><span>)</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Printing name'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>script</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>def </span><span>name</span><span> </span><span>=</span><span> </span><span>"${params.NAME}"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>def </span><span>gender</span><span> </span><span>=</span><span> </span><span>"${params.GENDER}"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>if</span><span>(</span><span>gender</span><span> </span><span>==</span><span> </span><span>"Male"</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"Mr. $name"</span><span>&nbsp;&nbsp;&nbsp;&nbsp;</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span><span> </span><span>else</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"Mrs. $name"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp; </span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-9.07.04-AM.png)

**script** block is wrapped inside the **steps** block. In the above example, we are printing the name passed as parameter suffixed with Mr. or Mrs. based on the Gender chosen.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-01-31-at-2.31.06-PM-1024x239.png)

We used **build with parameters** to pass params at the time of running the job.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-01-31-at-2.31.22-PM.png)

## environment

Key value pairs which helps passing values to job during job runtime from outside of Jenkinsfile. It’s one way of externalizing configuration.

**Example-** Usually, [Jenkins jobs](https://www.lambdatest.com/blog/jenkins-freestyle-project/) will run in a separate server. We may not be sure where the installation path of JAVA or JMeter is that server. Hence these are ideal candidates to be configured in Environment variables & passed during job run.

Read – [How To Set Jenkins Pipeline Environment Variables?](https://www.lambdatest.com/blog/set-jenkins-pipeline-environment-variables-list/)

### Method 1 : Configure Environment Variable in Jenkins CI/CD portal

**Step 1:** Open Jenkins Server URL (`http://localhost:8080`)

**Step 2:** Click on Manage Jenkins from the left sidebar menu.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-9.08.04-AM.png)

**Step 3:** Click on **Configure System** under System Configuration.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-9.08.25-AM-1024x272.png)

**Step 4:** Scroll down to the **Global Properties** section. This is where we will add our Environment variables.

**Step 5:** Click on the **Add** button under Environment Variables & enter the Key & value.

We have added the Java installation path under the variable name **JAVA\_INSTALLATION\_PATH**

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/unnamed.png)

**Step 6:** Click on **Save** Button.

**Refer Environment variable in Jenkinsfile**  
We can refer to the Environment variables in declarative pipeline using the `${} syntax`

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Initialization'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"${JAVA_INSTALLATION_PATH}"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp; </span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-9.10.17-AM.png)

### Method 2 : Creating & Referring Environment Variable in Jenkinsfile

We can create key value pairs of environment variables under **environment** block. It’s an **optional** block under pipeline.

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>environment</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>DEPLOY_TO</span><span> </span><span>=</span><span> </span><span>'production'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Initialization'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"${DEPLOY_TO}"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span></span><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.00.02-AM.png)

### Method 3 : Initialize Environment variables using sh scripts in Jenkinsfile

Let’s say we need the timestamp when the job gets run for logging purposes. We can create an Environment variable which can hold the timestamp. But how to initialize it ?

We can use the shell script to fetch the current timestamp and assign it to the Environment variable.

**Example-**

The following command will print date & time in shell

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>&gt;</span><span>&nbsp;&nbsp;</span><span>date</span><span> </span><span>'+%A %W %Y %X'</span><span></span></p><p><span>Tuesday</span><span> </span><span>03</span><span> </span><span>2021</span><span> </span><span>22</span><span>:</span><span>03</span><span>:</span><span>31</span></p></div></td></tr></tbody></table>

Lets see how to use the above command to initialize the Environment variable in Jenkinsfile.

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Initialization'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>environment</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>JOB_TIME</span><span> </span><span>=</span><span> </span><span>sh</span><span> </span><span>(</span><span>returnStdout</span><span>:</span><span> </span><span>true</span><span>,</span><span> </span><span>script</span><span>:</span><span> </span><span>"date '+%A %W %Y %X'"</span><span>)</span><span>.</span><span>trim</span><span>(</span><span>)</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>sh</span><span> </span><span>'echo $JOB_TIME'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.00.26-AM.png)

**returnStdout:** true makes sh step returning the output of the command so you can assign it to a variable.

Sample can be found here at [GitHub](https://github.com/iamvickyav/jenkins-declarative-pipeline-notes/blob/master/declarative-syntax-samples.md#environment-variables).

![Automation Chat Request](https://www.lambdatest.com/blog/wp-content/uploads/2020/04/Automation-Trial-2.jpg)

### Load credentials via environment block

credentials() is a special method that can be used inside the environment block. This method helps loading the credentials defined at Jenkins configuration.

Lets see it with an example. First lets configure credential in Jenkins CI/CD-

**Creating Credential in Jenkins portal**

**Step 1:** Open Jenkins Server URL (`http://localhost:8080`)

**Step 2:** Click on **Manage Jenkins** from the left sidebar menu.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.01.34-AM.png)

**Step 3:** Click on **Manage** Credentials under **Security**.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.01.59-AM-1024x263.png)

**Step 4:** Click on the **Global** hyperlink.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image6.png)

**Step 5:** Click on Add **Credentials** from the left side menu.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.02.32-AM.png)

For demonstration, let me use the following values

-   Kind – Username with password
-   Scope – Global
-   Username – admin
-   Password – root123
-   ID – MY\_SECRET
-   Description – Secret to access server files

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image9-1-1024x396.png)

**Step 6:** Click the **OK** button. Now our credentials are configured.

**Referring credential in Jenkinsfile**

We can use the special method called **credentials()** to load credentials we configured. We need to use the ID we used while configuring credentials to load a particular credential.

**Example-** credentials(‘MY\_SECRET’)

To refer to the username, append the word **\_USR** to the variable we assigned credentials() output & append the word **\_PSW** to get the password.

Example- Lets load the credential at **MY\_CRED** variable using the **credentials(‘MY\_SECRET’)** method. We need to append **\_USR ($MY\_CRED\_USR)** & **\_PSW ($MY\_CRED\_PSW)** to MY\_CRED variable to get the username & password.

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;</span><span>environment</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp; </span><span>MY_CRED</span><span> </span><span>=</span><span> </span><span>credentials</span><span>(</span><span>'MY_SECRET'</span><span>)</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Load Credentials'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"Username is $MY_CRED_USR"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"Password is $MY_CRED_PSW"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.03.09-AM.png)

Interestingly, Jenkins print username & password as \*\*\*\* to avoid accidental leakage of credentials.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image13.png)

## when

Acts like if condition to decide whether to run the particular stage or not . Its an **optional** block.

1.  **when** block

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stage</span><span>(</span><span>'build'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>when</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>branch</span><span> </span><span>'dev'</span><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>echo</span><span> </span><span>"Working on dev branch"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.03.58-AM.png)

4.  when block using **Groovy expression**

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stage</span><span>(</span><span>'build'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>when</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>expression</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>return</span><span> </span><span>env</span><span>.</span><span>BRANCH_NAME</span><span> </span><span>==</span><span> </span><span>'dev'</span><span>;</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>echo</span><span> </span><span>"Working on dev branch"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.04.39-AM.png)

7.  when block with **environment variables**

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>environment</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>DEPLOY_TO</span><span> </span><span>=</span><span> </span><span>'production'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Welcome Step'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>when</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>environment </span><span>name</span><span>:</span><span> </span><span>'DEPLOY_TO'</span><span>,</span><span> </span><span>value</span><span>:</span><span> </span><span>'production'</span></p><p><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>'Welcome to LambdaTest'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.04.57-AM.png)

10.  when block with **multiple conditions & all conditions should be satisfied**

<table><tbody><tr><td data-settings="show"><div><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p></div></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>environment</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>DEPLOY_TO</span><span> </span><span>=</span><span> </span><span>'production'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Welcome Step'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>when</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>allOf</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>branch</span><span> </span><span>'master'</span><span>;</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>environment </span><span>name</span><span>:</span><span> </span><span>'DEPLOY_TO'</span><span>,</span><span> </span><span>value</span><span>:</span><span> </span><span>'production'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>'Welcome to LambdaTest'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.05.27-AM.png)

13.  when block with **multiple conditions & any of the given conditions should be satisfied**

<table><tbody><tr><td data-settings="show"><div><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p></div></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>environment</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>DEPLOY_TO</span><span> </span><span>=</span><span> </span><span>'production'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Welcome Step'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>when</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>anyOf</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>branch</span><span> </span><span>'master'</span><span>;</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>branch</span><span> </span><span>'staging'</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>'Welcome to LambdaTest'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.05.50-AM.png)

Sample can be found here at [GitHub](https://github.com/iamvickyav/jenkins-declarative-pipeline-notes/blob/master/declarative-syntax-samples.md#when).

## tools

This block lets us add pre configured tools like Maven or Java to our job’s PATH. It’s an **optional** block.

To use any tool, they have to be pre-configured under the **Global Tools Configuration** section. For this example, let’s see how to configure Maven. You can also refer to our [Jenkins Maven integration tutorial](https://www.lambdatest.com/blog/selenium-maven-jenkins-integration/) for more information.

**Step 1:** Open Jenkins Server URL (`http://localhost:8080`)

**Step 2:** Click on **Manage Jenkins** from the left sidebar menu.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.01.34-AM-1.png)

**Step 3**: Click on **Global Tools Configuration** under System Configuration.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.06.40-AM-1024x256.png)

**Step 4:** Scroll down to the **Maven** section & click on **Maven Installations**.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.07.12-AM.png)

**Step 5:** Click on the **Add Maven** button. Enter the following values in the configuration.

-   Name – Maven 3.5.0
-   MAVEN\_HOME – Enter the maven installation path in local

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image10-1024x276.png)

Alternatively we can download Maven from Internet instead of pointing to the local installation path by enabling the Install Automatically checkbox.

**Step 6:** Click on **Add Installer** button & choose **Install from Apache**.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image12-1024x348.png)

**Step 7:** Choose the Maven version to download. We have chosen the Maven version **3.6.3**.

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image11-1024x314.png)

**Step 8:** Click on the **Save** button.

Refer to Maven tool in Pipeline in tools block with the name we used while configuring in Global Tools Configuration (MAVEN\_PATH in this example).

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>tools</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>maven</span><span> </span><span>'MAVEN_PATH'</span><span></span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stage</span><span>(</span><span>'Load Tools'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>sh</span><span> </span><span>"mvn -version"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.08.09-AM.png)

Sample can be found here at [GitHub](https://github.com/iamvickyav/jenkins-declarative-pipeline-notes/blob/master/declarative-syntax-samples.md#tools).

**Console Output-**

Run the job & we should see the mvn -version command response in console output-

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/image14.png)

Similarly we can also configure & use tools like Java, Gradle, Git etc from tools block in pipeline code.

## parallel

parallel blocks allow us to **run multiple stage blocks concurrently**. It’s ideal to parallelize stages which can run independently. We can define agents for each stage within a parallel block, so each stage will run on its own agent.

**Example-** When we want to run a piece of script in windows agent & another script in linux agent as part of the build, we can make them run concurrently using parallel blocks.

<table><tbody><tr><td data-settings="show"><div><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p></div></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stage</span><span>(</span><span>'Parallel Stage'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>parallel</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'windows script'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>label</span><span> </span><span>"windows"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>echo</span><span> </span><span>"Running in windows agent"</span></p><p><span></span><span>bat</span><span> </span><span>'echo %PATH%'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'linux script'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>label</span><span> </span><span>"linux"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>sh</span><span> </span><span>"Running in Linux agent"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

Sample can be found here at [GitHub](https://github.com/iamvickyav/jenkins-declarative-pipeline-notes/blob/master/declarative-syntax-samples.md#parallel).

## post

**post** block contains additional actions to be performed on completion of the pipeline execution. It can contain one or many of the following conditional blocks – always, changed, aborted, failure, success, unstable etc.

**post block conditions**

**always –** run this post block irrespective of the pipeline execution status  
**changed –** run this post block only if the pipeline’s execution status is different from the previous build run. E.g., the build failed at an earlier run & ran successfully this time.  
**aborted –** if the build is aborted in the middle of the run. Usually due to manual stopping of the build run  
**failure –** if the build status is failure  
**success –** if the build ran successfully  
**unstable –** build is successful but not healthy. E.g. On a particular build run, test cases ran successfully, but test coverage % is less than expected, then the build can be marked as unstable

**Example-**

<table><tbody><tr><td data-settings="show"><div><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p></div></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>stage</span><span>(</span><span>'build step'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>echo</span><span> </span><span>"Build stage is running"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>post</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>always</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>echo</span><span> </span><span>"You can always see me"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>success</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"I am running because the job ran successfully"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>unstable</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>"Gear up ! The build is unstable. Try fix it"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>failure</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>echo</span><span> </span><span>"OMG ! The build failed"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.09.29-AM.png)

Sample can be found here at [GitHub](https://github.com/iamvickyav/jenkins-declarative-pipeline-notes/blob/master/declarative-syntax-samples.md#post).

## Marking build as Unstable

There are scenarios where we don’t want to mark build as failure but want to mark build as unstable.

**Example-** When test case coverage doesn’t cross the threshold we expect, we can mark the build as **UNSTABLE**.

<table><tbody><tr><td data-settings="show"><div><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p><p>31</p><p>32</p><p>33</p><p>34</p><p>35</p><p>36</p><p>37</p><p>38</p></div></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>tools</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>maven</span><span> </span><span>'MAVEN_PATH'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>jdk</span><span> </span><span>'jdk8'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>"Tools initialization"</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>sh</span><span> </span><span>"mvn --version"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>sh</span><span> </span><span>"java -version"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>"Checkout Code"</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>git </span><span>branch</span><span>:</span><span> </span><span>'master'</span><span>,</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>url</span><span>:</span><span> </span><span>"https://github.com/iamvickyav/spring-boot-data-H2-embedded.git"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>"Building Application"</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>sh</span><span> </span><span>"mvn clean package"</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>"Code coverage"</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>jacoco</span><span>(</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>execPattern</span><span>:</span><span> </span><span>'**/target/**.exec'</span><span>,</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>classPattern</span><span>:</span><span> </span><span>'**/target/classes'</span><span>,</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>sourcePattern</span><span>:</span><span> </span><span>'**/src'</span><span>,</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>inclusionPattern</span><span>:</span><span> </span><span>'com/iamvickyav/**'</span><span>,</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>changeBuildStatus</span><span>:</span><span> </span><span>true</span><span>,</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>minimumInstructionCoverage</span><span>:</span><span> </span><span>'30'</span><span>,</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>maximumInstructionCoverage</span><span>:</span><span> </span><span>'80'</span><span>)</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span></span><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.09.53-AM.png)

Sample can be found here at [GitHub](https://github.com/iamvickyav/jenkins-declarative-pipeline-notes/blob/master/declarative-syntax-samples.md#mark-build-as-unstable-based-on-code-coverage).

**jacoco()** is a special method which helps us configure Jacoco reports including the minimum & maximum coverage threshold. In the above example, min coverage (minimumInstructionCoverage) check is set as 30 & max coverage (maximumInstructionCoverage) check is set as 80.

So if the code coverage is-

**Less than 30 –** Build will be marked as Failure.  
**Between 30 & 80 –** Build will be marked as Unstable.  
**Above 80 –** Build will be marked as Success.

## triggers

Instead of triggering the build manually, we can configure build to run in certain time intervals using the triggers block. We can use the special method called cron() within triggers block to configure the build schedule.

**Understanding cron**

Cron configuration contains 5 fields representing minute (0-59), hour (0-23), day of the month (1-31), month (1-12), day of the week (0-7)

**Example-**

**Every 15 minutes – H/15 \* \* \* \*  
Every 15 minutes but only between Monday & Friday – H/15 \* \* \* 1-5**

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>pipeline</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>agent</span><span> </span><span>any</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>triggers</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>cron</span><span>(</span><span>'H/15 * * * *'</span><span>)</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stages</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>stage</span><span>(</span><span>'Example'</span><span>)</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>steps</span><span> </span><span>{</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>echo</span><span> </span><span>'Hello World'</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>}</span></p></div></td></tr></tbody></table>

![](https://www.lambdatest.com/blog/wp-content/uploads/2021/04/Screenshot-2021-02-13-at-10.10.18-AM.png)

Sample can be found here at [GitHub](https://github.com/iamvickyav/jenkins-declarative-pipeline-notes/blob/master/declarative-syntax-samples.md#triggers-sample).

## Conclusion

In this blog, we have done an in-depth dive into Jenkins Declarative pipeline examples and their usage. Instead of configuring build steps using UI in a remote Jenkins portal, we would always recommend you to prefer creating Jenkinsfile with Declarative pipeline syntax. Jenkinsfile, since part of the application’s source code, will provide more control over CI/CD steps to developers. That’s the best way to make the most of Jenkins CI/CD and all the features it has to offer!

Happy building & testing!

[![](https://secure.gravatar.com/avatar/324c898e49d590f01efb23a481fd1329?s=150&d=mm&r=r)](https://www.lambdatest.com/blog/author/praveenmishra/)

#### [Praveen Mishra](https://www.lambdatest.com/blog/author/praveenmishra/)

Praveen is a Computer Science Engineer by degree, and a Digital Marketer by heart who works at LambdaTest. A social media maven, who is eager to learn & share about everything new & trendy in the tech domain.