# Born2beroot


<img width="614" alt="スクリーンショット 2023-07-18 15 19 21" src="https://github.com/Justhiro55/Born2beroot/assets/77094170/cfe174de-27f8-43cb-9972-d1314fee1888">


1. setting up Debian
    1. installation of Debian 2.
    2. language and keyboard setup
    3. Host account setup (must be done in 42)
    4. password must be strong and follow the specified password policy
    Create a user with your login name as user name 6.
    Partition disks
        - Guided - use entire disk and set up encrypted LVM
        - SCSI3 ~"
        - Separate /home, /var, adn /tmp partitions"
        - [!]. Configure the package manager
            - deb.debian.org 2.
2. sudo setup 1.
    1. root login
        - **`su -`**
            - Command to switch to another user, environment variables and current directory remain the same.
        - **`su -`**
            - Switching to another user will also update the environment variables and current directory at the same time. It will be as if you started a new login session. 2.
    2. apt update
        - **`apt update -y`** (yes)
            - Updates the DB updating the package. The package itself will not be updated.
        - **`apt upgrade -y`**
            - (yes) update the installed packages to the latest version. 3.
    3. install sudo
        - **`apt install sudo`** 4.
    4. add a user to the sudo group
        - **`su -`**
        - **`usermod -aG sudo username`**
    Confirm the user belongs to the sudo group **`getent group sudo
        - **`getent group sudo`**
            - Command to get all user and group information at once.
    6. give a user the privilege to execute all commands.
        - **`visudo`**
            - Open the sudoers file
        - Add the following line to the sudoers file.
            - **`username ALL=(ALL:ALL) ALL`**
Setup of SSH Server
    1. install SSH (Omitted as it was already installed by package manager during setup)
    Confirm that SSH is installed and check the status of the SSH Server
        - **`sudo systemctl status ssh`**
            - If it is not active, make it active by the following command.
            - **`sudo systemctl enable ssh`**
            - **`sudo systemctl start ssh`**
    Setup SSH Server (continued) 4.
    Change default port from 22 to 4242
        - **`sudo vi /etc/ssh/sshd_config`**
            - sshd_config is the SSH server side configuration file.
        - **Change ``#Port 22`** to **`Port 4242`**.
        - Check if the port has been changed.
            - **`sudo grep Port /etc/ssh/sshd_config`** 5.
    5. restart SSH
        - **`sudo systemctl restart ssh`** 4.
4. ufw setup
    1. ufw (Uncomplicated Firewall): Simple firewall
        - Used to control external access. 2.
    2. ufw installation
        - **`sudo apt install ufw`** 3.
    3. activate ufw
        - **`sudo ufw enable`** 4.
    4. check ufw status
        - **`sudo ufw status`** 5.
    Grant access to port 4242
        - **`sudo ufw allow 4242`** 6.
    6. check configuration
        - **`sudo ufw status`** 5.
Connect to the SSH server
    1. configure the target VM in VirtualBox
        - Network > Advanced > Port Forwarding
        - Set host port and guest port to 4242. 2.
    2. restart the SSH server
        - **`sudo systemctl restart ssh`**
    3. check SSH status
        - **`sudo systemctl status ssh`**
    From the host, connect to the guest OS via SSH using a terminal
        - **`ssh username@127.0.0.1 -p 4242`**
            - Since 127.0.0.1 is a loopback address (an address that points to your terminal), you can also connect to localhost.
        - Verify that you can connect from root to the username user.
            - **You can exit with ``exit``**.
6. setting up password policy
    1. install password quality check library
        - **`sudo apt install libpam-pwquality`** 2.
    2. set passwords
        - **`sudo vi /etc/pam.d/common-password`**
        - **Add the following after ``password requisite pam_pwquality.so retry=3`**:''
            
            ````
            password requisite pam_pwquality.so retry=3 minlen=10 lcredit=-1 ucredit=-1 dcredit=-1 maxrepeat=2 usercheck=1 difok=7 enforce_for_root
            ````
            
            - minlen: minimum number of characters
            - lcredit: minimum number of lowercase alphabetical characters6. set password policy (continued)
            - ucredit: Minimum number of uppercase alphabetical characters (specified as a negative number)
            - dcredit: Minimum number of numeric characters (specified as a negative number)
            - maxrepeat: Maximum number of consecutive repetitions of the same character
            - usercheck: check if user is included (must be non-zero)
            - difok: Minimum number of characters that must be different from the old password
            - enforce_for_root: apply the same policy to root (if not set, it will not be applied to root, but no comparison check with the old password will be performed)
    3. set password expiration date
        - **`sudo vi /etc/login.defs`**
    
        
        ````
        PASS_MAX_DAYS 99999
        PASS_MIN_DAYS 0
        PASS_WARN_AGE 7
        PASS_WARN_AGE 7
        
    Change it as follows: ````` ````.
        
        ````
        PASS_MAX_DAYS 30
        PASS_MIN_DAYS 2
        PASS_WARN_AGE 7
        PASS_WARN_AGE 7
        
        PASS_MAX_DAYS: Maximum password expiration time in days
        
        PASS_MIN_DAYS: Minimum password change interval in days.
        
        PASS_WARN_AGE: Number of days to display a warning before the password expires.
        
        *These values are not applied to users who existed before the expiration date was set (such as your own user and root), so use the following command to apply them separately.
        
        ```bash
        $ sudo chage -m 2 -M 30 -W 7 your_username
        $ sudo chage -m 2 -M 30 -W 7 root
        ```
        
        Restart
        
        ````bash
        $ sudo reboot
        ```` 7.
        
7. setting up the sudoers file
    1. open the sodoers file
    
    ```bash $ sudo visudo
    $ sudo visudo
    ```
    
    1. find the following location
    
    ```
    Defaults secure_path="..."
    ```
    
    Add the following line in red: ```` 1.
    
    ````
    Defaults secure_path="..."
    Defaults passwd_tries=3
    Defaults badpass_message="Password is wrong!
    Defaults logfile="/var/log/sudo/sudo.log"
    Defaults iolog_dir="/var/log/sudo"
    Defaults log_input
    Defaults log_output
    Defaults requiretty
    Defaults requiretty
    
    > `passwd_tries`
    
    - Set to allow up to 3 password authentications (tries) when using sudo.
    
    > `badpass_message`.
    
    - Set error message on password failure.
    
    > `logfile`.
    
    - PATH to output logs to.
    - By default, output via syslog.
    
    > `iolog_dir`.
    
    - PATH to which logs obtained by `log_input`,`log_output` are output.
    - By default, "/var/log/sudo-io".
    
    > `log_input`,`log_output` 
    
    - Log all user input and all output sent to the screen.
    
    > `requiretty`.
    
    - sudo can only be used when a user logs into a tty
    - sudo cannot be run from cron, scripts, etc.
        - More secure. 8.
8. create monitoring.sh
    
    <aside
    Requirements
    The monitoring.sh script will display the following items every 10 minutes when the server starts
    
    OS configuration/kernel version
    Number of physical processors
    Number of virtual processors
    Currently available memory on the server and its utilization (in %)
    Current available disk space on the server and its utilization (in %)
    Current processor utilization (in %)
    Date and time of last reboot
    Whether LVM is active or not
    Number of active connections
    Number of users using the server
    IPv4 address of the server and its MAC address
    Number of commands executed by the sudo program
    
    </aside>
    
    Each item can be checked with the following files or commands
    
    - OS configuration and kernel version
        - The `uname -a` command
    - Number of physical processors
        - /proc/cpuinfo" file: "physical id
    - Number of virtual processors
        - Number of virtual processors "/proc/cpuinfo" file: "processor" Number of virtual processors
    - Currently available memory on the server and its utilization (in %)
        - `free` command: "Mem" line (total/used)
    - Currently available disks on the server and their utilization (in %)
        - `df -BM -T --total` command: "total" rows (1M-blocks/Used/Use%)
    - Current processor utilization (in %)
        - `top -bn1` command: "load average
    - Date and time of last restart.
        - `who -b` command
    - Whether LVM is active or not.
        - `lsblk` command: If "lvm" is specified, LVM is in use.
    - Number of active connections
        - `netstat` command: number of "ESTABLISHED" connections
    - Number of users using the server
        - `who` command: number of users (de-duplicated)
    - IPv4 address of the server and its MAC address
        - IPv4 address
            - The `hostname - I` command
        - MAC address
            - `ip a` command: "link/ether"
    - Number of commands executed by the sudo program
        - Number of "COMMAND" in the file "/var/log/sudo/sudo.log
    
    1. install net-tools (to use `netstat` command)
    
    Run ``bash
    sudo apt update
    sudo apt install net-tools
    1.
    
    1. create monitoring.sh file (/usr/local/bin/monitoring.sh)
    
    <aside>
    💡 #!/bin/bash
    
    arc="$(uname -a)"
    pcpu="$(cat /proc/cpuinfo | grep 'physical id' | wc -l)"
    vcpu="$(cat /proc/cpuinfo | grep 'processor' | wc -l)"
    uram="$(free -m | awk 'NR==2{printf "%s", $3}')"
    fram="$(free -m | awk 'NR==2{printf "%s", $2}')"
    pram="$(free -m | awk 'NR==2{printf "%.2f", $3*100/$2}')"
    udisk="$(df -BG --total | grep 'total' | awk '{print $4}')"
    fdisk="$(df -BG --total | grep 'total' | awk '{print $2}')"
    pdisk="$(df -BG --total | grep 'total' | awk '{printf "%.2f", $4/$2*100}')"
    cpul="$(top -bn1 | grep 'load' | awk '{printf "%.2f", $(NF-2)}')"
    lb="$(who -b | awk '{print $3" "$4" "$5}')"
    lvmu="$(lsblk | grep 'lvm' | awk '{if ($1) {print "yes";exit;} else {print "no"} }')"
    ctcp="$(netstat -an | grep 'ESTABLISHED' | wc -l)"
    ulog="$(who | cut -d ' ' -f 1 | sort -u | wc -l)"
    ip="$(hostname -I)"
    mac="$(ip a | grep 'link/ether' | awk '{print $2}')"
    cmds="$(grep -a 'COMMAND' /var/log/sudo/sudo.log | wc -l)"
    
    wall "	#Architecture: $arc
    #CPU physical: $pcpu
    #vCPU: $vcpu
    #Memory Usage: $uram/${fram}MB ($pram%)
    #Disk Usage: $udisk/${fdisk}GB ($pdisk%)
    #CPU load: $cpul
    #Last boot: $lb
    #LVM use: $lvmu
    #Connexions TCP: $ctcp ESTABLISHED
    #User log: $ulog
    #Network: IP $ip ($mac)
    #Sudo: $cmds cmd"
    
    <aside>
    💡 wall command
    Displays a message to all logged in users.
    
    </aside>
    
    </aside>

