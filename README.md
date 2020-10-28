# Gatewaydocumentation
Documentation on how to set DNS, DHCP, Firewall, NAT and Caching services on Ubuntu 20.04
Note: This installation was done on VMWare, But the configuration files will work for any install.

## Description
This is a lab enviroment done in VMWare to deploy services on a ubuntu server.
 This server will provide DNS resolution for the clients, DHCP, Firewall (ufw) rules, NAT and Caching for clients.
On this environment there are two machines, the server and a client machine (in this case Ubuntu desktop). The objective install and configurer all of this services in one server.

- There are two NIC on the server, ens38 with the ip addresess 192.168.0.15/24 (External Network)
- ens33 with the ip address of 10.0.0.250/24 (Internal Network)

## Requirements

-Ubuntu Server 20.04
-Ubuntu Desktop 20.04

-The server needs to have two NIC cards attached.

### Intalation of Server

### 1. Download
- Grab a copy of Ubuntu server 20.04 
- Use this link 
https://ubuntu.com/download/server

### 2. Install on VMWare
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

### 3. Network
- It is a good idea to set up a static ip address to be able to access the server though ssh.
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

To install DNS on the server you can type the following command:

```
sudo apt install bind9 dnsutils
```
#### To check the status of the service
```
sudo systemctl status named
```
#### To activate and deactivate the service
```
sudo systemctl start named
sudo systemctl stop named
```
Go to the config files to 





