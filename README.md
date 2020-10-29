# Gateway Services - Ubuntu Server 20.04
Documentation on how to set DNS, DHCP, Firewall, NAT and Caching services on Ubuntu 20.04
Note: This installation was done on VMWare, But the configuration files will work for any install.

## Description
This is a lab enviroment done in VMWare to deploy services on Ubuntu server.
This server will provide DNS resolution for the clients, DHCP, Firewall (ufw) rules, NAT and Caching for clients.
On this environment there are two machines, the server and a client machine (in this case Ubuntu desktop). The objective install and configurer all of this services in one server.

- There are two NIC on the server, ens38 with the ip addresess 192.168.0.15/24 (External Network)
- ens33 with the ip address of 10.0.0.250/24 (Internal Network)

## Requirements

- Ubuntu Server 20.04.01 LTE use this link -> <https://ubuntu.com/download/server>

- Ubuntu Desktop 20.04.1 LTE use this link -> <https://ubuntu.com/download/desktop>

- The server needs to have two NIC cards attached.

### Install on VMWare
- Create a new VM and Install the Ubuntu Server. 
- Create an account to access ubuntu server
- Install SSH service right on the installation page
- Note: Remember to add two NIC cards for this VM. One will be a external network and the other the internal network 
- For the internal network you will have to go on the VM Settings on both Server and Client, choose the NIC that will be used on the internal network and set to Custom and then specify a "VMnet" (You can choose any, as long as both server and client have the same VMnet custom network on the internal NIC)
- External Nic on the Server is set to Bridged
- When the system boot up remember to update & upgrade
```
sudo apt update
sudo apt upgrade
```
- Go ahead and also Install the client VM at this point.

### Network
- It's a good idea to set up a static ip address to access the server though SSH.
- To do this, use this command sudo nano /etc/netplan/00-installer-config.yaml
- Add your network configuration as the example bellow:

```
#This is the network config written by 'subiquity'
network:
 ethernets:
    ens33:
      addresses: [192.168.0.15/24]
      gateway4: 192.168.0.1
      nameservers:
        addresses: [8.8.8.8]
    ens38:
      addresses: [10.0.0.250/24]
  version: 2
  ```
After configuring the netplan you must save this configuration (Ctrl-X and save) then apply with the following command:
```
sudo netplan apply
```
After that you can check if it's working by first checking the network connections and then ping google.com to verify that you can reach the internet.
```
ip a

ping google.com
```
If you have a response you're good to go.

Note: You can test you internal connection by statically assigning an IP address on the 10.0.0.0/24 range just for the purpose of testing their connections. After you verified that both machines can ping and are on the same network, leave the network configuration on automatic.

## DNS

To install DNS on the server type the following command:

```
sudo apt install bind9 dnsutils
```
#### To check the status of the service
```
sudo systemctl status named
```
#### To activate, deactivate and restart the service
```
sudo systemctl start named
sudo systemctl stop named
sudo systemctl restart named
```
#### Configuring Forwarders

Go to the file
```
sudo nano /etc/bind/named.conf.options
```
and add 8.8.8.8 (google.com) as forwarder:
```
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      8.8.8.8;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        listen-on-v6 { any; };
};
```
#### Configure a zone transfer file

Go to the file
```
sudo nano /etc/bind/named.conf.local
```
and add a zone at the end like the example bellow (in this case my zone is called "daniel.sysninja"
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "daniel.sysninja" {
     type master;
     file "/etc/bind/db.daniel";
};
```
#### Configure a new record for your Zone

Open the file "db.local" fill your record information and save as a new file, in this case I save it as "db.daniel"

Example bellow:
```
;
; BIND data file for local loopback interface
;
$TTL    604590
@       IN      SOA     daniel.sysninja. root.danielsysninja.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      daniel.sysninja.
ns      IN      A       10.0.0.250
@       IN      A       1.1.1.1
*       IN      A       10.0.0.250
```
- Make sure to add a wildcard so it can find anything ending with your record

- After doing all the changes you can use the command to restart the service "named"



## DHCP

- Install DHCP to provide ip addresses for your client
```
sudo apt install isc-dhcp-server
```
#### To check the status of the service
```
sudo systemctl status isc-dhcp-server
```
#### To activate, deactivate and restart the service
```
sudo systemctl start isc-dhcp-server
sudo systemctl stop isc-dhcp-server
sudo systemctl restart isc-dhcp-server
```
#### Configure DHCP range
Go to dhcpd.conf file and add the following statement
```
sudo nano /etc/dhcp/dhcp.conf
```
```
subnet 10.0.0.0 netmask 255.255.255.0 {
  range 10.0.0.100 10.0.0.200;
  option routers 10.0.0.250;
  option domain-name-servers 10.0.0.250, 8.8.8.8;
  option domain-name "daniel";
```
Now just restart the service and verify that your client received a ip address.

## Firewall & NAT

- First step is to activate "ufw" to allow or deny applications from accesing your server
- To Activate use the folling command:
```
sudo ufw enable
```
to check the firewall rules:
```
sudo ufw status
```
It must diplay your rules. On this case there are some rules already in place.
```
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
53                         ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
22/tcp                     DENY        10.0.0.10
22/tcp                     ALLOW       192.168.0.30
8888/tcp                   ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
53 (v6)                    ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
8888/tcp (v6)              ALLOW       Anywhere (v6)
```
You can add rules to allow or block machines with:
```
sudo ufw allow 10.0.0.10
sudo ufw deny 10.0.0.10
```
You can also allow or deny protocols and ports
```
sudo ufw allow ssh
sudo ufw deny ssh
sudo ufw allow 22
```
To view a list of applications that are running and that can be referenced by name
```
sudo ufw app list
```
- In this case I want to allow protocol tcp on port 80 (This will be important when NAT is in place) and I don't want my client to connect though ssh.

#### Activating IP Mascarating and NAT so client can access web though the server.

Edit the ufw file to accept DEFAULT_FORWARD_POLICY:
```
sudo nano /etc/default/ufw
```
```
DEFAULT_FORWARD_POLICY="ACCEPT"
```
Now edit the before.rules files and add the rule in the begginng of the file, go to /etc/ufw/before.rules:
```
sudo nano /etc/ufw/before.rules
```
```
# Nat tables rules
*nat

:POSTROUTING ACCEPT [0:0]
#portforwarding

#Forward trafiic from ens38 through ens33
-A POSTROUTING -s 10.0.0.0/24 -o ens33 -j MASQUERADE

COMMIT
```
Now you can verify that your client can access the internet.

[## Proxying server - SQUID]

- First download SQUID
```
sudo apt install squid
```
- Now before make a copy and edit ownership of the squid.conf file 
```
sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.original
sudo chmod a-w /etc/squid/squid.conf.original
```
- To configure the server as a transparent proxy server go to:
```
sudo nano /etc/squid/squid.conf
```
- You can opt to change the port of SQUID
```
http_port 8888
```
Note: Remember to add this port to your firewall rule.
- In the file find the cache_dir and uncomment. If you want it's possible to increase the cache values:
```
# Uncomment and adjust the following to add a disk cache directory.
cache_dir ufs /var/spool/squid 100 16 256
```
- Now you need to add the acl allowing access to your network. Can be use to deny access to certain websites as well. Search for acl and add yours after the acl rules:
```
#danielk acl network
acl danielknetwork src 10.0.0.0/24
```
- Adding a denied website:
```
#denied website
acl blacklist dstdomain .neverssl.com
```
- Now find the http_access to apply your rules:
```
http_access deny blacklist
http_access allow danielknetwork
```
Tip: Add the statment before the "http_access deny all"

- Save you file and now restart the service
```
sudo systemctl restart squid
```
Then check the status with:
```
sudo systemctl status squid
```
If everything is correct now you have a server with DNS, DCHP, Firewall rules, NAT and proxy for your clients.

Thannk you!


