## Configure data flow: Syslog Server --> Intermediate Forwarder --> Indexer


i.Go to your AWS console Create a 3 new instance (syslog-ng, intermediate forwarder, indexing).

ii.Enable the Security groups in your each servers (5514 & 8000-9999) ports

iii. Install syslog-ng (syslog-ng server) and install heavy forwarder on forwarder server & finally install splunk on indexer server.

## 1.Setting up syslog-ng server in redhat linux

**Install Syslog-ng on Linux (RHEL-9)**

Login as root user
```bash
sudo su
```
```bash
subscription-manager repos --enable codeready-builder-for-rhel-9-noarch-rpms
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```
Navigate to the below mentioned directory
```bash
cd /etc/yum.repos.d/
```
Install wget (Package installer)
```bash
yum install wget
```
```bash
wget https://copr.fedorainfracloud.org/coprs/czanik/syslog-ng336/repo/epel-8/czanik-syslog-ng41-epel-8.repo
```
```bash
yum install syslog-ng --nobest
```
Enable and start Syslog
```bash
systemctl enable syslog-ng
```
```bash
systemctl start syslog-ng
```
Check the status of the syslog using the below command
```bash
systemctl status syslog-ng
```
Exit
```bash
exit
```

Splunk installation (follow the documentation for both the intermediate forwarder and the Splunk instance)

https://github.com/URahuman/Splunk-System-admin-training/blob/main/Splunk_enterprise_instalation_linux_9.2.2.md

## 2.Configure the Syslog-ng server to send data to an intermediate sender.

Login as root user using below command
```bash
sudo su
```
Enable syslog
```bash
systemctl enable syslog-ng
```
Go to the directory mentioned below
```bash
cd /etc/syslog-ng
```
List the files to check if syslog-ng.conf is available
```bash
ls
```
Edit the syslog-ng.conf file
```bash
vi syslog-ng.conf
```
Add the below listed lines to receive the data from syslog-ng and forward them to the intermediate forwarder. 

This is configuration for Syslog-ng server, In destination stanza give the ip address of Forwarder server.
```bash
source s_network{
        udp();
};
destination d_from_net{
file("/var/log/from_net");};

destination splunk_tcp {
network("Intermediate Forwarder IP Address" transport("tcp") port(5514));
};

log{
source(s_network);
destination(d_from_net);
};

log {
source(s_network);
destination(splunk_tcp);
};
```
Restart syslog-ng service
```bash
systemctl restart syslog-ng
```
## 3.Configure to the intermediate forwarder

i. Login to the forwarder with your credentials.

ii.Go to Intermediate Forwarder server  **“Settings” >> “Forwarding & receiving” >> Click on “Configure Forwarding” >> “New Forwarding Host” >> Add your “ indexer Host  ip : 9997 ” >> “Save”**

## 3.Configure to the Indexer (splunk)

i.Login to the indexer with your credentials.

ii. Go to Splunk indexer server  **“Settings” >> “Forwarding & receiving” >> Click on “Configure Receiving” >> “New Receiving Port” >> Add “ 9997 ” port >> “Save”**

iii.Finally check if data are sending or not  **Go to “search bar” >> Enter “index=main”**
