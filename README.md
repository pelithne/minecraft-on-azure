Create Minecraft user

````
$ sudo useradd -m -r -d /opt/minecraft minecraft
````

Install Minecraft Server
````
$ sudo mkdir /opt/minecraft/server
$ sudo wget -O /opt/minecraft/server/minecraft_server.jar https://s3.amazonaws.com/Minecraft.Download/versions/1.12.2/minecraft_server.<current version>.jar
$ sudo bash -c "echo eula=true > /opt/minecraft/survival/eula.txt"
$ sudo chown -R minecraft /opt/minecraft/server/
````

Autostart

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

ExecStart=/usr/bin/screen -DmS mc-%i /usr/bin/java -Xmx4    G -jar minecraft_server.jar nogui

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