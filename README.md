# linux_server
This readme file serves as a documentation file for the project "Linux Server" in the Cybersecurity program at BeCode.

## Project context 
The local library in your little town has no funding for Windows licenses so the director is considering Linux. Some users are sceptical and ask for a demo. The local IT company where you work is taking up the project and you are in charge of setting up a server and a workstation.
To demonstrate this setup, you will use virtual machines and an internal virtual network (your DHCP must not interfere with the LAN).

You may propose any additional functionality you consider interesting.

## Must Have

Set up the following Linux infrastructure:

1. One server (no GUI) running the following services:
    - DHCP (one scope serving the local internal network)  isc-dhcp-server
    - DNS (resolve internal resources, a redirector is used for external resources) bind
    - HTTP+ mariadb (internal website running GLPI)
    - **Required**
        1. Weekly backup the configuration files for each service into one single compressed archive
        2. The server is remotely manageable (SSH)
    - **Optional**
        1. Backups are placed on a partition located on  separate disk, this partition must be mounted for the backup, then unmounted

2. One workstation running a desktop environment and the following apps:
    - LibreOffice
    - Gimp
    - Mullvad browser
    - **Required** 
        1. This workstation uses automatic addressing
        2. The /home folder is located on a separate partition, same disk 
    - **Optional**
        1. Propose and implement a solution to remotely help a user

===========================================================================

The project setup includes a server and a workstation within a virtualized environment, carefully designed to ensure non-interference with the existing LAN configuration. 

I will create two virtual machines, one for the server and one for the client. The server will be running Ubuntu Server 20.04 LTS, and the client will be running Ubuntu Desktop 20.04 LTS.

The server operates headless, optimizing resource usage. The workstation, dressed with a GUI, hosts essential applications for daily library operations.

![alt text](assets/vm.png)

# Client Ubuntu VM:

I've created a new virtual machine with the following specifications:
- 2 GB of RAM
- 2 CPU cores
- 25 GB of storage
- Bridged network adapter
- Ubuntu 20.04 LTS Desktop Edition
- User: `library`
- Password: `2024kamkar`
- IP Address: `192.168.0.131`
- Hostname: `Ubuntu`


### Install Ubuntu

1. **Mount the Installation ISO**
2. **Install Ubuntu**:
    I've installed Ubuntu 20.04 LTS Desktop Edition. The installation process is straightforward, and you can follow the on-screen instructions to complete the installation.

    As for user creation, I've created a user named `library` with the password `2024kamkar`.

### Configuration

1. **Install Updates and SSH** 
   ```bash
   sudo apt update
   sudo apt upgrade
   sudo apt install openssh-server
   ```

2. **Enable SSH**:
   ```bash
   sudo systemctl enable ssh
   sudo systemctl start ssh
   ```

3. **Configure UFW Firewall**:
   ```bash
   sudo ufw allow ssh
   sudo ufw enable
   ```

4. **Install Additional Software**:
   - **LibreOffice** :
     ```bash
     sudo apt install libreoffice
     ```
   - **GIMP**:
     ```bash
     sudo apt install gimp
     ```
   - **Mullvad Browser** :
      I will install the Mullvad Browser on the Ubuntu Desktop VM.
        ```bash
        cd ~/Downloads
        wget https://mullvad.net/download/app/linux/latest/ -O mullvad-browser.tar.gz
        sudo tar -xzf mullvad-browser.tar.gz -C /opt
        cd /opt/mullvad-browser/
        sudo ln -s /opt/mullvad-browser/mullvad-browser /usr/local/bin/mullvad-browser
        mullvad-browser
        ```

 5. **Set Up Remote Assistance**:
   To set up remote assistance on a Linux client,we can set up `xrdp` , which uses the RDP protocol and can be accessed by most RDP clients, including Remmina, which is often used on Linux systems.

### Installing xrdp on the Client:

1. Install `xrdp`:

   ```bash
   sudo apt install xrdp
   ```

2. After installation, the `xrdp` service should start automatically. To check the status of the service:

   ```bash
   sudo systemctl status xrdp
   ```

   You should see that `xrdp` is active (running).
   Like so: 
   ![alt text](/assets/xrdp.png)

    To allow the RDP port through the firewall, run the following command:
    ```bash
    sudo ufw allow 3389/tcp
    ```
    We add an user to the `ssl-cert` group to allow the user to access the SSL certificate:
    ```bash
    sudo adduser library ssl-cert
    ```

    ![alt text](assets/adduserssh.png)
    To make sure all changes are applied, restart the xrdp service:

    ```bash
    sudo systemctl restart xrdp
    ```

### Using Remmina to Connect to the Client:

1. Install Remmina along with plugins for RDP and VNC, which are the most commonly used protocols:
    |
    ```bash
    sudo apt install remmina remmina-plugin-rdp remmina-plugin-vnc
    ```

    After installation, you can launch Remmina from your application menu or from the terminal:

    ```bash
    remmina
   ```

2. Configure Remmina to connect to the client:

-  Click the '+' icon to add a new connection.
-  Choose the RDP protocol from the drop-down menu.
-  In the 'Server' field, input the client machine's IP address `192.168.0.131`
-  Enter the username and password of the client machine:  `library` and `2024kamkar`
-  Save and Connect.
    
    ![alt text](assets/remmina.png)

    And Voila! We have successfully set up a Linux client with remote assistance enabled. The client is now accessible via RDP using Remmina or any other RDP client.

Final client setup:
    - **LibreOffice**
    - **GIMP**:
    - **Mullvad Browser**:
    - **Remote Assistance**:

![alt text](assets/client.png)


# Server Ubuntu VM:

I've created a new virtual machine with the following specifications:
- 2 GB of RAM
- 2 CPU cores
- 25 GB of storage
- Bridged network adapter
- Ubuntu 20.04 LTS Server Edition
- User: `latte` Password: `kamkar8955`
- IP Address: `192.168.0.248/24 `
- Hostname: `localibrary`

### Install Ubuntu

1. **Mount the Installation ISO**
2. **Install Ubuntu**:
    I've installed Ubuntu 20.04 LTS Server Edition. The installation process is straightforward, and you can follow the on-screen instructions to complete the installation.

    As for user creation, I've created a user named `latte` with the password `kamkar8955`.


## Configuration: 

### Configuring DHCP Server

1. **Install ISC DHCP Server**:
   ```bash
   sudo apt install isc-dhcp-server -y
   ```

2. **Configure DHCP Server**:
   - Edit the DHCP configuration file.
    ```bash
     sudo nano /etc/dhcp/dhcpd.conf
    ```
   - We add the following configuration to the file:
    ```bash
    subnet 192.168.0.0 netmask 255.255.255.0 {
        range 192.168.0.100 192.168.0.200;
        option domain-name-servers 192.168.0.1; 
        option routers 192.168.0.1; 
        option broadcast-address 192.168.0.255;
        default-lease-time 600;
        max-lease-time 7200;
    }
    ```
    ![alt text](assets/dhcpSubnets.png)

1. **Specify the Network Interface**:
   - Edit `/etc/default/isc-dhcp-server` to specify the interface DHCP listens on.
    ```bash
     sudo nano /etc/default/isc-dhcp-server
     ```
   - and once inside we set the `INTERFACESv4` variable to our server's network interface (aka `enp0s3`).

    ![alt text](assets/sv4.png)


2. **Start and Enable DHCP Service**:
   ```bash
   sudo systemctl restart isc-dhcp-server
   sudo systemctl enable isc-dhcp-server
   sudo systemctl status isc-dhcp-server
    ```
    Aaand Voila! We have successfully set up a DHCP server on our Ubuntu server! 

    ![alt text](assets/dhcpServer.png) 

    The DHCP server is now running and ready to assign IP addresses to clients on the network.
     
    Test the DHCP server by connecting a client to the network and verifying that it receives an IP address from the server.

    ![alt text](assets/dhcpTest.png)



### Setting Up a DNS Server with BIND

1. **Install BIND**:
   ```bash
   sudo apt install bind9 dnsutils -y
   ```

2. **Configure DNS Server**:
    - Edit the BIND configuration file.
     ```bash
      sudo nano /etc/bind/named.conf.options
     ```
    - Add the following configuration to the file:
     ```bash
     forwarders {
        8.8.8.8; - Google DNS
        8.8.4.4.; - Cloudflare DNS 
    };

    listen-on {any;};
    ```
  
   ![alt text](assets/forwarders.png)



1. **Start and Enable BIND**:
   ```bash
   sudo systemctl restart bind9
   sudo systemctl enable bind9
   ```

2. **Verify DNS Configuration**:
    - Check the status of the BIND service.
    ```bash
      sudo systemctl status bind9
      ```
    ![alt text](assets/DNS.png) 

    The DNS server is now running and ready to resolve domain names for clients on the network.

    Test the DNS server by configuring a client to use the server - and check if it can resolve domain names. 

    `nslookup google.com ` 
     
     aand Voila!

    ![alt text](assets/dbsTest.png)


### Setting Up HTTP and MariaDB for GLPI

1. **Install Apache and MariaDB**:
   ```bash
   sudo apt install apache2 mariadb-server
   ```

2. **Secure MariaDB Installation**:
   ```bash
   sudo mysql_secure_installation
   ```
    And here we follow the on-screen instructions to set a root password, remove anonymous users, disallow remote root login, and remove the test database.

    In this case scenario I set the password to `kamkar` for simplicity, we remove the anonymous users, remove the test database, allowed root remote login and reload privileges.
        ![alt text](assets/mysql.png) 

1. **Create a Database for GLPI**:
   Access MariaDB prompt:
   ```bash
   sudo mysql -u root -p
   ```
        
    ```sql
    CREATE DATABASE glpi;
    GRANT ALL PRIVILEGES ON glpi.* TO 'admin@localhost' IDENTIFIED BY 'kamkar';
    FLUSH PRIVILEGES;
    EXIT;
    ```
    ![alt text](assets/database.png)

### Install GLPI via Command Line

1. **Download GLPI**:
   ```bash
   wget -O /tmp/glpi.tgz https://github.com/glpi-project/glpi/releases/download/10.0.14/glpi-10.0.14.tgz
   ```

2. **Extract GLPI**:
   ```bash
   tar -xzf /tmp/glpi.tgz -C /var/www/html/
   ```

3. **Set Permissions**:
   ```bash
   chown -R www-data:www-data /var/www/html/glpi
   ```

4. **Run the Install Script**:
   ```bash
   php /var/www/html/glpi/bin/console db:install --db-host=localhost --db-name=glpi --db-user=admin --db-password='kamkar'
   ```


### Create Apache Virtual Host for GLPI:

1. Configure a new Virtual Host for GLPI:

    ```bash
    sudo nano /etc/apache2/sites-available/glpi.conf
    ```

    Add the following configuration:    

    ```apacheconf
    <VirtualHost *:80>
        ServerAdmin admin@localhost
        DocumentRoot /var/www/html/glpi
        ServerName 192.168.0.248/24

        <Directory /var/www/html/glpi>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
    ![alt text](assets/apacheVHglbi.png)

2. Enable the Virtual Host and Reload Apache:
    ```bash
    sudo a2ensite glpi.conf
    sudo systemctl reload apache2
    ```
    The GLPI application is now accessible via a web browser at `http://192.168.0.248`

    ![alt text](assets/apache.png)
    ![alt text](assets/glpi.png)
    
    We firs need to login with the default credentials `glpi` and `glpi` and then we can change the password to `kamkar` and `kamkar` for the admin account.

    ![alt text](assets/glpiLogged.png)
    ![alt text](assets/glpiAdmin.png)



### Setting Up SSH

1. **Install Updates and SSH** 
   ```bash
   sudo apt update
   sudo apt upgrade
   sudo apt install openssh-server
   ```

2. **Install SSH** 
   ```bash
   sudo apt install openssh-server -y
   ```
   Verify it's running:
   ```bash
   sudo systemctl status ssh
   ```

    ![alt text](assets/sshStatus.png)

    Aaand it works! We have successfully set up an Ubuntu server with SSH access. I've provided the IP address and login credentials for the SSH connection.

    `ssh latte@192.168.0.248` with the password `kamkar8955`

    ![alt text](assets/sshLogin.png) 

    and we can check on the server too:

    ![alt text](assets/sshWho.png)


### Configuring Firewall 

Let's make sure the firewall allows traffic on the necessary ports:
```bash
sudo ufw allow ...
sudo ufw enable
sudo ufw status
```
I use the TCP (Transmission Control Protocol) and I have the following ports open :

 - 22/tcp: Standard port for SSH (Secure Shell) - remote administration.
 - 2222/tcp: alternative SSH port.
 - 80/tcp: Standard port for HTTP (HyperText Transfer Protocol) unsecured web traffic.
 - 443/tcp: Standard port for HTTPS (HTTP Secure) - secured web traffic.
  
    ![alt text](assets/firewall.png) 

    The entries ending in (v6) represent the rules that apply to IPv6 traffic for the same ports.



### Backup Configuration Files

1.  Let's create our script in `/usr/local/bin/backup_glpi.sh`:

    ```bash
    sudo nano /usr/local/bin/backup_glpi.sh
    ```

    Inside the script, we have the following content:

    ```bash
    #!/bin/bash
    # Define backup paths
    BACKUP_DIR="/var/backups/glpi"
    GLPI_DIR="/var/www/html/glpi"
    DB_NAME="glpi"
    DB_USER="admin"
    DB_PASSWORD="kamkar" 
    DATE=$(date +%Y-%m-%d)

    # Create a new directory for today's backup
    mkdir -p "${BACKUP_DIR}/${DATE}"

    # Backup GLPI files
    tar -czf "${BACKUP_DIR}/${DATE}/glpi_files_${DATE}.tar.gz" -C "${GLPI_DIR}" .

    # Backup GLPI database
    mysqldump -u "${DB_USER}" -p"${DB_PASSWORD}" "${DB_NAME}" > "${BACKUP_DIR}/${DATE}/glpi_db_${DATE}.sql"

    # Remove backups older than 30 days
    find "${BACKUP_DIR}" -type f -mtime +30 -exec rm {} \;
    ```
  
    ![alt text](assets/backupScript.png)
    Make the script executable:

    ```bash
    sudo chmod +x /usr/local/bin/backup_glpi.sh
    ```
    We can schedule the backup script to run weekly using a cron job.

    ```bash
    sudo crontab -e
    ```
    Add this line to execute the script every Sunday at 2 AM:

    ```cron
        0 2 * * Sun /usr/local/bin/backup_glpi.sh
    ```
**Annd we save!**

The backup script will now run weekly, and we should see the backup files within `/var/backups/glpi` dated for each week.


### Install Fail2ban to Protect Against Brute Force Attacks

Fail2ban is a log-parsing application that monitors system logs for symptoms of an automated attack on your VPS. When an attempted compromise is located, using the defined parameters, Fail2ban will add a new rule to iptables to block the IP address of the attacker, either for a set amount of time or permanently.

1. Install Fail2ban:

    ```bash
    sudo apt install fail2ban
    ```
2. Configure Fail2ban:

    ```bash
    sudo cp /etc/fail2ban/jail.{conf,local}
    ```

    ```bash
    sudo nano /etc/fail2ban/jail.local
    ```

3. Start Fail2ban:

    ```bash
    sudo systemctl start fail2ban
    sudo systemctl enable fail2ban
    sudo systemctl status fail2ban
    ```
    ![alt text](assets/fail2ban.png)

    Fail2ban is now running and protecting the server from brute force attacks.

    To test Fail2ban, you can try to log in to the server with the wrong password multiple times. Fail2ban should block the IP address after a few failed attempts.

    ![alt text](assets/fail2banTest.png) 
