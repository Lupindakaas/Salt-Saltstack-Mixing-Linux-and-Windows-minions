# SaltStack - Mixing Linux and Windows Minions

## Prerequisites

- Two Linux machine with internet access
- One Windows server machine with internet access

---

## Setting up the master

SSH into your linux machine and download Salt's bootstrapper:

    curl -o bootstrap-salt.sh -L https://bootstrap.saltproject.io

Give yourself execute permssion:

    	chmod +x bootstrap-salt.sh

Finally, run the bootstrapper:

	sudo ./bootstrap-salt.sh -M -N

This should install the Salt Master service on all supported Linux distro's.

Next, get the IP address of your master and write it down. 

### Setting up the firewall

The master needs TCP ports 4505 and 4506 to be accepted on all incomming connections. This changes depending om the Linux distro you use, so please look at SaltStack's official guide: https://docs.saltproject.io/en/latest/topics/tutorials/firewall.html

---

## Setting up the Linux Minion

SSH into the second Linux machine and download Salt's bootstrapper:

    curl -o bootstrap-salt.sh -L https://bootstrap.saltproject.io

Give yourself execute permssion:

    chmod +x bootstrap-salt.sh

Finally, run the bootstrapper:

	sudo ./bootstrap-salt.sh -M -N

This will install the Salt minion software. When it is finished update the configuration file with the master's IP address:

    sudo vi /etc/salt/minion

Change the line: "#master: salt" to "master: Your.masters.ip.address".

Then restart the salt-minion service:

    sudo systemctl restart salt-minion

The Linux minion should be all set now!

## Setting up the Windows Minion

### Firewall configuration

The minions need TCP ports 4505 and 4506 to be accepted on all outgoing connections. This can be configured by the following powershell command:

    netsh advfirewall firewall add rule name="Salt" dir=in action=allow protocol=TCP localport=4505-4506

### Install the minion software

First, download the Salt minion installer from their website.
https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/windows.html#install-windows

When it is finished downloading, run the installer. Next, Next, Next, until it asks for the IP address of the Salt master, where you should input the address of your salt-master. You can also input a unique name for the minion, but when left empty it will use the host name.

The Windows minion should be all set!

---

## Connecting the minion to the master

On the Salt master machine give the command: 
    
    sudo salt-key

This should show the two machine's under "unaccepted". To accept these keys give the following command:

    sudo salt-key -A

This will accept all unaccepted keys, so make sure there are no unknown machines from potential hackers!

It will ask for confirmation, so input 'Y' to accept.

It should return a list of the keys that have been accepted.

## Testing the connection

To test wether the connection is working, we will run the "ipconfig" command on the minion from the master. In the SSH session to your Linux machine run the following command:

    sudo salt '*' test.version

This should GET the current version of both minions, and PUT them into the console. 

---

## Troubleshooting

### The public keys aren't showing up in the master.

First do a sanity check by trying to ping between both machines(ICMP has to be accepted by the firewall). If you can ping, this is most likely a firewall issue. If you can't check the IP address and the networking/routing between the two machines.

### Error: "Master isn't responding"

Check if the "Salt-Master" service is actually running:

    sudo systemctl status salt-master.service

It should say active in green text. If it says anything else start it manually:

    sudo systemctl start salt-master.service

If that didn't fix the issue, try a reboot. That should fix most issues.

I ran into the issue of salt-master receiving a SIGKILL because of RAM limitations, so check your ram usage!

### Windows Salt minion key showed up, but command's fail

Check the logs in "C:\ProgramData\Salt Project\Salt\var\log\salt\minion". It should contain the error, so based on this error try to fix the error. In my case i got the error:

>2022-09-15 18:09:17,704 [salt.crypt       :1187][ERROR   ][16396] The master key has changed, the salt master could have been subverted, verify salt master's public key
2022-09-15 18:09:17,704 [salt.crypt       :800 ][CRITICAL][16396] The Salt Master server's public key did not authenticate!
The master may need to be updated if it is a version of Salt lower than 3004.2, or
If you are confident that you are connecting to a valid Salt Master, then remove the master public key and restart the Salt Minion.
The master public key can be found at:
C:\ProgramData\Salt Project\Salt\conf\pki\minion\minion_master.pub

This was fixed by deleting the minion_master.pub file and restarting the "Salt-minion" service in services.msc. I actually had to do this twice, for some reason.

### Windows keys don't show up

I have had exactly zero luck with either the winget version of salt-minion, or random bootstrappers. So make sure you get the salt-minion service from the download link in this tutorial.
