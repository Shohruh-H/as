*This project has been created as part of the 42 curriculum by shkholma*

# Born2beroot

## Description

This project involves setting up a configured Linux server on a virtual machine following certain system administration guidelines. For this setup, **Debian** was selected as the operating system since it seems easier to handle.

### Choice of Operating System: Debian vs. Rocky Linux

#### Debian
* **Advantages:** Exceptionally stable, light resource footprint, vast package repository (`apt`), wide community support, straightforward security management via AppArmor.
* **Disadvantages:** Uses older software versions in stable releases to prioritize reliability over cutting-edge features.

#### Rocky Linux (according to AI)
* **Advantages:** Enterprise-grade reliability (downstream binary-compatible with Red Hat Enterprise Linux), strong focus on enterprise security defaults (SELinux enabled by default).
* **Disadvantages:** Steeper learning curve, stricter configuration requirements, slightly heavier system resource consumption.


---

### Comparisons (according to internet)

#### 1. Debian vs. Rocky Linux
* **Package Management:** Debian uses `apt` / `.deb`; Rocky uses `dnf` / `.rpm`.

* **Target Environment:** Debian is ideal for general-purpose servers and lightweight environments; Rocky Linux targets enterprise enterprise environments aligned with RHEL standards.

* **Security Framework:** Debian defaults to **AppArmor**; Rocky Linux defaults to **SELinux**.

#### 2. AppArmor vs. SELinux
* **AppArmor (Debian):** Path-based Mandatory Access Control (MAC). Easier to learn, write, and manage profiles based on file paths.

* **SELinux (Rocky Linux):** Label-based MAC. Highly granular and extremely secure, but carries a significantly steeper configuration learning curve.

#### 3. UFW vs. firewalld
* **UFW (Uncomplicated Firewall):** A user-friendly frontend for `iptables`/`nftables` common in Debian/Ubuntu. Simple syntax for host-based firewall rules.

* **firewalld:** Dynamic firewall manager common in RHEL/Rocky Linux using zone-based configurations without restarting firewall connections when rules change.

#### 4. VirtualBox vs. UTM
* **VirtualBox:** Cross-platform type-2 hypervisor with wide compatibility, standard for x86 architecture setups.

* **UTM:** macOS-optimized virtualization tool leveraging Apple's Hypervisor framework (QEMU backend), offering superior performance and native execution on Apple Silicon (M1/M2/M3) ARM architecture.



---

### Partitioning:

Configured using LVM (Logical Volume Manager) with standard secure partitioning scheme (`boot`, `/` (root), `/home`, `/swap`, `/srv`, `/var`, `/tmp`, `/var/log`, etc.) to isolate system directories and prevent partition overflow attacks.
Actually, each partition size based on Linux administration practices (also according to bonus part of the project), system stability, and standard directory usage:

  * `boot` -- ca. 500 MB: primary volume partition, which contains the files for booting the system

  * `/` (Root) -- ca. 10 GB: the root directory holds the core operating system files, binaries (/bin, /sbin), libraries (/lib), and system configurations (/etc). While a base Linux install takes less, 10 GB provides a safe safety buffer for OS packages, system updates, temporary package cache (apt or dnf), and minor application installs over time without risking a disk-full crash.

  * `/home` -- ca. 5 GB: this partition isolates user personal files, documents, and user-specific application configurations (~/.config). In a standard administrative or test setup, 5 GB is plenty of room for normal user documents and configuration files while preventing a single user from filling up the entire drive and crashing the system.

  * `/swap` -- ca. 2.3 GB: swap acts as an overflow for RAM and is essential for system stability if physical memory gets exhausted. A common guideline for systems with moderate RAM is to match or slightly exceed RAM size (e.g., supporting systems with around 2 GB of RAM), and 2.3 GB provides a precise, reliable amount for hibernation and memory paging without wasting excessive disk space.


  * each `/var`, `/srv`, `/tmp` -- ca. 3 GB (shared or pooled space): these directories handle transient and service-oriented data and 3 GB offers a balanced space for temporary working files and lightweight services without over-allocating on a constrained disk:

    + `/var` holds variable data like spool directories and databases.
    + `/srv` contains site-specific data served by the system (like web or FTP files).
    + `/tmp` holds temporary files created by applications and users.

  * `/var/log` -- ca. 4 GB: isolating /var/log into its own dedicated partition is a critical security and stability practice. System daemons, the kernel, authentication logs (auth.log), and service errors write continuously here. Giving it a dedicated 4 GB ensures that a sudden log explosion (e.g., a runaway service spamming debug logs or a brute-force attack attempt) fills only this partition, preventing logs from consuming root space and crashing the entire operating system.

### Security Policies and user management:

  * **Strict password policies**:
    + Password expires every 30 days.
    + The minimum number of days between password changes is 2 (days).
    + The user receives a warning message 7 days before his/her password expires.
    + Password must be at least 10 characters long. It must contain an uppercase
letter, a lowercase letter, and a number. Also, it must not contain more than 3
consecutive identical characters.
    + Password must not include the name of the user.
    + Password must contain at least 7 characters that were not part of the previous password (not applicable to root).
    + Root password has to comply with this policy.

    These policies are enforced via `PAM` (see the file */etc/security/pam.d/common-password*, `LOGIN` (see the file */etc/login.defs*) and the library `libpam-pwquality` (see the file */etc/security/pwquality.conf*). One should also use the command
    > `chage -l username`

    to see the password properties/characteristics,
    while
    > `chage -M 30 -m 2 -W 7 username`

    makes the password details exactly.


  * **Sudo** strictly limited to authorized users with logged actions and explicit security rules (limited retries, custom TTY-teletypewriter restrictions):
    + Authentication using sudo is limited to 3 attempts in the event of an incorrect password.
    + A (custom) message is displayed if an incorrect password is
entered when using sudo.
    + Each action performed with sudo is logged, including both inputs and
outputs. The log files are saved in /var/log/sudo/ .
    + The TTY mode is enabled for security reasons.
    + For security reasons, the paths that can be used by sudo are also restricted to
*/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin*

    See these rules typing
    > `sudo visudo`

  * **SSH** restricted to non-standard unique port **4242**, and root login disabled. See the file */etc/ssh/sshd_config*


  * **UFW** (uncomplicated firewall) is installed to secure the network traffic. It is configured as the primary host-based firewall and blocks all incoming traffic (`deny (incoming)`), allow all outgoing traffic (`allow (outgoing)`).

  * **User Management:** Hostname is chosen as *shkholma42*.

### Website in the bonus part

* We set up a functional WordPress website with the following services lighttpd, MariaDB, and PHP (excluding NGINX / Apache2). Required for the WordPress site:

  + lighttpd (web server): answers browser requests on port 80. It’s a good fit for a small VM because it is lightweight.

  + PHP 8.4 with PHP-FPM: WordPress is written in PHP, so something has to run the code. PHP-FPM is a separate process pool that lighttpd hands PHP requests to over FastCGI. This keeps the web server light.

  + MariaDB (database): WordPress stores posts, users, settings, and the site address in a database. MariaDB is the standard open-source MySQL-compatible choice on Debian.

  + WordPress installation: the actual website.

* Tooling

  + WP-CLI: the command-line tool for WordPress. It makes installing, configuring, and managing WordPress (and enabling the Redis plugin) scriptable without clicking through a browser. It isn’t running as a service, it just helps to manage the site.

  + Redis server: an in-memory key-value store used here as a persistent object cache. WordPress runs many database queries per page, and Redis keeps the results in RAM, so repeated requests are answered without hitting MariaDB. That reduces database load and page time, which is a good justification for a VM with little CPU and memory.

  + PHP Redis extension: the library that lets PHP talk to the Redis server.

  + WordPress Redis Object Cache plugin: connects WordPress to Redis.


# Review checklist:

| Component | Status |
| :--- | :--- |
| lighttpd web server | ✅ Running |
| PHP 8.4 with PHP-FPM | ✅ Running |
| MariaDB database | ✅ Running |
| WordPress installation | ✅ Complete |
| WP-CLI | ✅ Installed |
| Redis server | ✅ Running |
| PHP Redis extension | ✅ Enabled |
| WordPress Redis Object Cache | ✅ Connected |
| Redis listening locally on 127.0.0.1:6379 | ✅ Confirmed |
| UFW incoming rules | ✅ Only port 4242 allowed |
| WordPress HTTP response | ✅ 200 OK |


### System monitoring

At every 10 minutes a bash script called `monitoring.sh` located in */usr/local/bin/* appears in all open terminals. It shows the following information:

* The architecture of your operating system and its kernel version.

* The number of physical processors.

* The number of virtual processors.

* The currently available RAM on your server and its utilization rate as a percentage.

* The currently available storage on your server and its utilization rate as a percent-
age.

* The current CPU utilization rate as a percentage.

* The date and time of the last reboot.

* Whether LVM is active or not.

* The number of active connections.

* The number of users using the server.

* The IPv4 address of your server and its MAC (Media Access Control) address.

* The number of commands executed with the sudo program.

A typical output looks like:
> ```bash
> #Architecture: Linux shkholma42 6.12.113+deb13-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.12.113-1 (2026-10-15) x86_64 GNU/Linux
> #Physical CPU: 1
> #vCPU: 1
> #Memory Usage: 478/1973MB (24.23%)
> #Disk Usage: 1.7/27.1GB (6.43%)
> #CPU load: 0.0
> #Last boot: 2026-10-23 02:52
> #LVM use: yes
> #TCP Connections: 1 ESTABLISHED
> #User log: 3
> #Network: IP 10.0.2.15 (08:00:27:b8:db:68)
> #Sudo: 47 cmd
> ```



## Instructions

* **Strict password policies**: one can always change passord using
    > `passwd`

    for the current (active) user and
    > `sudo passwd username`

    for the user with `username`. If there is no username aor username=root, then this changes the password of the `root`.


* **SSH**: to connect the server from other computers, you need:
    > VM Settings → Network → Adapter 1 → Advanced → Port Forwarding

    Click the button/green + and fill in:
    >*Name:* ssh

    >*Protocol:* TCP

    >*Host IP:* 127.0.0.1 (or other IP up to your choice)

    >*Host Port:* 4242

    >*Guest IP:* 10.0.2.15 (your server's ip adress)

    >*Guest Port:* 4242

    Similarly, you can add extra ports for other services like wordpress. Now you connect with ssh as
    > ssh username@127.0.0.1 -p 4242

    Here username is the username of the server

* **UFW**: to check the firewall status and rules, run:
    > `sudo ufw status`

    To enable, disable or reload the firewall you can type:
    > `sudo ufw enable`

    > `sudo ufw disable`

    > `sudo ufw reload`

    You allow a port N using
    > `sudo ufw allow N/tcp`

    If you wish to disallow that port, then just delete it:
    > `sudo ufw delete allow N/tcp`

    A command
    > `ss -tulpn`

    allows to see all ports not connected (in a `UNCONN` mode) and waiting for a connection (in a `LISTEN` mode)

* **Hostname**: one can readily change it typing
    > `sudo hostnamectl set-hostname your_hostname`

    and then updating the hostname config file */etc/hosts*.
    Standard user (with my name) created besides root; it belongs to groups (`sudo`, `user42`). One can add as many users as they want using
    > `sudo adduser newuser`

    in the interactive mode, which automatically creates a home directory for the new user. To delete the user, run:
    > `sudo deluser --remove-home newuser`

    which also removes home directory of the new user. To add a person to a group run:
    > `sudo usermod -aG groupname username`

    If you wish to delete a user in a group, run:
    > `sudo deluser username groupname`

    To see all actual groups in the system you can write:
    > `sudo getent group`

    If you wish to check the list of groups the user belongs, just type:
    > `groups username`


### Website in the bonus part
* Install Lighttpd, PHP (certain packages) and MariaDB, as well as curl and tar:
  > `sudo apt install lighttpd mariadb-server php-fpm php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl curl tar`

  + To check installed packages of php run:
    > `php -m`

    it shows PDO, mysqli, curl, dom, gd, mbstring, xml, zip, etc.

  + Configure maria-db:
    > `sudo mariadb-secure-installation`

    Create first database:
    > `sudo mariadb`

    > ```SQL
    > CREATE DATABASE wordpress
    > CHARACTER SET utf8mb4
    > COLLATE utf8mb4_unicode_ci;
    > CREATE USER 'wpuser'@'localhost'
    > IDENTIFIED BY 'PASSWORD';
    > GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
    > FLUSH PRIVILEGES;
    > EXIT;
    >```

    Verify if everything is ok:
    >```SQL
    > mariadb -u wpuser -p wordpress
    > SHOW GRANTS;
    >```


  + Check PHP-FPM configuration for lighttpd:
    >```bash
    > ls -l /etc/lighttpd/conf-available/ | grep fastcgi
    > ls -l /etc/lighttpd/conf-enabled/
    > ls -l /run/php/
    > readlink -f /run/php/php-fpm.sock
    >```

    and now run:
    > `sudo lighty-enable-mod fastcgi-php-fpm`

    to enable lighttpd modules for php, and then reload lighttpd:
    > `sudo systemctl reload lighttpd`

    Now check lighttpd document root:
    > `grep -n 'server.document-root' /etc/lighttpd/lighttpd.conf`

    which typically */var/www/html*
    and remove the index-file there:
    > `sudo mv /var/www/html/index.lighttpd.html /var/www/html/index.lighttpd.html.bak`

  + To download files from internet, we need specific library:
    > `sudo apt install wget`

  + Using wget we download latest version of WordPress:
    > `wget https://wordpress.org/latest.tar.gz`

    If we open the archiv:
    > `tar -xzf latest.tar.gz`

    we see a folder called *wordpress*, which contains bunch of folders and files like
    *index.php*, *wp-admin/* etc. We compy wordpress folder with all entries to lighttpd doc-root:
    > `sudo cp -a wordpress/. /var/www/html/`

    Then in that copied folder we change wp-config-sample file:
    > `sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php`

    This file contains main configurations of wordpress. Add the following commands:
    > ```
    > define( 'DB_NAME', 'wordpress' );
    > define( 'DB_USER', 'wpuser' );
    > define( 'DB_PASSWORD', 'PASSWORD' );
    > define( 'DB_HOST', 'localhost' );
    >```

    Type:
    > `php -l /var/www/html/wp-config.php`

    to check for syntax errors in the config-file.
    Next, change the owner of www/html files and directories (except the config-file):
    >```bash
    > sudo chown -R www-data:www-data /var/www/html
    > sudo find /var/www/html -type d -exec chmod 755 {} \;
    > sudo find /var/www/html -type f -exec chmod 644 {} \;
    > sudo chmod 640 /var/www/html/wp-config.php
    > ```

    + Finally, we install worpress. First download it:
      >```bash
      > sudo wget -O /tmp/wp-cli.phar https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
      > sudo install -m 755 /tmp/wp-cli.phar /usr/local/bin/wp
      >```

      To verify one can run:
      >```bash
      > php /tmp/wp-cli.phar --info
      > wp --info
      > ```


      Check wordpress installing configuration:
      > ```
      > sudo -u www-data wp config get DB_NAME --path=/var/www/html
      > sudo -u www-data wp core is-installed --path=/var/www/html
      > sudo -u www-data wp core is-installed --path=/var/www/html --url=http://localhost
      > ```

      Then install basic features of the website:
      > ```bash
      > sudo -u www-data wp core install \
      > --path=/var/www/html \
      > --url=http://localhost \
      > --title="My WordPress Site" \
      > --admin_user="wpadmin" \
      > --admin_password="PASSWORD" \
      > --admin_email="you@example.com"


      Thus, wordpress website URL is configured as *http://localhost*. Check:
      > `curl -I http://localhost`

      It should show *HTTP... 200 OK*

* To complete the bonus part, we need to install also some other service. AI is suggesting *redis*, which allows to caching the newly intalled website and communicates with php:
  > ```
  > sudo apt install redis-server
  > sudo apt install php-redis
  > sudo systemctl restart php8.4-fpm
  > ```

  + Install the WordPress Redis Object Cache plugin:
    > `sudo -u www-data wp plugin install redis-cache --activate --path=/var/www/html --url=http://localhost`

  + Because Redis is running on the same server as WordPress, we'll use the local address 127.0.0.1 and the default Redis port 6379:
    >```bash
    > sudo -u www-data wp config set WP_REDIS_HOST 127.0.0.1 --type=constant --path=/var/www/html --url=http://localhost
    > sudo -u www-data wp config set WP_REDIS_PORT 6379 --raw --type=constant --path=/var/www/html --url=http://localhost
    > sudo -u www-data wp redis enable --path=/var/www/html --url=http://localhost
    > ```


  + Finally, we add wordpress urls so that we can open it using the browsers:
    >```bash
    > sudo -u www-data wp config set WP_HOME 'http://127.0.0.1:8080' --type=constant --path=/var/www/html
    > sudo -u www-data wp config set WP_SITEURL 'http://127.0.0.1:8080' --type=constant --path=/var/www/html
    >```


### Note

Actually, you have to allow the port 80 to open the website from your browser.
But this should be done while checking the project, not before.


## Resources

### Documentation & References

Apart from YouTube videos handling Born2beroot project, we refer to:

* **Debian Documentation:** [Debian GNU/Linux Installation Guide & Wiki](https://www.debian.org/doc/)

* **Sudo Manual:** [Sudoers File Syntax & Options (`man sudoers`)](https://www.sudo.ws/docs/man/sudoers.man/)

* **UFW Documentation:** [Ubuntu/Debian Uncomplicated Firewall Guide](https://wiki.ubuntu.com/UncomplicatedFirewall)

* **Linux Partitioning:** [FHS (Filesystem Hierarchy Standard) Guidelines](https://refspecs.linuxfoundation.org/fhs.shtml)

---

### Use of AI

AI was used as a support tool during development in the following capacities:

* **Drafting & Documentation:** Assisting in structuring and writing sections of this `README.md` (e.g., formatting comparisons and LaTeX syntax explanations).

* **Configuration Syntax Checking:** Verifying parameter syntax for system configurations, such as the `badpass_message` option in `/etc/sudoers` or creating `monitoring.sh`.

> **Note:** All command execution, operating system installations, and firewall configurations were performed and verified manually on the target machine.

