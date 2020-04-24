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


B2S - 2 cores 4 Gig

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

ExecStop=/usr/bin/screen -p 0 -S mc-%i -X eval 'stuff "say SERVER SHUTTING DOWN IN 5 SECONDS. SAVING ALL MAPS..."\015'
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