# Gatewaydocumentation
Documentation on how to set DNS, DHCP, Firewall, NAT and Caching services on Ubuntu 20.04

### Note: This installation was done on VMWare, But the configuration files will work for any install.

## Description
This is a lab enviroment done in VMWare to deploy services on a ubuntu server.
 This server will provide DNS resolution for the clients, DHCP, Firewall (ufw) rules, NAT and Caching for clients.
On this eniroment there are two machines, the server and a client machine (in this case Ubuntu desktop). The objective install and configurer all of thi server on one server.

- There are two NIC on the server, ens38 with the ip addresess 192.168.0.15/24 
- ens33 with the ip address of 10.0.0.250/24

## Intalation of Server

## 1. Download
- Grab a copy of Ubuntu server 20.04 
- Use this link 
https://ubuntu.com/download/server

## 2. Install
- Install and set up an account to access ubuntu server
- Install SSH service right on the installation page
- After installition is done 

## 3. Network
- It is a good idea to set up a static ip address to be able to access though ssh.

To do this, use this command sudo nano /etc/netplan/00-installer-config.yaml
