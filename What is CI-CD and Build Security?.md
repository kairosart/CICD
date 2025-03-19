## Introduction  

Establishing a secure build environment is crucial to safeguarding your software development process against potential threats and vulnerabilities. In light of recent events such as the SolarWinds supply chain attack, it has become increasingly evident that building a robust security foundation is imperative. This task will explore key strategies to create a secure build environment, considering the lessons learned from the SolarWinds use case.

## Fundamentals of CI/CD  

According to [Gitlab](https://about.gitlab.com/topics/ci-cd/), there are eight fundamentals for CI/CD:

- **A single source repository** - Source code management should be used to store all the necessary files and scripts required to build the application.  

- **Frequent check-ins to the main branch** - Code updates should be kept smaller and performed more frequently to ensure integrations occur as efficiently as possible.  

- **Automated builds** - Build should be automated and executed as updates are being pushed to the branches of the source code storage solution.  

- **Self-testing builds** - As builds are automated, there should be steps introduced where the outcome of the build is automatically tested for integrity, quality, and security compliance.  

- **Frequent iterations** - By making frequent commits, conflicts occur less frequently. Hence, commits should be kept smaller and made regularly.  

- **Stable testing environments** - Code should be tested in an environment that mimics production as closely as possible.  

- **Maximum visibility** - Each developer should have access to the latest builds and code to understand and see the changes that have been made.  

- **Predictable deployments anytime** - The pipeline should be streamlined to ensure that deployments can be made at any time with almost no risk to production stability.

While all of these fundamentals will help to ensure that CI/CD can streamline the DevOps process, none of these really focus on ensuring that the automation does not increase the attack surface or make the pipeline vulnerable to attacks.

## A Typical CI/CD Pipeline

So what does a typical CI/CD-enabled pipeline look like? The network diagram of this room helps a bit to explain this. Let's work through the different components that can be found in this pipeline:

- **Developer workstations** - Where the coding magic happens, developers craft and build code. In this network, this is simulated through your AttackBox.  

- **Source code storage solution** - This is a central placeholder to store and track different code versions. This is the Gitlab server found in our network.  

- **Build orchestrator** - Coordinates and manages the automation of the build and deployment environments. Both Gitlab and Jenkins are used as build servers in this network.  

- **Build agents** - These machines build, test and package the code. We are using GitLab runners and Jenkins agents for our build agents.  

- **Environments** - Briefly mentioned above, there are typically environments for development, testing (staging) and production (live code). The code is built and validated through the stages. In our network, we have both a DEV and PROD environment.  


Throughout this room, we will explore the CI/CD components in more detail and show the common security misconfigurations found here. It should be noted that DevOps pipelines with CI/CD can take many forms depending on the tools used, but the security principles that should be applied remain the same.

## SolarWinds Case

The SolarWinds breach was a significant cyberattack discovered in December 2020. The attackers compromised SolarWinds' software supply chain, injecting a malicious code known as SUNBURST into the company's Orion software updates. The attackers managed to gain unauthorised access to numerous organisations' networks, including government agencies and private companies. The breach highlighted the critical importance of securing software supply chains and the potential impact of a single compromised vendor on many entities.

We will use this case as we go through measures we can apply to ensure our build environment is secure. These measures are implementing isolation and segmentation techniques and setting up appropriate access controls and permissions to limit unauthorised access.

## **Implement Isolation and Segmentation Techniques**

The SolarWinds incident highlighted the significance of isolating and segmenting critical components within the build system. By separating different stages of the build process and employing strict access controls, you can mitigate the risk of a single compromised component compromising the entire system. Isolation can be achieved through containerisation or virtualisation technologies, creating secure sandboxes to execute build processes without exposing the entire environment.

## **Set Up Appropriate Access Controls and Permissions**

Limiting unauthorised access to the built environment is crucial for maintaining the integrity and security of the system. Following the principle of least privilege, grant access only to individuals or groups who require it to perform their specific tasks. Implement robust authentication mechanisms such as multi-factor authentication (MFA[^1]) and enforce strong password policies. Additionally, regularly review and update access controls to ensure that access privileges align with the principle of least privilege.

Implementing strict controls on privileged accounts is essential, including limiting the number of individuals with administrative access and strict monitoring and auditing mechanisms for privileged activities.

## **A note on Network Security**  

Network security is vital in protecting the build system from external threats. Implementing appropriate network segmentation, such as dividing the built environment into separate network zones, can help contain potential breaches and limit lateral movement. Here are a few more essential points to consider:

- Implement secure communication channels for software updates and ensure that any third-party components or dependencies are obtained from trusted sources. 
- Regularly monitor and assess the security of your software suppliers to identify and address potential risks or vulnerabilities.

Learning from incidents such as the SolarWinds attack helps us recognise the critical importance of securing the entire build process, from code development to deployment, to safeguard against potential threats and ensure the trustworthiness of your software.

[^1]: MFA stands for Multi-Factor Authentication, is a security process that requires users to provide two or more forms of identification before accessing an account or system. This enhances security by adding an additional layer of protection against unauthorised access.

---
>[!Question]
>1. What element of a CI/CD pipeline coordinates and manages the automation of build and deployment environments?
>`build orchestrator`
>2. What element of a CI/CD pipeline builds, tests, and packages code?
>`build agents`
>3. What fundamental of CI/CD promotes developers in having access to the latest builds and code in order to understand and see the changes that have been made?
>`maximum visibility`

**Next step:** [[Creating your own Pipeline]]
