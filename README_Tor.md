# Setting up a Tahoe-LAFS Grid with Tor

This manual explains the configuration of a Tahoe-LAFS Cloud Storage Network with Tor.

**For more info on Tahoe-LAFS (Least Authority File System) visit https://tahoe-lafs.readthedocs.io/**

## Setting up an Introducer

The introducer is needed to connect all Storage Nodes and allow users to join the Grid.

Before anything else, let's install some dependencies and packages:

~~~bash
apt install tahoe-lafs python-pip tor nyx
pip install tahoe-lafs[tor]
~~~

Now let's create a new unprivileged user:

~~~bash
adduser --disabled-password --gecos "" tahoe
~~~

To be able to receive connections from Storage Nodes, you need to enable the Tor Hidden Service configuration in `/etc/tor/torrc` (change the following lines):

~~~
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 1234 127.0.0.1:1234
~~~

Also, execute these commands as `root`:

~~~bash
mkdir /var/lib/tor/hidden_service/
chown debian-tor:debian-tor /var/lib/tor/hidden_service/
chmod 0700 /var/lib/tor/hidden_service/
systemctl restart tor
cat /var/lib/tor/hidden_service/hostname
~~~

This will enable the Hidden Service and give you a `.onion`-Address. Write it down or save it somewhere.

Switch to the newly created user `tahoe` with:

~~~bash
su - tahoe
~~~

Next, run the command to create an introducer config file:

~~~bash
tahoe create-introducer --port=tcp:1234:interface=127.0.0.1 --location=tor:<onion_address>:1234 --basedir=introducer
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

**Warning:** You need to check if the `.onion`-Address ends with `onion`. Go to `~/introducer/tahoe.cfg` and make sure that the address at `tub.location` ends with `.onion`. For example:

~~~
tor:k22dn332q43t53g8rfd58673h0r9zje9r9segy8rh7t6653u1z4a.onion:1234
~~~

We also need to enable the tor connection in the same configuration file. Append the following text to the end of the file:

~~~
[connections]
tcp=tor
~~~

Then, make sure that under `[node]`, the line `reveal-IP-address` is `false`.

Finally, create a Systemd service for automatically running the introducer at boot time: Navigate to `/etc/systemd/system/tahoe-introducer.service` and paste the following text:

~~~
[Unit]
Description=Tahoe-LAFS Introducer
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

The Storage Nodes actually store the client's encrypted data. The first steps are exactly the same as above.

After changing to the `tahoe`-user, you need to create a Node configuration file:

~~~bash
tahoe create-node --nickname=<replace with a nickname for the node> --introducer=pb://<FURL> --port=tcp:1234 --location=tor:<onion_address>:1234
~~~

**Warning:** You need to check if the `.onion`-Address ends with `onion`. Go to `~/.tahoe/tahoe.cfg` and make sure that the address at `tub.location` ends with `.onion`. For example:

~~~
tor:kdcfobekwdcisogp3wehlds673h0r9ze9r9segy8rh7t6653u1z4a.onion:1234
~~~

We also need to enable the tor connection in the same configuration file. Append the following text to the end of the file:

~~~
[connections]
tcp=tor
~~~

Then, make sure that under `[node]`, the line `reveal-IP-address` is `false`.

Now, `exit`.

Next, create a new systemd service at `/etc/systemd/system/tahoe-node.service`:

~~~
[Unit]
Description=Tahoe-LAFS Node
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
apt install tahoe-lafs tor nyx python-pip
pip install tahoe-lafs[tor]
tahoe create-client --nickname=<nickname> --introducer=<FURL>
~~~

Before running the Client, you need to check if the `.onion`-Address ends with `onion`. Go to `~/.tahoe/tahoe.cfg` and make sure that the address at `tub.location` ends with `.onion`. For example:

~~~
tor:0xt01yfobekwdcip3wehlds673h0r9ze9r9segy8rh7t4ewd653u1z4a.onion:1234
~~~

You also need to enable the tor connection in the same configuration file. Append the following text to the end of the file:

~~~
[connections]
tcp=tor
~~~

Then, make sure that under `[node]`, the line `reveal-IP-address` is `false`.

To start the Tahoe-LAFS Web Interface at `localhost:3456`, simply type:

~~~bash
tahoe run
~~~

Open a web browser at `localhost:3456` and check the Grid status. Everything should now be up and running.

## Using the Grid with Gridsync

Download Gridsync from the GitHub Repository: `https://github.com/gridsync/gridsync` Start it up and select `Manual configuration`. Then type in a Grid name and paste your introducer FURL. Then enable the `Tor` Checkbox and connect to the Network.

**For more info on Gridsync, check out the GitHub Project Repository**

Thanks for reading.
