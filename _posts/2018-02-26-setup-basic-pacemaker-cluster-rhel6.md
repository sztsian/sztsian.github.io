---
id: 192
title: Setup basic pacemaker cluster on RHEL 6
date: 2018-02-26T12:39:00+08:00
author: Zamir
layout: post
permalink: /setup-basic-pacemaker-cluster-rhel6/
categories:
  - Uncategorized
---
This is a note to setup a basic RHEL 6 cluster environment without any additional cluster resources. In this example I am using two VMs.

0.Configure /etc/hosts on both of the VMs, so that they can access each other by hostname. I added the following lines to /etc/hosts.
This should be done on all nodes.

> 192.168.122.111 rhel6-pcs-cluster-n1
> 192.168.122.112 rhel6-pcs-cluster-n2

1.Install related packages. This requires subscription for RHEL 6 High Availability Add On, or equvaliance if you are using community driven RHEL derivatives like CentOS.
This should be done on all nodes.

```bash
sudo yum install pacemaker pcs
```

2.Start related daemon
This should be done on all nodes.

```bash
sudo service pcsd start
sudo chkconfig pcsd on
sudo service corosync start
sudo chkconfig corosync on
```

3.Configure the password for cluster authentication. This should be the same for all nodes!

```bash
sudo passwd hacluster
```

4.Config firewall. In this case, this environment is for testing purpose, so I simply disabled iptables.
This should be done on all nodes.

```bash
sudo service iptables stop
sudo chkconfig iptables off
```

5.Configure cluster authentication. Execute the following on ONE NODE.
```bash
sudo pcs cluster auth rhel6-pcs-cluster-n1 rhel6-pcs-cluster-n2
```
It should ask you for username and password. Use the user hacluster and password you set in step 3.

After this, you should see something like the following

> rhel6-pcs-cluster-n1: Authorized

> rhel6-pcs-cluster-n2: Authorized

From now on, you can configure it on this machine (instead of doing it on all nodes!)

6.Create the cluster

```bash
sudo pcs cluster setup --name pcscluster rhel6-pcs-cluster-n1 rhel6-pcs-cluster-n2
```

7.Starting the cluster

```bash
sudo pcs cluster start --all
```

Then you can check cluster status in any of the nodes using

```bash
sudo pcs status
```

> Note: Each time you added a new node (or created the cluster), it will take sometine for all nodes to go online.

This is basically done.

References:
* https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/configuring_the_red_hat_high_availability_add-on_with_pacemaker/
* https://www.digitalocean.com/community/tutorials/how-to-set-up-an-apache-active-passive-cluster-using-pacemaker-on-centos-7

