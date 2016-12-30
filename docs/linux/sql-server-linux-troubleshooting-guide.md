---
# required metadata

title: Troubleshoot SQL Server on Linux - SQL Server vNext | Microsoft Docs
description: Provides troubleshooting tips for using SQL Server vNext on Linux.
author: annashres 
ms.author: anshrest 
manager: jhubbard
ms.date: 11/16/2016
ms.topic: article
ms.prod: sql-linux
ms.technology: database-engine
ms.assetid: 99636ee8-2ba6-4316-88e0-121988eebcf9S

# optional metadata

# keywords: ""
# ROBOTS: ""
# audience: ""
# ms.devlang: ""
# ms.reviewer: ""
# ms.suite: ""
# ms.tgt_pltfrm: ""
# ms.custom: ""
---
# Troubleshoot SQL Server on Linux

This document describes how to troubleshoot Microsoft SQL Server running on Linux or in a Docker container. When troubleshooting SQL Server on Linux, please make remember the limitations of this private preview release. You can find a list of these in the [Release Notes](sql-server-linux-release-notes.md).

## <a id="connection"></a> Troubleshoot connection failures
If you are having difficulty connecting to your Linux SQL Server, there are a few things to check. 

- Verify that the server name or IP address is reachable from your client machine.

   > [!TIP]
   > To find the IP address of your Ubuntu machine, you can run the ifconfig command as in the following example:
   >
   >   ```bash
   >   sudo ifconfig eth0 | grep 'inet addr'
   >   ```
   > For Red Hat, you can use the ip addr as in the following example:
   >
   >   ```bash
   >   sudo ip addr show eth0 | grep "inet"
   >   ```
   > One exception to this technique relates to Azure VMs. For Azure VMs, [find the public IP for the VM in the Azure portal](sql-server-linux-azure-virtual-machine.md#connect).

- If applicable, check that you have opened the SQL Server port (default 1433) on the firewall.

- For Azure VMs, check that you have a [network security group rule for the default SQL Server port](sql-server-linux-azure-virtual-machine.md#remote).

- Verify that the user name and password do not contain any typos or extra spaces or incorrect casing.

- Try to explicitly set the protocol and port number with the server name like the following: **tcp:servername,1433**.

- Network connectivity issues can also cause connection errors and timeouts. After verifying your connection information and network connectivity, try the connection again.

## Manage the SQL Server service

The following sections show how to start, stop, restart, and check the status of the SQL Server service. 

### Manage the mssql-server service in Red Hat Enterprise Linux (RHEL) and Ubuntu 

Check the status of the status of the SQL Server service using this command:

   ```bash
   sudo systemctl status mssql-server
   ```

You can stop, start, or restart the SQL Server service as needed using the following commands:

   ```bash
   sudo systemctl stop mssql-server
   sudo systemctl start mssql-server
   sudo systemctl restart mssql-server
   ```

### Manage the execution of the mssql Docker container

You can get the status and container ID of the latest created SQL Server Docker container by running the following command (The ID will be under the “CONTAINER ID” column):

   ```bash
   sudo docker ps -l
   ```
   
You can stop or restart the SQL Server service as needed using the following commands:
   
   ```bash
   sudo docker stop <container ID> 
   sudo docker restart <container ID> 
   ```

You can run a new container by using the following command:

   ```bash
   sudo docker run –e 'ACCEPT_EULA=Y' –e 'SA_PASSWORD=<YourStrong!Passw0rd>' -p 1433:1433 -d microsoft/mssql-server-linux 
   ```

## Execute commands in a Docker container

If you have a running Docker container, you can execute commands within the container from a host terminal.

To get the container ID run:
   
   ```bash
   sudo docker ps
   ```
To start a bash terminal in the container run:

   ```bash
   sudo docker exec -ti <container ID> /bin/bash
   ```

Now you can run commands as though you are running them at the terminal inside the container.

Example of how you could read the contents of the error log in the terminal window:

   ```bash
   sudo docker exec -ti d6b75213ef80 /bin/bash root@d6b75213ef80:/# cat /var/opt/mssql/log/errorlog
   ```

### Copy files from a Docker container

To copy a file out of the container you could do something like this:

   ```bash
   sudo docker cp <container ID>:<container path> <host path>
   ```

Example:

   ```bash
   sudo docker cp d6b75213ef80:/var/opt/mssql/log/errorlog /tmp/errorlog
   ```

To copy a file in to the container you could do something like this:
   
   ```bash
   sudo docker cp <host path> <container ID>:<container path>
   ```

Example:
   
   ```bash
   sudo docker cp /tmp/mydb.mdf d6b75213ef80:/var/opt/mssql/data
   ```

## Access the log files
   
The SQL Server engine logs to the /var/opt/mssql/log/errorlog file in both the Linux and Docker installations. You need to be in ‘superuser’ mode to browse this directory.

The installer logs here: /var/opt/mssql/setup-< time stamp representing time of install>
You can browse the errorlog files with any UTF-16 compatible tool like ‘vim’ or ‘cat’ like this: 

   ```bash
   sudo cat errorlog
   ```

If you prefer, you can also convert the files to UTF-8 to read them with ‘more’ or ‘less’ with the following command:
   
   ```bash
   sudo iconv –f UTF-16LE –t UTF-8 <errorlog> -o <output errorlog file>
   ```
## Extended events

Extended events can be queried via a SQL command.  More information about extended events can be found [here](https://technet.microsoft.com/en-us/library/bb630282.aspx):

## Crash dumps 

Look for dumps in the log directory in Linux. Check under the /var/opt/mssql/log directory for Linux Core dumps (.tar.gz2 extension) or SQL minidumps (.mdmp extension)

For Core dumps 
   ```bash
   sudo ls /var/opt/mssql/log | grep .tar.gz2 
   ```

For SQL dumps 
   ```bash
   sudo ls /var/opt/mssql/log | grep .mdmp 
   ```

## Common issues

1. You can not connect to your remote SQL Server instance.

   See the troubleshooting section of the topic, [Connect to SQL Server on Linux](#connection).

2. Port 1433 conflicts when using the Docker image and SQL Server on Linux simultaneously.

   When trying to run the SQL Server Docker image in parallel to SQL Server running on Ubuntu, check for the port number that it is running under. If it tries to run on the same port number, it will throw the following error: “failed to create endpoint <container name> on network bridge. Error starting proxy: listen tcp 0.0.0.0:1433 bind: address already in use.” This can also happen when running two Docker containers under the same port number.

3. ERROR: Hostname must be 15 characters or less.

   This is a known-issue that happens whenever the name of the machine that is trying to install the SQL Server Debian package is longer than 15 characters. There are currently no workarounds other than changing the name of the machine. One way to achieve this is by editing the hostname file and rebooting the machine. The following [website guide](http://www.cyberciti.biz/faq/ubuntu-change-hostname-command/) explains this in detail.

4. Resetting the system administration (SA) password.

   If you have forgotten the system administrator (SA) password or need to reset it for some other reason please follow these steps.

   > [!NOTE]
   > Following these steps will stop the SQL Server service temporarily.

   Log into the host terminal, run the following commands and follow the prompts to reset the SA password:

   ```bash
   sudo systemctl stop mssql-server.service
   sudo /opt/mssql/bin/sqlservr-setup
   sudo systemctl start mssql-server.service
   ```

5. Using special characters in password.

   If you use some characters in the SQL Server login password you may need to escape them when using them in the Linux terminal. You will need to escape the $ anytime using the backslash character you are using it in a terminal command/shell script:

   Does not work:

   ```bash
   sudo sqlcmd -S myserver -U sa -P Test$$
   ```

   Works:

   ```bash
   sqlcmd -S myserver -U sa -P Test\$\$
   ```

   Resources:
   [Special characters](http://tldp.org/LDP/abs/html/special-chars.html)
   [Escaping](http://tldp.org/LDP/abs/html/escapingsection.html)

## Support

Support is available through the community and monitored by the engineering team. For specific questions head to [Stack Overflow](http://stackoverflow.com/), discuss  on [reddit.com/r/sqlserver](http://www.reddit.com/r/sqlserver), and report bugs to [connect](http://connect.microsoft.com/).