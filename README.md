# Setting up a Tahoe-LAFS Grid

This manual explains the configuration of a simple Tahoe-LAFS Cloud Storage Network on Linux.

**For more info on Tahoe-LAFS (Least Authority File System) visit https://tahoe-lafs.readthedocs.io/**

## Setting up an Introducer

The introducer is needed to connect all Storage Nodes and allow users to join the Grid.

First, let's create a new unprivileged user:

~~~bash
adduser --disabled-password --gecos "" tahoe
~~~

Then install the Tahoe-LAFS Software and switch to the newly created user:

~~~bash
apt install tahoe-lafs
su - tahoe
~~~

Next, run the command to create an introducer config file:

~~~bash
tahoe create-introducer --port=tcp:1234 --location=tcp:<replace with public IP>:1234 --basedir=introducer
~~~

This also generates a so-called FURL that you need to know in order to connect Nodes or Clients to the network. Run:

~~~bash
tahoe run --basedir introducer
~~~

...wait some time and press `Ctrl-C`.

Then, display the FURL and exit back to `root`:

~~~bash
cat introducer/private/introducer.furl # write it down or copy it into a text file
exit
~~~

Finally, create a Systemd service for automatically running the introducer at boot time: Go to `/etc/systemd/system/tahoe-introducer.service` and paste the following text:

~~~
[Unit]
Description=Tahoe-LAFS autostart introducer
After=network.target

[Service]
Type=simple
User=tahoe
WorkingDirectory=/home/tahoe
ExecStart=/usr/bin/tahoe run introducer --logfile=logs/introducer.log

[Install]
WantedBy=multi-user.target
~~~

To enable the service, type `systemctl enable tahoe-introducer.service` and restart the system. Your introducer should now be running.

## Setting up some Storage Nodes

The Storage Nodes actually store the client's encrypted data. The first steps are the same as above:

~~~bash
adduser --disabled-password --gecos "" tahoe
apt install tahoe-lafs
su - tahoe
~~~

Then, you need to create a Node configuration file:

~~~bash
tahoe create-node --nickname=<replace with a nickname for the node> --introducer=pb://<replace with introducer FURL> --port=tcp:1235 --location=tcp:<replace with node public IP>:1235
~~~

Now, `exit`.

Next, create a new systemd service at `/etc/systemd/system/tahoe-node.service`:

~~~
[Unit]
Description=Tahoe-LAFS autostart node
After=network.target

[Service]
Type=simple
User=tahoe
WorkingDirectory=/home/tahoe
ExecStart=/usr/bin/tahoe run .tahoe --logfile=logs/node.log

[Install]
WantedBy=multi-user.target
~~~

Also, run `systemctl enable tahoe-node.service`. Finally, restart your machine.

**Repeat this step for every new node in the grid.**

## Testing the Grid

To test if all nodes are connected to the introducer, you need to set up a client.

~~~bash
apt install tahoe-lafs
tahoe create-client --nickname=<relpace with client nickname> --introducer=<replace with introducers FURL>
~~~

To start the Tahoe-LAFS Web Interface at `localhost:3456`, simply type:

~~~bash
tahoe run
~~~

Open a web browser at `localhost:3456` and check the Grid status. Everything should now be up and running.

## Using the Grid with Gridsync

Download Gridsync from the GitHub Repository: `https://github.com/gridsync/gridsync` Start it up and select `Manual configuration`. Then type in a Grid name and paste your introducer FURL. Then connect to the Network.

**For more info on Gridsync, check out the GitHub Project Repository**

Thanks for reading.
