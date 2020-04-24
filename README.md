# Introduction
This page contains very brief instructions for setting up a minecraft server on a virtual machine in Azure. 

To be able to follow along, you need an Azure subscription. If you don't have one, you can sign up for free here: https://azure.microsoft.com/en-us/free/

The instructions are based on what can be found here: 
https://minecraft.gamepedia.com/Tutorials/Setting_up_a_server


## Start Ubuntu 18.04 server on Azure
First make sure you are logged in to the Azure portal, on https://portal.azure.com

Once logged in, search for "Ubuntu 18.04 server" from the search bar on the left hand side of the portal (this might change over time). Among the search results you should find the Ubuntu 18.04 server from **Canonical**, make sure to use this one.

<p align="left">
  <img width="40%"  src="./media/ubuntu.png">
</p>

Click on the **Create** button to start the process of creating the virtual machine.

In the next screen, fill out the details in a similar way as in the picture below. 

For the "Resource Group" choose **create new** and give is a nice name. I called it "mc". 

For "size" I selected B2s, which is a rather inexpensive VM size with 2 vCPUs and 4 Gig RAM, that will probably stand up for 10 simultaneous users (just guessing). 

<p align="left">
  <img width="40%"  src="./media/create-vm.png">
</p>


Further down the same page, you need to add a username/password (or ssh key, which is much more convenient if you know how to do it. I will not go into that now though). Also, you need to allow ````ssh```` in the inbound port section. 

THis is how it should look, sort of:

<p align="left">
  <img width="40%"  src="./media/access.png">
</p>

At this point you could either go ahead and click "Review + Create" or you can click on "next" to customize your VM further. If you click next, I suggest the following settings:

* Disks: Choose Standard SSD
* Networking: Keep defaults
* Management: Enable Auto-Shutdown (I will automatically shut down the server at midnight, for example)
* Advanced: Keep defaults
* Tags: A good thing to use, but lets leave that for a later time

Then go ahead and create the server. It will take a couple of minutes, after which you will be able to ssh into the VM and start installing and setting up minecraft.


## Create minecraft user

````
$ sudo useradd -m -r -d /opt/minecraft minecraft
````

## Install minecraft Server
````
$ sudo mkdir /opt/minecraft/server
$ sudo wget -O /opt/minecraft/server/minecraft_server.jar https://s3.amazonaws.com/Minecraft.Download/versions/<current version>/minecraft_server.<current version>.jar
$ sudo bash -c "echo eula=true > /opt/minecraft/survival/eula.txt"
$ sudo chown -R minecraft /opt/minecraft/server/
````

More details can be found here: 
https://minecraft.gamepedia.com/Tutorials/Setting_up_a_server


## Autostart

Create a file named /etc/systemd/system/minecraft@.service file with the following content:

````
[Unit]
Description=Minecraft Server: %i
After=network.target

[Service]
WorkingDirectory=/opt/minecraft/%i

User=minecraft
Group=minecraft

Restart=always

ExecStart=/usr/bin/screen -DmS mc-%i /usr/bin/java -Xmx3G -jar minecraft_server.jar nogui

ExecStop=/usr/bin/screen -p 0 -S mc-%i -X eval 'stuff "say server shutting down."\015'
ExecStop=/bin/sleep 5
ExecStop=/usr/bin/screen -p 0 -S mc-%i -X eval 'stuff "save-all"\015'
ExecStop=/usr/bin/screen -p 0 -S mc-%i -X eval 'stuff "stop"\015'


[Install]
WantedBy=multi-user.target


Start Minecraft Server
````
$ sudo systemctl start minecraft@survival
````

Confirm status:
````
$ sudo systemctl status minecraft@survival
````


Make sure  the server starts after reboot:
````
$ sudo systemctl enable minecraft@survival
````    