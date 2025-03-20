## Introduction

Before we explore misconfigurations, it is worth us first creating our very own pipeline so we can play around.

### Gitlab Registration

We will start the process by creating an account on the GitLab instance. Navigate to [http://gitlab.tryhackme.loc](http://gitlab.tryhackme.loc/) and select the **Register Now** option on the page. Fill in the required details to create your account and register. Remember to select a secure password and save it for your journey through this network! Once authenticated, you should be met by this page:

![[Creating your own Pipeline-20250319193905987.webp]]

Feel free to explore the features of the Gitlab server. Gitlab is quite similar to Github; however, it allows you to host your very own server!

## Project Creation

Now that you have an account, the next step is to create a new project. Feel free to create some of your projects directly to play around with the feature. However, since we want to play around with pipelines, you can make a fork of an existing project that has been created for this purpose.

Click on the _Your work tab_, then the Explore option, and search for BasicBuild. You will see a project like this:

![[Creating your own Pipeline-20250319201459439.webp]]

Once done, you can fork the project! This now creates a copy of the project you own and can configure.

## Understanding CI/CD Configuration

In Gitlab, project automation is defined in the .gitlab-ci.yml file. This file contains the steps that will be performed automatically when a new commit is made to the repo. Understanding what the commands in this file do will be important on your journey of learning about build security. Let's take a look at the contents of the file.

### **Stages**

GitLab CI files allow you to define various jobs. Each job has different steps that must be executed, as found in the _script_ section of the job. Usually, there are three stages to any CI pipeline, namely the build, test, and deploy stage, but there can be more. Let's take a look at the build job:

**.gitlab-ci.yml**

```
build-job: stage: build 
	script: 
		- echo "Hello, $GITLAB_USER_LOGIN!"
```

- The first line, build-job, is the name of the job. 
- The stage value is to tell Gitlab which stage of the CI Pipeline this job belongs to. 
- As mentioned before, this can be built, tested, or deployed. 
- Jobs in each stage are executed in parallel. That means all build jobs will execute simultaneously, as will all test and deploy jobs. 
- If a job fails, later stages will not start. 
- If a job does not specify a stage, it will automatically be assigned to the test stage. 
- If you have to change the job order, you can use the _needs_ section to define the names of jobs that have to be completed before the next job can be executed.

The script portion details the commands that will be executed as part of the job. As you can see, only an echo command will execute for this build job. However, we normally complete all the build activities in the build stage, such as loading dependencies and compiling our code. Since we are deploying a simple PHP website, there is no reason for code compiling.

### **Tests**

Tests jobs are meant to perform tests on the build to ensure everything works as expected. Usually, you would execute more than one test job to ensure that you can individually test portions of the application. If one test job fails, the other test jobs will still continue, allowing you to determine all the issues with the current build, rather than having to do multiple builds. Let's take a look at the two test cases:

**.gitlab-ci.yml**

```
test-job1:
  stage: test
  script:
    - echo "This job tests something"

test-job2:
  stage: test
  script:
    - echo "This job tests something, but takes more time than test-job1."
    - echo "After the echo commands complete, it runs the sleep command for 20 seconds"
    - echo "which simulates a test that runs 20 seconds longer than test-job1"
    - sleep 20
```

As you can see, we don't have anything to test with our simple PHP web application. However, we can simulate that one test case will take longer than the other.

### **Deployment**

In the deployment stage, we want to deploy our application to the relevant environment if both the build and test stages succeed. Usually, branches are used in source code repos, with the main branch being the only one that can deploy to production. Other branches will deploy to environments such as DEV or UAT. Let's take a look at what we are doing in the deployment job:

**.gitlab-ci.yml**

```
deploy-prod:
  stage: deploy
  script:
- echo "This job deploys something from the $CI_COMMIT_BRANCH branch."
    - echo "Making the website directory"
    - mkdir -p /tmp/time/cicd
    - echo "Copying the website files"
    - cp website_src/* /tmp/time/cicd/
    - echo "Hosting website using a screen"
    - screen -d -m php -S 127.0.0.1:8081 -t /tmp/time/cicd/ &    
    - echo "Deployment complete! Navigate to http://localhost:8081/ to test!"
  environment: production
```

- The first step of the deploy job is to create a new directory under `/tmp/` where we can place our web application. 
- We then copy the web application files to the directory and alter the permissions of the files. 
- Once done, host the application, making use of PHP. Now, we are ready to launch our application!

_CI[^1] files_ can become a lot more complex as there are a lot more sections and keywords that you could use. If you want to learn more, you can look [here](https://docs.gitlab.com/ee/ci/yaml/index.html). Now that we better understand the embedded automation, let's look at actually using it. To have the build execute, we need to register a runner.

### Runner Registration

In Gitlab, we use runners to execute the tasks configured in the project. Let's follow the process to register your attack machine as a runner for your project.


>[!Note]
Make sure that PHP is installed (`sudo apt install php7.2-cli`) before continuing. Note that if you are doing this on your own machine, ensure you are okay with the runner deploying a web application to your machine. If not, it is best to perform this on the AttackBox.

In your project, click on Settings and then CI/CD:

![[Creating your own Pipeline-20250319203214621.webp]]

Expand the Runners section, and you should see the following screen:

![[Creating your own Pipeline-20250319203421602.webp]]

Here, you will be able to configure a new runner for your project. Click the three dots and the Show Runner installation steps button. You will see the following screen:

![[Creating your own Pipeline-20250319203504173.webp]]

- The first code block is to install the GitLab-runner application. Follow these steps to install the application on your attack machine. 
- Once done, use the command in the second code block to register your runner. Follow the prompts as shown below for the installation process:

![[Creating your own Pipeline-20250319204550692.webp]]

Now that your runner is configured, you can refresh the page on Gitlab, and you should see your runner:

![[Creating your own Pipeline-20250319204625833.webp]]

The last step is configuring your runner to execute untagged jobs, as our current CI Pipeline does not use tags. Normally, tags would be used to ensure that the correct runners pick up the various jobs. However, since we have a simple job, we can just tell our runner to execute all jobs. Click the little pencil icon and click Run untagged jobs:

![[Creating your own Pipeline-20250319204701220.webp]]

You are now ready to start the build process!

### Build Automation

Now that the runner is registered, we can test the build process by making a new commit. The easiest change to make that would kick off a build is to update the README.md file:

1. Click on the file in the repo
2. Select Edit
3. Select Edit single file
4. Make an update to the file
5. Click Commit changes

Once done, your build process will have started! We can follow the process by clicking on Build and then Pipelines[^2]:

![[Creating your own Pipeline-20250319205100619.webp]]

Once there, you should see that your pipeline has kicked off! Click on the pipeline to view which job it is currently performing and the output thereof:

![[Creating your own Pipeline-20250320204119865.webp]]

You can also click on the pipeline to view its progress:

![[Creating your own Pipeline-20250320204205318.webp]]

Once completed, your application will have been deployed! You can verify this by navigating to [http://127.0.0.1:8081/](http://127.0.0.1:8081/), and you should be met by the web application homepage.

![[Creating your own Pipeline-20250320204332513.webp]]

Congrats! You have created your very own CI/CD pipeline and build process! Feel free to play around more with the CI/CD pipeline configuration and your runner!

>[!Note]
>If you wish to remove the website, you can use `sudo su gitlab-runner` followed by `screen -r` to connect to the screen that is hosting your website. From here, you will be able to terminate the website.


---



>[!Question]
>1. What is the name of the build agent that can be used with Gitlab?
>`Gitlab Runner`
>2. What is the value of the flag you receive once authenticated to Timekeep?
>Go to "Project Overview" and click on README.md. There you'll get the credentials to log in on Timekeep Server.
>`admin:admin`
>Go to Timekeep Server and log in.
>You'll get the flag.
>`THM{Welcome.to.CICD.Pipelines}`



**Next step: **[[Securing the Build Source]]

---

## Footnotes
[^1]: Continuous Integration is a software development practice that involves automatically building, testing and implementing changes to an application's source code

[^2]: Are a set of automated processes and tools that allows both developers and operations professionals to build and deploy code to a production environment.
