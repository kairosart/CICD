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

1. Click on your profile icon and select Preferences:
	![[Securing the Build Source-20250321200135628.webp]]
	
2. Click on Access Tokens:
	![[Securing the Build Source-20250321200236047.webp]]
3. Enter a name for a new API token and select the api, read_api, and read_repository scopes:
	![[Securing the Build Source-20250321200535101.webp]]
4. Click Create personal access token, reveal, and copy your access token to a safe location:
	![[Securing the Build Source-20250321200701599.webp]]
	
>[!Note]
>Personal access token:
>On `.env` file.

#Attacking_Machine
5. Create the `enumerator.py` script.
6. Add the token to the script and execute it to download all repos:
	![[Securing the Build Source-20250321202256661.webp]]
Now you have successfully downloaded all of the publicly available repos! At this point, there are several ways you can look for sensitive information. The easiest would be to extract all repos and run a grep for specific keywords, such as `secret`. Make sure to read through the available repos to find the hidden secret flag! 

>[!warning]
Don't change the name of the script (enumerator.py) as it will throw errors!

## Securing the Build Source

Granular access control is crucial to managing repositories and the GitLab platform. It involves defining specific permissions and restrictions for different users or groups, ensuring that only authorised individuals have the appropriate level of access to sensitive resources. This helps maintain security, confidentiality, and effective collaboration within a development environment.

In GitLab, group-based access control is a powerful mechanism that simplifies permissions management across multiple repositories and projects. Here's how it works:

1. **Group-Based Access Control**: GitLab allows you to organise projects into groups. Instead of managing access for each project separately, you can set permissions at the group level. This means that the same access rules apply to all projects within the group, making it easier to maintain consistent security policies. For example, you can create a group for the development team and define permissions, such as who can view, edit, or contribute to projects within that group. This approach streamlines access management and reduces the chances of errors or oversights when configuring permissions for individual repositories.
2. **Access Levels** : GitLab offers different access levels, such as Guest, Reporter, Developer, Maintainer, and Owner. Each level comes with specific capabilities and permissions. Assigning the appropriate access level to each user or group ensures they have the necessary privileges without granting unnecessary permissions.
3. **Sensitive Information Protection** : One critical consideration is preventing the accidental exposure of sensitive information. GitLab provides features to help with this:

4. **GitLab's .gitignore** : This file specifies which files or directories should be excluded from version control. It's crucial for preventing sensitive data like passwords, API keys, and configuration files from being committed to repositories.
5. **Environment Variables** : GitLab allows you to define and manage environment variables securely, separate from the source code. This is especially useful for storing sensitive data needed during the CI/CD process without exposing it in the repository.
6. **Branch Protection** : Branches, like master or main, can be protected to prevent direct pushes, ensuring that changes go through code review and automated testing before merging.

Remember, maintaining the security of both the repositories and the GitLab instance itself requires constant vigilance and best practices:

- Review and update access permissions regularly as team members change roles or leave the organisation.
- Implement two-factor authentication (2FA) to add an extra layer of security to user accounts.
- Monitor audit logs to track who has accessed or modified repositories and projects.
- Regularly scan repositories for sensitive information using tools designed for this purpose.


---
>[!Question]
>1. Which file specifies which directories and files should be excluded for version control?
>`.gitignore`
>2. What can you protect to ensure direct pushes and vulnerable code changes are avoided?
>`Branches`
>3. What issue does lack of access control and unauthorised code changes lead to?
>`Tnauthorised Tampering`
>4. What is the API key stored within the Mobile application that can be accessed by any Gitlab user?
>- Unzip Mobile `App_7ab91f90-f077-4817-bd05-5fc956ab21bd.zip`.
>- Run: `grep -r "THM" .`
>`THM{You.Found.The.API.Key}`


**Next step:**  [[Securing the Build Process]]
