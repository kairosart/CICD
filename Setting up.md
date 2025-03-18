Patching In.  

Let's get you connected to the Nostromo and the greater extra-terrestrial network!

![[Setting up-20250318201956293.webp]]

**AttackBox**

If you are using the Web-based AttackBox, you will be connected to the network automatically if you start the AttackBox from the room's page. You can verify this by running the ping command against the IP of the Gitlab host. You should also take the time to make note of your VPN IP. Using `ifconfig` or `ip a`, make a note of the IP of the **cicd** network adapter. This is your IP and the associated interface that you should use when performing the attacks in the tasks.

**Other Hosts**

If you are going to use your own attack machine, an OpenVPN configuration file will have been generated for you once you join the room. Go to your [access](https://tryhackme.com/access) page. Select 'Cicdandbuildsecurity' from the VPN servers (under the network tab) and download your configuration file.

![[Setting up-20250318195909237.webp]]

Use an OpenVPN client to connect. This example is shown on the [Linux](https://tryhackme.com/access#pills-linux) machine; use this guide to connect using [Windows](https://tryhackme.com/access#pills-windows) or [macOS](https://tryhackme.com/access#pills-macos).

![[Setting up-20250318200024252.webp]]

The "Initialization Sequence Completed" message tells you you are now connected to the network. Return to your access page. You can verify you are connected by looking on your access page. Refresh the page and see a green tick next to Connected. It will also show you your internal IP address.

![[Setting up-20250318200105066.webp]]

## Configuring DNS

There is only two DNS entries for this network that are important. Thus, the simplest is to embed these DNS entry directly into your hosts file regardless of whether you are using the AttackBox or your own machine. To do this, review the network diagram above and make note of the IP of the Gitlab and Jenkins[^1] host. Then, perform the following command from a terminal:

`sudo echo <Gitlab IP> gitlab.tryhackme.loc >> /etc/hosts && sudo echo <Jenkins IP> jenkins.tryhackme.loc >> /etc/hosts`  

However, if you have already started the network and need to re-add this entry or update it, use your favourite text editor program to directly modify the entry in your `/etc/hosts` file. Once configured, you can navigate to [http://gitlab.tryhackme.loc](http://gitlab.tryhackme.loc/) to verify that your access is working. You should be met with the following page:

![[Setting up-20250318200238620.webp]]

## Contacting MU-TH-UR 6000  

As you progress through this network, you must report your work to the MU-TH-UR 6000 mainframe, better known as Mother. You will have to register with Mother before you begin this perilous journey. SSH is being used for communication as detailed below:

| **SSH Username** | mother          |
| ---------------- | --------------- |
| **SSH Password** | motherknowsbest |
| **SSH IP**       | X.X.X.250       |

Use your network diagram to replace the X values in your SSH IP. Once you authenticate, you will be able to communicate with Mother. Follow the prompts to register for the challenge, and save the information you get for future reference. Once registered, follow the instructions to verify that you can access all the relevant systems.

The VPN server and the Mother mainframe are not in scope for this network, and any security testing of these systems may lead to a ban from the network.

As you make your way through the network, you will need to prove your compromises. To do that, you will be requested to perform specific steps on the host that you have compromised. Please note the hostnames in the network diagram above, as you will need this information. Flags can only be accessed from matching hosts.  

**Note: If the network has been reset or you have joined a new subnet after your time in the network expired, your Mother account will remain active.**

[^1]: Jenkins is an open-source automation server widely used in DevOps for building, testing, and deploying software applications.
