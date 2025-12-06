Day 001 - Day 020

# Linux User Setup with Non-interactive Shell

##  Objective

Create a user with non-interactive shell for your organization on a specific server. This is essential for service accounts and automated processes that don't require interactive login capabilities.

## Prerequisites

- Access to a Linux server (CentOS/Ubuntu/RHEL)
- sudo privileges
- Basic understanding of Linux user management

## Technologies Used

- Linux user management commands
- SSH access
- System administration

## Steps

1. First, login into the app server using `SSH`:

    ```sh
    ssh user@app-server-ip or ssh user@server-name
    ```

    > It will ask for user password, enter the correct password.

2. After login into server, run the following command to create user with non-interactive shell

    ```sh
    sudo useradd -m -s /usr/sbin/nologin user-name
    ```

    `s`: for shell, here we are giving nologin shell

    `m`: for user home directory, It will create a directory with user-name under /home

3. Verify the result

    ```sh
    cat /etc/passwd
    ```

    It should give you a list of users where you will find your created user. It will look like this:
    `kareem:x:1003:1004::/home/kareem:/usr/sbin/nologin`

    Try to login using:

    ```sh
    sudo su user-name
    ```

    Output: `This account is currently not available.`

============================================================================================================================

# Temporary User Setup with Expiry Date

As part of the temporary assignment to the Nautilus project, a developer named yousuf requires access for a limited duration. To ensure smooth access management, a temporary user account with an expiry date is needed. Here's what you need to do:

> Create a user named `yousuf` on `App Server 1` in Stratos Datacenter. Set the expiry date to `2024-01-28`, ensuring the user is created in lowercase as per standard protocol.

## Steps

1. Follow the [Day 01](./001.md) to connect server and run the following command:

    ```sh
    sudo useradd -m -e 2024-01-28 yousuf
    ```

2. Verify

    ```sh
    cat /etc/passwd
    sudo su yousuf
    ```
	
============================================================================================================================
# Secure SSH Root Access

Disable all app server SSH root access.

## Steps

1. Login into each app server ([this way](./001.md))
2. Modify `sshd_config` and restart sshd `service`

    ```sh
    sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config
    sudo systemctl restart sshd
    ```

============================================================================================================================
# Script Execute Permissions

In a bid to automate backup processes, the `xFusionCorp Industries` sysadmin team has developed a new bash script named `xfusioncorp.sh`. While the script has been distributed to all necessary servers, it lacks executable permissions on `App Server 1` within the Stratos Datacenter.

Your task is to grant executable permissions to the `/tmp/xfusioncorp.sh` script on `App Server 1`. Additionally, ensure that all users have the capability to execute it.

## Steps

1. Connect to App server 1
2. Check the current file permission status:

    ```sh
    ls -la /tmp
    ```

    ```txt
    4 ---------- 1 root root   40 Jul 30 02:21 xfusioncorp.sh
    ```

3. Run the following command to update permissions:

    ```sh
    chmod 755 /tmp/xfusioncorp.sh
    ```

4. Verify the results:

    ```sh
    ls -la /tmp
    ```

    ```txt
    4 -rwxr-xr-x 1 root root   40 Jul 30 02:21 xfusioncorp.sh
    ```

============================================================================================================================
# Install and Configuration Selinux

Following a security audit, the xFusionCorp Industries security team has opted to enhance application and server security with SELinux. To initiate testing, the following requirements have been established for App server 2 in the Stratos Datacenter:

- Install the required SELinux packages.
- Permanently disable SELinux for the time being; it will be re-enabled after necessary configuration changes.
- No need to reboot the server, as a scheduled maintenance reboot is already planned for tonight.
- Disregard the current status of SELinux via the command line; the final status after the reboot should be disabled.

## Steps

1. Install selinux packages:

    ```sh
    sudo dnf install selinux-policy selinux-policy-targeted policycoreutils policycoreutils-python-utils
    ```

2. Modify file in `/etc/selinux/config:

    ```sh
    vi /etc/selinux/config
    ```

    add this line:

    ```nano
    SELINUX=disabled
    ```

============================================================================================================================
# Setup a Cron Job

The `Nautilus` system admins team has prepared scripts to automate several day-to-day tasks. They want them to be deployed on all app servers in `Stratos DC` on a set schedule. Before that they need to test similar functionality with a sample cron job. Therefore, perform the steps below:

- Install `cronie` package on all `Nautilus` app servers and start `crond` service.
- Add a cron `*/5 * * * * echo hello > /tmp/cron_text` for `root` user.

## Steps

1. Login into each server using ssh (check [day01](./001.md))
2. Install `cronie` package into centos:

    ```sh
    sudo yum install cronie -y
    ```

3. Start crond service

    ```sh
    sudo systemctl enable crond
    sudo systemctl start crond
    ```

4. Create cron schedule:

    ```sh
    sudo crontab -e
    */5 * * * * echo hello > /tmp/cron_text
    ```

5. Verify crontab:

    ```sh
    sudo crontab -l
    ```

    and wait 5 minutes to check cron_text in /tmp/

============================================================================================================================
# Linux SSH Automation

The system admins team of xFusionCorp Industries has set up some scripts on jump host that run on regular intervals and perform operations on all app servers in Stratos Datacenter. To make these scripts work properly we need to make sure the thor user on jump host has password-less SSH access to all app servers through their respective sudo users (i.e tony for app server 1). Based on the requirements, perform the following:

Set up a password-less authentication from user thor on jump host to all app servers through their respective sudo users.

## Steps

1. On jump server, run the following command:

    ```sh
    ssh-keygen -t rsa -b 2048
    ```

    It will generate an ssh pub key and private key. We are going to share the pub key to all the app server for respective users.

2. Login into each app server and run the following command:

    ```sh
    mkdir -p .ssh
    vi .ssh/authorized_keys
    ```

    copy id_rsa.pub key from jump host inside /home/thor/.ssh/ and paste it there

## Or

```sh
#!/bin/sh

ssh-copy-id user@host
```

============================================================================================================================

# Setup Ansible

During the weekly meeting, the Nautilus DevOps team discussed about the automation and configuration management solutions that they want to implement. While considering several options, the team has decided to go with Ansible for now due to its simple setup and minimal pre-requisites. The team wanted to start testing using Ansible, so they have decided to use jump host as an Ansible controller to test different kind of tasks on rest of the servers.

Install ansible version 4.8.0 on Jump host using pip3 only. Make sure Ansible binary is available globally on this system, i.e all users on this system are able to run Ansible commands.

## Steps

To install run the following command on the jump host server:

```sh
sudo pip3 install ansible==4.8.0
```

============================================================================================================================
# Debugging MariaDB Issues

There is a critical issue going on with the Nautilus application in Stratos DC. The production support team identified that the application is unable to connect to the database. After digging into the issue, the team found that mariadb service is down on the database server.

## Steps

We have to consider few things:

- files permission
- socket file
- data directory
- config issue

1. Login into Database Server:

    ```sh
    ssh peter@stdb01
    ```

2. look at the mariadb logs:

    ```sh
    tail -f /var/log/mariadb.log
    ```

    ```bash
    [root@stdb01 mariadb]# tail -f mariadb.log 
    2025-08-03  8:54:14 0 [Note] /usr/libexec/mariadbd (initiated by: unknown): Normal shutdown
    2025-08-03  8:54:14 0 [Note] Event Scheduler: Purging the queue. 0 events
    2025-08-03  8:54:14 0 [Note] InnoDB: FTS optimize thread exiting.
    2025-08-03  8:54:14 0 [Note] InnoDB: Starting shutdown...
    2025-08-03  8:54:14 0 [Note] InnoDB: Dumping buffer pool(s) to /var/lib/mysql/ib_buffer_pool
    2025-08-03  8:54:14 0 [Note] InnoDB: Buffer pool(s) dump completed at 250803  8:54:14
    2025-08-03  8:54:14 0 [Note] InnoDB: Removed temporary tablespace data file: "ibtmp1"
    2025-08-03  8:54:14 0 [Note] InnoDB: Shutdown completed; log sequence number 45091; transaction id 21
    2025-08-03  8:54:14 0 [Note] /usr/libexec/mariadbd: Shutdown complete
    ```

3. lets see the mariadb service status

    ```sh
    sudo systemctl status mariadb
    ```

    ```bash
    [peter@stdb01 ~]$ systemctl status mariadb
    ○ mariadb.service - MariaDB 10.5 database server
    Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: disabled)
    Active: inactive (dead) since Sun 2025-08-03 08:54:14 UTC; 5min ago
    Duration: 5.819s
    Docs: man:mariadbd(8)
    https://mariadb.com/kb/en/library/systemd/
    ...
    ...
    Status: "MariaDB server is down"
    ```

4. Lets up the mariadb service:

    ```sh
    sudo systemctl enable mariadb
    sudo systemctl start mariadb
    ```

5. Still failed to run, lets looking into config:

    ```sh
    cat /etc/my.cnf.d/mariadb.service
    ```

    datadir: `/var/lib/mysql/`

    actual location: `/var/lib/mysqld`

    so update config and or add `ib_buffer_pool`

6. Restart

    ```sh
    systemctl restart mariadb
    ```

============================================================================================================================
# Create a BASH Script

The production support team of `xFusionCorp Industries` is working on developing some bash scripts to automate different day to day tasks. One is to create a bash script for taking websites backup. They have a static website running on `App Server 3` in Stratos Datacenter, and they need to create a bash script named `beta_backup.sh` which should accomplish the following tasks. (Also remember to place the script under `/scripts` directory on `App Server 3`).

- Create a zip archive named `xfusioncorp_beta.zip` of `/var/www/html/beta` directory.
- Save the archive in `/backup/` on `App Server 3`. This is a temporary storage, as backups from this location will be clean on weekly basis. Therefore, we also need to save this backup archive on Nautilus Backup Server.
- Copy the created archive to Nautilus Backup Server server in `/backup/` location.
- Please make sure script won't ask for **password** while copying the archive file. Additionally, the respective server user (for example, tony in case of App Server 1) must be able to run it.

## Steps

1. Login into app server 3 and generate ssh-key:
2. Copy ssh-pub key to backup server. Follow [day 07](./007.md) to complete these two steps
3. Write the following script in `/scripts/beta_backup.sh`:

    ```sh
    #!/bin/sh

    zip -r /backup/xfusioncorp_beta.zip /var/www/html/beta
    scp /backup/xfusioncorp_beta.zip clint@stbkp01:/backup/
    ```

4. Give the execute permission:

    ```sh
    chmod +x /scripts/beta_backup.sh
    ```

============================================================================================================================
# Install and Setup Tomcat Server

The Nautilus application development team recently finished the beta version of one of their Java-based applications, which they are planning to deploy on one of the app servers in Stratos DC. After an internal team meeting, they have decided to use the tomcat application server. Based on the requirements mentioned below complete the task:

- Install tomcat server on `App Server 1`.
- Configure it to run on port `8086`.
- There is a `ROOT.war` file on Jump host at location `/tmp`.

Deploy it on this tomcat server and make sure the webpage works directly on base URL i.e `curl http://stapp01:8086`

## Steps

1. Install JVM

    ```sh
    sudo yum install java-1.8.0-openjdk-devel -y
    ```

2. Create `tomcat` user

    ```sh
    sudo groupadd tomcat
    sudo useradd -M -U -d /opt/tomcat -s /bin/nologin -g tomcat tomcat
    ```

3. Create `/opt/tomcat` directory

    ```sh
    sudo mkdir /opt/tomcat
    ```

4. Download tomcat and Extract

    ```sh
    wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.80/bin/apache-tomcat-9.0.80.tar.gz

    sudo tar -xf apache-tomcat-9.0.80.tar.gz -C /opt/tomcat --strip-components=1
    ```

5. Set Permissions

    ```sh
    sudo chown -R tomcat:tomcat /opt/tomcat
    ```

6. Create a systemd service

    ```sh
    sudo vi /etc/systemd/system/tomcat.service
    ```

    Copy and paste the following lines:

    ```text
    [Unit]
    Description=Apache Tomcat Web Application Container
    After=network.target

    [Service]
    Type=forking
    User=tomcat
    Group=tomcat
    Environment="JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.362.b09-4.el9.x86_64"
    Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
    Environment="CATALINA_HOME=/opt/tomcat"
    Environment="CATALINA_BASE=/opt/tomcat"
    ExecStart=/opt/tomcat/bin/startup.sh
    ExecStop=/opt/tomcat/bin/shutdown.sh
    RestartSec=10
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```

7. Start daemon service

    ```sh
    sudo systemctl daemon-reload
    sudo systemctl enable tomcat
    sudo systemctl start tomcat
    ```

8. Setup firewall, you can skip this step

    ```sh
    sudo firewall-cmd --permanent --zone=public --add-port=8080/tcp
    sudo firewall-cmd --reload
    ```

9. Test you can access using `curl http://stapp01:8080`. Now lets modify the port and deploy ROOT.war

10. To modify the port edit the `/opt/tomcat/conf/server.xml` and change port 8080 to 8086

    ```sh
    vi /opt/tomcat/conf/server.xml
    ```

    ```xml
     <Connector port="8086" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxParameterCount="1000"
               />
    ```

11. Take backup of `/opt/tomcat/webapps/ROOT`

    ```sh
    cd /opt/tomcat/webapps
    mv ROOT ROOT.bak
    ```

12. Copy `/tmp/ROOT.war` from jump host server to app server.

    ```sh
    scp /tmp/ROOT.war tony@stapp01:/home/tony/
    ```

13. Unzip the ROOT.war

    ```sh
    unzip /home/tony/ROOT.war -d /opt/tomcat/webapps/ROOT
    ```

14. Restart tomcat service

    ```sh
    systemctl restart tomcat
    ```

15. Test

    ```sh
    curl http://stapp01:8086
    ```

    ```html
    <!DOCTYPE html>
    <!--
    To change this license header, choose License Headers in Project Properties.
    To change this template file, choose Tools | Templates
    and open the template in the editor.
    -->
    <html>
        <head>
            <title>SampleWebApp</title>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
        </head>
        <body>
            <h2>Welcome to xFusionCorp Industries!</h2>
            <br>
        
        </body>
    </html>
    ```

============================================================================================================================
# Linux Network Services

Our monitoring tool has reported an issue in Stratos Datacenter. One of our app servers has an issue, as its Apache service is not reachable on port 3000 (which is the Apache port). The service itself could be down, the firewall could be at fault, or something else could be causing the issue.

- Use tools like telnet, netstat, etc. to find and fix the issue. Also make sure Apache is reachable from the jump host without compromising any security settings.

- Once fixed, you can test the same using command `curl http://stapp01:3000` command from jump host.

## Steps

1. Login into app server.
2. Check `httpd/apache/nginx` service status

    ```shell
    tony@stapp01 ~]$ sudo systemctl status httpd
    ● httpd.service - The Apache HTTP Server
    Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset
    : disabled)
    Active: failed (Result: exit-code) since Wed 2025-08-06 01:38:21 UT
    C; 13min ago
        Docs: man:httpd.service(8)
    Process: 491 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exit
    ed, status=1/FAILURE)
    Main PID: 491 (code=exited, status=1/FAILURE)
    Status: "Reading configuration..."

    Aug 06 01:38:21 stapp01.stratos.xfusioncorp.com httpd[491]: (98)Address already i
    n use: AH00072: make_sock: could not bind to address 0.0.0.0:3000
    Aug 06 01:38:21 stapp01.stratos.xfusioncorp.com httpd[491]: no listening sockets 
    available, shutting down
    top -
    ```

3. Lets check the network port status

    ```sh
    sudo netstat -tlnup
    ```

    ```txt
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 127.0.0.11:36025        0.0.0.0:*               LISTEN      -                   
    tcp        0      0 127.0.0.1:3000          0.0.0.0:*               LISTEN      430/sendmail: accep 
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      298/sshd            
    tcp6       0      0 :::22                   :::*                    LISTEN      298/sshd            
    udp        0      0 127.0.0.11:56145        0.0.0.0:*                           -                   
    ```

    > It's clearly visible that the '3000' port is already being used by `sendmail`

4. So Either we need to change port 3000 on sendmail or we can run httpd on different port. Since target is to run apache on 3000, we have to change sendmail port.

5. Changing sendmail port

    ```sh
    cd /etc/mail
    cp sendmail.mc sendmail.mc.bak
    vi sendmail.mc
    ```

    Find the following line and change port with some other value (i,e; `1234`):

    ```sh
    DAEMON_OPTIONS(`Port=3000,Addr=127.0.0.1, Name=MTA')dnl
    ```

    ```sh
    sudo systemctl restart sendmail
    ```

6. Now lets check port and servicec status

    ```sh
    sudo netstat -tlnup
    sudo systemctl status httpd sendmail
    ```

7. Test

    From app server:

    ```sh
    curl http://localhost:3000
    ```

    From jump host:

    ```sh
    curl http://stapp01:3000
    ```

8. Debugging and Decision
    - From `netstat` we can see port `3000` listening on all interfaces.
    - `ifconfig` we can see jump host and app server connected with route
    - if we use `telnet` we see its giving no route to host.
    So we should check the firewall.

9. Fixing firewall using `iptables`

    ```sh
    sudo iptables -L -n
    ```

    ```bash
    sudo iptables -L -n
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination         
    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
    ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
    REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination         
    REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination         
    # Warning: iptables-legacy tables present, use iptables-legacy to see them
    ```

    That `FORWARD` rule is blocking the connection.

    Run the following command:

    ```sh
    sudo iptables -I INPUT 4 -p tcp --dport 3000 -j ACCEPT
    ```

10. Finally it should work: `curl http://stapp01:3000`

============================================================================================================================
# IPtables Installation And Configuration

We have one of our websites up and running on our Nautilus infrastructure in Stratos DC. Our security team has raised a concern that right now Apache’s port i.e `6200` is open for all since there is no firewall installed on these hosts. So we have decided to add some security layer for these hosts and after discussions and recommendations we have come up with the following requirements:

1. Install `iptables` and all its dependencies on each app host.
2. Block incoming port `6200` on all apps for everyone except for `LBR` host.
3. Make sure the rules remain, even after system reboot.

## Steps

1. Login into each app server and run the following command:

    ```sh
    sudo yum install -y iptables iptables-services
    sudo iptables -A INPUT -p tcp --dport 6200 -s 172.16.238.14 -j ACCEPT
    sudo iptables -A INPUT -p tcp --dport 6200 -j REJECT
    sudo service iptables save
    ```

    - We have installed iptables packages
    - set accept rules for only lbr host on port: 5004
    - set reject rule incoming request to port: 5004 from anywhere
    - commit the changes for persistency on reboot

    > port can be changed according to your task

## Available Iptables Commands

```shell
iptables v1.8.10 (legacy)

Usage: iptables -[ACD] chain rule-specification [options]
       iptables -I chain [rulenum] rule-specification [options]
       iptables -R chain rulenum rule-specification [options]
       iptables -D chain rulenum [options]
       iptables -[LS] [chain [rulenum]] [options]
       iptables -[FZ] [chain] [options]
       iptables -[NX] chain
       iptables -E old-chain-name new-chain-name
       iptables -P chain target [options]
       iptables -h (print this help information)

Commands:
Either long or short options are allowed.
  --append  -A chain            Append to chain
  --check   -C chain            Check for the existence of a rule
  --delete  -D chain            Delete matching rule from chain
  --delete  -D chain rulenum
                                Delete rule rulenum (1 = first) from chain
  --insert  -I chain [rulenum]
                                Insert in chain as rulenum (default 1=first)
  --replace -R chain rulenum
                                Replace rule rulenum (1 = first) in chain
  --list    -L [chain [rulenum]]
                                List the rules in a chain or all chains
  --list-rules -S [chain [rulenum]]
                                Print the rules in a chain or all chains
  --flush   -F [chain]          Delete all rules in  chain or all chains
  --zero    -Z [chain [rulenum]]
                                Zero counters in chain or all chains
  --new     -N chain            Create a new user-defined chain
  --delete-chain
            -X [chain]          Delete a user-defined chain
  --policy  -P chain target
                                Change policy on chain to target
  --rename-chain
            -E old-chain new-chain
                                Change chain name, (moving any references)

Options:
    --ipv4      -4              Nothing (line is ignored by ip6tables-restore)
    --ipv6      -6              Error (line is ignored by iptables-restore)
[!] --protocol  -p proto        protocol: by number or name, eg. `tcp'
[!] --source    -s address[/mask][...]
                                source specification
[!] --destination -d address[/mask][...]
                                destination specification
[!] --in-interface -i input name[+]
                                network interface name ([+] for wildcard)
 --jump -j target
                                target for rule (may load target extension)
  --goto      -g chain
                               jump to chain with no return
  --match       -m match
                                extended match (may load extension)
  --numeric     -n              numeric output of addresses and ports
[!] --out-interface -o output name[+]
                                network interface name ([+] for wildcard)
  --table       -t table        table to manipulate (default: `filter')
  --verbose     -v              verbose mode
  --wait        -w [seconds]    maximum wait to acquire xtables lock before give up
  --line-numbers                print line numbers when listing
  --exact       -x              expand numbers (display exact values)
[!] --fragment  -f              match second or further fragments only
  --modprobe=<command>          try to insert modules using this command
  --set-counters -c PKTS BYTES  set the counter during insert/append
[!] --version   -V              print package version.
```

============================================================================================================================
# Linux Process Troubleshooting

The production support team of xFusionCorp Industries has deployed some of the latest monitoring tools to keep an eye on every service, application, etc. running on the systems. One of the monitoring systems reported about Apache service unavailability on one of the app servers in Stratos DC.

> Identify the faulty app host and fix the issue. Make sure Apache service is up and running on all app hosts. They might not have hosted any code yet on these servers, so you don’t need to worry if Apache isn’t serving any pages. Just make sure the service is up and running. Also, make sure Apache is running on port ***`3004`*** on all app servers.

## Steps

1. Login into App server
2. Check httpd status

    ```sh
    root@stapp01 ~]# systemctl status httpd.service
    ● httpd.service - The Apache HTTP Server
    Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
    Active: failed (Result: exit-code) since Fri 2025-08-08 05:04:33 UTC; 12s ago
        Docs: man:httpd(8)
            man:apachectl(8)
    Process: 1004 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=1/FAILURE)
    Process: 1003 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
    Main PID: 1003 (code=exited, status=1/FAILURE)

    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com systemd[1]: Starting The Apache HTTP Server...
    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com httpd[1003]: (98)Address already in use: AH0007...4
    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com httpd[1003]: no listening sockets available, sh...n
    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com httpd[1003]: AH00015: Unable to open logs
    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: main process exited,...E
    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com kill[1004]: kill: cannot find process ""
    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: control process exit...1
    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com systemd[1]: Failed to start The Apache HTTP Server.
    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com systemd[1]: Unit httpd.service entered failed s....
    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service failed.
    Hint: Some lines were ellipsized, use -l to show in full.
    ```

    I see two problems:

    ```shell
    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com httpd[1003]: (98)Address already in use: AH0007...4
    Aug 08 05:04:33 stapp01.stratos.xfusioncorp.com httpd[1003]: no listening sockets available, sh...n
    ```

3. Lets check the port status

    ```sh
    sudo netstat -tlnup
    ```

    ```shell
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/init              
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      443/sshd            
    tcp        0      0 127.0.0.11:35047        0.0.0.0:*               LISTEN      -                   
    tcp        0      0 127.0.0.1:3004          0.0.0.0:*               LISTEN      680/sendmail: accep 
    tcp6       0      0 :::111                  :::*                    LISTEN      424/rpcbind         
    tcp6       0      0 :::22                   :::*                    LISTEN      443/sshd            
    udp        0      0 0.0.0.0:111             0.0.0.0:*                           1/init              
    udp        0      0 0.0.0.0:1021            0.0.0.0:*                           424/rpcbind         
    udp        0      0 127.0.0.11:35464        0.0.0.0:*                           -                   
    udp6       0      0 :::111                  :::*                                424/rpcbind         
    udp6       0      0 :::1021                 :::*                                424/rpcbind         
    ```

    So `3004` port already being used by sendmail. We already fixed this port conflict in the [previous day](./012.md). Lets fix the issue using the same way describe there.

4. Verify the status:

    ```sh
    curl http://stapp01:3004 # from jump host
    ```

============================================================================================================================
# Setup SSL for NGINX

The system admins team of xFusionCorp Industries needs to deploy a new application on App Server 3 in Stratos Datacenter. They have some pre-requites to get ready that server for application deployment. Prepare the server as per requirements shared below:

1. Install and configure `nginx` on `App Server 3`.

2. On App Server 3 there is a self signed SSL certificate and key present at location `/tmp/nautilus.crt` and `/tmp/nautilus.key`. Move them to some appropriate location and deploy the same in Nginx.

3. Create an `index.html` file with content `Welcome!` under Nginx document root.

4. For final testing try to access the App Server 3 link (either hostname or IP) from jump host using curl command. For example `curl -Ik https://<app-server-ip>/`.

## Steps

1. Install nginx

    ```sh
    sudo yum install nginx -y
    ```

2. Change default `index.html` content:

    ```sh
    echo "Welcome!" | sudo tee /usr/share/nginx/html/index.html
    ```

3. Restart and check if nginx is accesible from jump host:

    ```sh
    sudo systemctl restart nginx
    curl http://stapp03
    ```

4. Copy ssl from `/tmp`:

    ```sh
    sudo mkdir -p /etc/certs
    sudo cp /tmp/nautilus.* /etc/certs
    ```

5. Configure SSL:

    ```sh
    sudo vi /etc/nginx/nginx.conf
    ```

    We have to be cautious here, otherwise nginx could be broken.

    We need to add this line inside `server:80` just after `server_name`.

    ```shell
    return 301 https://$host$request_uri;
    ```

    Then we have to uncomment `server:443` and add following lines:

    ```nginx
    ssl_certificate     /etc/certs/nautilus.crt;
    ssl_certificate_key /etc/certs/nautilus.key;
    ```

    If you need to update anything else do respectively. Before restart the `ngninx` server make sure you are testin it using:

    ```sh
    sudo nginx -t
    ```

    If it returns successful, you are ready to restart nginx:

    ```sh
    sudo systemctl restart nginx
    ```

6. Finally test

    ```
	curl -k https://stapp03
    ```

============================================================================================================================
# Install and Configure NGINX as Load Balancer

Day by day traffic is increasing on one of the websites managed by the Nautilus production support team. Therefore, the team has observed a degradation in website performance. Following discussions about this issue, the team has decided to deploy this application on a high availability stack i.e on Nautilus infra in Stratos DC. They started the migration last month and it is almost done, as only the LBR server configuration is pending. Configure LBR server as per the information given below:

- Install `nginx` on `LBR` server
- Configure load-balancing with the an http context making use of all App Servers. Ensure that you update only the main `Nginx` configuration file located at `/etc/nginx/nginx.conf`
- Make sure you do not update the apache port that is already defined in the apache configuration on all app servers, also make sure apache server is up and running on all app servers
- Once done, you can access the website using StaticApp button on the top bar

## Steps

1. Login into each app server and make sure httpd service is running. We have to find in which port they are running:

    ```sh
    sudo ss -tlnup
    ```

    ```shell
    Netid     State      Recv-Q     Send-Q         Local Address:Port            Peer Address:Port     Process                                                                                            
    udp       UNCONN     0          0                 127.0.0.11:45089                0.0.0.0:*                                                                                                           
    tcp       LISTEN     0          511                  0.0.0.0:5001                 0.0.0.0:*         users:(("httpd",pid=1690,fd=3),("httpd",pid=1689,fd=3),("httpd",pid=1688,fd=3),("httpd",pid=1680,fd=3))
    tcp       LISTEN     0          128                  0.0.0.0:22                   0.0.0.0:*         users:(("sshd",pid=1102,fd=3))                                                                    
    tcp       LISTEN     0          4096              127.0.0.11:42483                0.0.0.0:*                                                                                                           
    tcp       LISTEN     0          128                     [::]:22                      [::]:*         users:(("sshd",pid=1102,fd=4))                  
    ```

    > Apache service is running on port: `5001`

2. Login into lbr server and install nginx

    ```sh
    sudo yum install nginx -y
    sudo systemctl enable nginx
    sudo systemctl start nginx
    ```

3. Configure Load Balancer, lets modify `/etc/nginx/nginx.conf`:

    - First, lets add upstream servers. copy and paste following servers inside `http` section just before `server:80` in `/etc/nginx/nginx.conf` file:

        ```conf
            upstream stapp {
            server stapp01:5001;
            server stapp02:5001;
            server stapp03:5001;
        }
        ```

    - Then redirect call to these server using `proxy_pass`, copy paste following lines inside `server:80`:

        ```conf
        location / {
            proxy_pass http://stapp;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";

            proxy_connect_timeout 5s;
            proxy_read_timeout 60s;
        }
        ```

    - Done, lets check config is okay and restart nginx server:

        ```sh
        sudo nginx -t
        sudo systemctl restart nginx
        ```

## Full NGINX LBR Configuration

```conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}


http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    upstream stapp {
        server stapp01:5001;
        server stapp02:5001;
        server stapp03:5001;
    }

    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        #root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }

        location / {
            proxy_pass http://stapp;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";

            proxy_connect_timeout 5s;
            proxy_read_timeout 60s;
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
```

============================================================================================================================
# Install and Configure PostgreSQL

The Nautilus application development team has shared that they are planning to deploy one newly developed application on Nautilus infra in Stratos DC. The application uses PostgreSQL database, so as a pre-requisite we need to set up PostgreSQL database server as per requirements shared below:

PostgreSQL database server is already installed on the Nautilus database server.

- Create a database user `kodekloud_aim` and set its password to `your-password`.
- Create a database `kodekloud_db6` and grant full permissions to user `kodekloud_aim` on this database.

> Please do not try to restart PostgreSQL server service.

## Steps

1. Login into Database server

    ```sh
    ssh user@db_host
    ```

2. Switch to postgres user and run following commands:

    ```sh
    sudo -i -u postgres
    psql -c "CREATE DATABASE kodekloud_db6;"
    psql -c "CREATE ROLE kodekloud_aim LOGIN PASSWORD 'your-password';"
    psql -c "GRANT ALL PRIVILEGES ON DATABASE kodekloud_db6 TO kodekloud_aim;"
    psql -c "ALTER DATABASE kodekloud_db6 OWNER TO kodekloud_aim;"
    ```

    > This is based on terminal, you don't have to login inside pgsql
    - It will create database
    - Create user and grant all privileges for the database

3. Alternatively, login inside the postgres query and run the following commands:

    ```sh
    sudo psql -U postgres
    ```

    ```SQL
    CREATE DATABASE kodekloud_db6;
    CREATE ROLE kodekloud_aim LOGIN PASSWORD 'your-password';
    GRANT ALL PRIVILEGES ON DATABASE kodekloud_db6 TO kodekloud_aim;
    ALTER DATABASE kodekloud_db6 OWNER TO kodekloud_aim;
    ```

============================================================================================================================
# Configure LAMP Server (LAMP Stack)

xFusionCorp Industries is planning to host a WordPress website on their infra in Stratos Datacenter. They have already done infrastructure configuration—for example, on the storage server they already have a shared directory `/vaw/www/html` that is mounted on each app host under `/var/www/html` directory. Please perform the following steps to accomplish the task:

- Install `httpd`,`php` and its dependencies on all app hosts.
- Apache should serve on port `3003` within the apps.
- Install/Configure `MariaDB` server on DB Server.
- Create a database named `kodekloud_db2` and create a database user named `kodekloud_cap` identified as password `your-pass`. Further make sure this newly created user is able to perform all operation on the database you created.
- Finally you should be able to access the website on LBR link, by clicking on the App button on the top bar. You should see a message like App is able to connect to the database using user kodekloud_cap

## LAMP Stack

LAMP Stack is a group of open-source software that is typically installed together to enable a server to host dynamic web apps. Full form of LAMP:

- **L**: Linux Operating Systems
- **A**: Apache Web Server
- **M**: MySQL/MariaDB Database
- **P**: Php

## Steps

### Configure App Server

1. Install `httpd`, `php` and `php-mysqli`:

    ```sh
    sudo yum install -y httpd php php-mysqli
    ```

2. Change Apache port and Restart service

    ```sh
    sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak
    sudo sed -i 's/\<80\>/3003/g' /etc/httpd/conf/httpd.conf
    sudo systemctl enable --now httpd
    ```

    > Always make sure you kept a backup of your file before modifying it.

> Follow these two steps for all app server

### Configure Database Server

1. Install mariadb-server on Database server

    ```sh
    sudo yum install -y mariadb-server
    sudo systemctl enable --now mariadb
    sudo systemctl status mariadb | grep "running"
    ```

2. Create User and Database

    ```sh
    mysql -u root -e "CREATE DATABASE kodekloud_db2;"
    mysql -u root -e "CREATE USER 'kodekloud_cap'@'%' IDENTIFIED BY 'your-pass';"
    mysql -u root -e "GRANT ALL ON kodekloud_db2.* TO 'kodekloud_cap'@'%';"
    mysql -u root -e "FLUSH PRIVILEGES;"
    ```

    > `'%'` This allows remote connection otherwise  user will be failed to connect DB.

### Verify Result

Click on `App` button to see the result. It should print the DB user.

## MYSQL Cheat Sheet

> Help with SQL commands to interact with a MySQL database

### MySQL Locations

- Mac:             */usr/local/mysql/bin*
- Windows:         */Program Files/MySQL/MySQL version/bin*
- Xampp:           */xampp/mysql/bin*

### Add mysql to your PATH

```bash
# Current Session
export PATH=${PATH}:/usr/local/mysql/bin
# Permanantly
echo 'export PATH="/usr/local/mysql/bin:$PATH"' >> ~/.bash_profile

============================================================================================================================
# Install and Configure Web Application

xFusionCorp Industries is planning to host two static websites on their infra in Stratos Datacenter. The development of these websites is still in-progress, but we want to get the servers ready. Please perform the following steps to accomplish the task:

- Install `httpd` package and dependencies on `app server 3`.
- `Apache` should serve on port `6400`.
- There are two website's backups `/home/thor/official` and `/home/thor/games` on `jump_host`. Set them up on `Apache` in a way that official should work on the link `http://localhost:6400/official/` and games should work on link `http://localhost:6400/games/` on the mentioned app server.
- Once configured you should be able to access the website using curl command on the respective app server, i.e `curl http://localhost:6400/official/` and `curl http://localhost:6400/games/`

## Steps

1. Login into App Server 3 and Install httpd

    ```sh
    sudo yum install -y httpd
    ```

2. Change Apache port:

    ```sh
    sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak
    sudo sed -i 's/80/6400/g' /etc/httpd/conf/httpd.conf
    ```

3. Restart Apache Server

    ```sh
    sudo systemctl restart httpd
    ```

4. Copy backup from Jump Host to App Server

    ```sh
    scp -r /home/thor/official banner@stapp03:/home/banner
    scp -r /home/thor/games banner@stapp03:/home/banner/
    ```

5. Place websites

    ```sh
    sudo cp -r /home/banner/official /var/www/html/
    sudo cp -r /home/banner/games /var/www/html
    ```

6. Restart `httpd`

7. Verify result

    ```sh
    curl http://localhost:6400/games/
    curl http://localhost:6400/official/
    ```

    ```html
    <!DOCTYPE html>
    <html>
    <body>

    <h1>KodeKloud</h1>

    <p>This is a sample page for our official website</p>

    </body>
    </html>
    ```

============================================================================================================================
# Configure Nginx + PHP-FPM Using Unix Sock

The Nautilus application development team is planning to launch a new PHP-based application, which they want to deploy on Nautilus infra in Stratos DC. The development team had a meeting with the production support team and they have shared some requirements regarding the infrastructure. Below are the requirements they shared:

- Install `nginx` on `app server 1` , configure it to use port `8093` and its document `root` should be `/var/www/html`.
- Install `php-fpm` version `8.2` on `app server 1`, it must use the unix socket `/var/run/php-fpm/default.sock` (create the parent directories if don't exist).
- Configure `php-fpm` and `nginx` to work together.
- Once configured correctly, you can test the website using `curl http://stapp01:8093/index.php` command from jump host.

## Steps

1. Login into App Server and run the following commands:

    ```sh
    sudo dnf update -y
    sudo dnf install nginx -y
    sudo dnf module install php:8.2 -y # change version here if requires
    ```

    - It will update packge repo
    - Install nginx and php with expected version

2. Configure php-fpm config:

    ```sh
    sudo mkdir -p /var/run/php-fpm
    sudo vi /etc/php-fpm.d/www.conf
    ```

    - `listen = /run/php-fpm/www.sock` update this line with expected directory. It should be `listen = /var/run/php-fpm/default.sock`

3. Configure nginx

    ```sh
    sudo vi /etc/nginx/nginx.conf
    ```

    - change port 80 to `8093`

4. Configure php with nginx

    ```sh
    sudo vi /etc/nginx/default.d/php.conf
    ```

    - Update `fastcgi_pass php-fpm;` to this: `fastcgi_pass unix:/var/run/php-fpm/default.sock;`

5. Restart php-fpm and nginx

    ```sh
    sudo systemctl enable --now nginx
    sudo systemctl enable --now php-fpm
    ```

6. Test: `curl http://stapp01:8093/index.php`

