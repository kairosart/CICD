Now that we have paid attention to protecting the build source of our pipeline, next on our journey is ensuring that the build process itself does not have misconfigurations that can lead to compromise.

## Managing Dependencies

The first step to securing the build process is to secure the dependencies of the build. As our pipeline kicks off, it will start by compiling our source code into the final build. However, our source code may depend on external libraries and SDKs for its functionality. Thus, during this compilation process, the build pipeline will gather these dependencies to perform the build. There are two main concerns for our build process when it comes to dependencies:

- Supply Chain Attacks - If a threat actor can take over one of these dependencies, they would be able to inject malicious code into the build
- Dependency Confusion - If an internally developed dependency is used, an attacker could attempt a dependency confusion attack to inject code into the build process itself.  


Both of these attacks have already been covered in the [Dependency Management room](http://tryhackme.com/jr/dependencymanagement) if you want to learn more and practically exploit these attacks. In this task, we will focus on a misconfiguration within the build process itself.

## Knowing When to Start the Build

A big issue with pipelines and the build process is that, in a nutshell, whether you like to hear it or not, it is remote code execution as a feature. Once a pipeline kicks off, the build server communicates to one of the build agents to perform the build, which includes reading the commands that have to be executed from the CI file and performing them. While this creates automation, it also creates the risk that if an attacker can alter what is being built or when, they might be able to leverage this code execution to compromise systems. Therefore, we need to pay special attention to the following:

- What actions do we allow to kick off the build process?
- Who has permission to perform these actions to kick off the build process?
- Where will the build process occur?  

The answers to these questions can help you determine the attack surface of your pipeline. While, in most cases, a bad answer to one of these questions won't compromise either the build or the pipeline, there are some toxic combinations that you must be aware of. Let's take a bit of a closer look at these three questions.

### **What actions start the build process**

We have the ability to decide what actions can start the build process. Normally, by default, a commit of new code to the source will start the pipeline. But we do have the ability to provide a much more granular configuration. For example, we can decide that only commits to specific branches, such as main, should start the pipeline. This configuration means that we can, with a lot more peace of mind, allow developers to make direct commits to other branches. As long as we limit who has the ability to either directly commit to the main branch or approve merge requests for it, we can limit the attack surface of our pipeline.

However, this might run us into the issue where those merge requests break in the pipeline, causing us to perform multiple merges just to fix the issue, which can be tedious. Therefore, there may be a use case to have the build process already start on other branches or when new merge requests are made to indicate whether the merge request would break our pipeline. If we choose to go down this path, we must understand that our attack surface has grown since multiple actions could start the build process. Nothing to worry about just yet, but we must ensure that these actions cannot be performed simply by anyone!

### **Who can start the build process**

Once we decide which actions can start the build process, we need to narrow down who can perform these actions. As mentioned before, the pipeline only executes when code is merged to the main branch; this can be a very small list of users who have the ability to approve these merges. The question becomes more complicated if we allow builds to kick off from other actions. Based on the actions (and branches) that can start the build, we will have to ask who can start it and add them to the list, thereby growing the attack surface. For example, if we allow builds to start on merge requests themselves, we have to ensure that the attacker cannot make a merge request or that the merge build will occur in a segregated environment.

### **Where will the build occur**

Lastly, we need to decide where the build will occur. We don't have to simply rely on a single build agent to perform all of our builds. In the above example, if we want developers to run builds on other branches, we can just simply register a new build agent that will run a build in a different environment than the build agent of the main branch. Based on our answers to the previous two questions, we may need to ensure we secure where the build will execute. If we allow multiple actions to start the build, we probably want to ensure that the same build agent is not used for all of these actions, as they have different degrees of sensitivity.  

Now that we understand the three questions we must answer, let's explore a very interesting but common toxic combination!