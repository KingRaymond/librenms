@@ -61,39 +61,56 @@ Connect to the server command line and follow the instructions below.
        dnf module enable php:remi-8.1
        dnf install bash-completion cronie fping gcc git httpd ImageMagick mariadb-server mtr net-snmp net-snmp-utils nmap php-fpm php-cli php-common php-curl php-gd php-gmp php-json php-mbstring php-process php-snmp php-xml php-zip php-mysqlnd python3 python3-devel python3-PyMySQL python3-redis python3-memcached python3-pip python3-systemd rrdtool unzip 
        ```
=== "Almalinux 9"
    === "NGINX"
        ```
        sudo dnf install -y epel-release
        sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
        sudo dnf module reset php
        sudo dnf module enable php:8.1
        sudo dnf install -y bash-completion cronie fping git ImageMagick mariadb-server mtr net-snmp net-snmp-utils nginx nmap php-fpm php-cli php-common php-curl php-gd php-gmp php-json php-mbstring php-process php-snmp php-xml php-zip php-mysqlnd python3 python3-PyMySQL python3-redis python3-memcached python3-pip python3-systemd rrdtool unzip
        sudo usermod -a -G librenms nginx
        ```

    === "Apache"
        ```
        Not tested / used
        ```

=== "Debian 12"
    === "NGINX"
        ```
        apt install apt-transport-https lsb-release ca-certificates wget acl curl fping git graphviz imagemagick mariadb-client mariadb-server mtr-tiny nginx-full nmap php8.2-cli php8.2-curl php8.2-fpm php8.2-gd php8.2-gmp php8.2-mbstring php8.2-mysql php8.2-snmp php8.2-xml php8.2-zip python3-dotenv python3-pymysql python3-redis python3-setuptools python3-systemd python3-pip rrdtool snmp snmpd unzip whois
        ```

## Add librenms user
## Add librenms group and user

```
useradd librenms -d /opt/librenms -M -r -s "$(which bash)"
sudo groupadd librenms
sudo useradd librenms -d /opt/librenms -M -r -s "$(which bash)"
sudo usermod -a -G librenms librenms
```

## Download LibreNMS

```
cd /opt
git clone https://github.com/librenms/librenms.git
sudo git clone https://github.com/librenms/librenms.git
```

## Set permissions

```
chown -R librenms:librenms /opt/librenms
chmod 771 /opt/librenms
setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
sudo chown -R librenms:librenms /opt/librenms
sudo chmod 771 /opt/librenms
sudo setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
sudo setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
```

## Install PHP dependencies

```
su - librenms
sudo su - librenms
./scripts/composer_wrapper.php install --no-dev
exit
```
@@ -126,6 +143,10 @@ Ensure date.timezone is set in php.ini to your preferred time zone.
    ```
    vi /etc/php.ini
    ```
=== "Almalinux 9"
    ```
    sudo vi /etc/php.ini
    ```

=== "Debian 12"
    ```bash
@@ -136,7 +157,7 @@ Ensure date.timezone is set in php.ini to your preferred time zone.
Remember to set the system timezone as well.

```
timedatectl set-timezone Etc/UTC
sudo timedatectl set-timezone Etc/UTC
```


@@ -156,7 +177,11 @@ timedatectl set-timezone Etc/UTC
    ```
    vi /etc/my.cnf.d/mariadb-server.cnf
    ```

=== "Almalinux 9"
    ```
    sudo vi /etc/my.cnf.d/mariadb-server.cnf
    ```

=== "Debian 12"
    ```
    vi /etc/mysql/mariadb.conf.d/50-server.cnf
@@ -172,13 +197,13 @@ lower_case_table_names=0
Then restart MariaDB

```
systemctl enable mariadb
systemctl restart mariadb
sudo systemctl enable mariadb
sudo systemctl restart mariadb
```
Start MariaDB client

```
mysql -u root
sudo mysql -u root
```

> NOTE: Change the 'password' below to something secure.
@@ -209,6 +234,11 @@ exit
    cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/librenms.conf
    vi /etc/php-fpm.d/librenms.conf
    ```
=== "Almalinux 9"
    ```bash
    sudo cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/librenms.conf
    sudo vi /etc/php-fpm.d/librenms.conf
    ```

=== "Debian 12"
    ```bash
@@ -420,7 +450,7 @@ Feel free to tune the performance settings in librenms.conf to meet your needs.
          </IfModule>

          <FilesMatch ".+\.php$">
            SetHandler "proxy:unix:/run/php-fpm-librenms.sock|fcgi://localhost"
            SetHandler "proxy:unix:/run/php-fpm/librenms.sock|fcgi://localhost"
          </FilesMatch>
        </VirtualHost>
        ```
@@ -432,7 +462,55 @@ Feel free to tune the performance settings in librenms.conf to meet your needs.
        systemctl enable --now httpd
        systemctl enable --now php-fpm
        ```
=== "Almalinux 9"
    === "NGINX"
        ```
        sudo vi /etc/nginx/conf.d/librenms.conf
        ```

        Add the following config, edit `server_name` as required:

        ```
        server {
        listen      80;
        server_name librenms.example.com;
        root        /opt/librenms/html;
        index       index.php;

        charset utf-8;
        gzip on;
        gzip_types text/css application/javascript text/javascript application/x-javascript image/svg+xml text/plain text/xsd text/xsl text/xml image/x-icon;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ [^/]\.php(/|$) {
            fastcgi_pass unix:/run/php-fpm/librenms.sock;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            include fastcgi.conf;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
        }
        ```

        > NOTE: If this is the only site you are hosting on this server (it
        > should be :)) then you will need to disable the default site.

        Delete the `server` section from `/etc/nginx/nginx.conf`

        ```
        sudo systemctl enable --now nginx
        sudo systemctl enable --now php-fpm
        ```

    === "Apache"
        ```
        Not tested / used
        ```
=== "Debian 12"
    === "NGINX"
        ```bash
@@ -530,6 +608,66 @@ Feel free to tune the performance settings in librenms.conf to meet your needs.

    Additional SELinux problems may be found by executing the following command

    ```
    audit2why < /var/log/audit/audit.log
    ```
=== "Almalinux 9"
    Install the policy tool for SELinux:

    ```
    sudo dnf install policycoreutils-python-utils
    ```

    <h3>Configure the contexts needed by LibreNMS</h3>

    ```
    sudo semanage fcontext -a -t httpd_sys_content_t '/opt/librenms/html(/.*)?'
    sudo semanage fcontext -a -t httpd_sys_rw_content_t '/opt/librenms/(rrd|storage)(/.*)?'
    sudo semanage fcontext -a -t httpd_log_t "/opt/librenms/logs(/.*)?"
    sudo semanage fcontext -a -t bin_t '/opt/librenms/librenms-service.py'
    sudo restorecon -RFvv /opt/librenms
    sudo setsebool -P httpd_can_sendmail=1
    sudo setsebool -P httpd_execmem 1
    sudo chcon -t httpd_sys_rw_content_t /opt/librenms/.env
    ```

    <h3>Allow fping</h3>

    Create the file http_fping.tt:

    ```
    vi http_fping.tt
    ```

    With the following contents. You can create this file anywhere, as it is a throw-away file. The last step
    in this install procedure will install the module in the proper
    location.

    ```

    module http_fping 1.0;

    require {
    type httpd_t;
    class capability net_raw;
    class rawip_socket { getopt create setopt write read };
    }

    #============= httpd_t ==============
    allow httpd_t self:capability net_raw;
    allow httpd_t self:rawip_socket { getopt create setopt write read };
    ```

    Then run these commands

    ```
    sudo checkmodule -M -m -o http_fping.mod http_fping.tt
    sudo semodule_package -o http_fping.pp -m http_fping.mod
    sudo semodule -i http_fping.pp
    ```

    Additional SELinux problems may be found by executing the following command

    ```
    audit2why < /var/log/audit/audit.log
    ```
@@ -546,11 +684,15 @@ Feel free to tune the performance settings in librenms.conf to meet your needs.
    Firewall not enabled by default

=== "CentOS 8"

    ```
    firewall-cmd --zone public --add-service http --add-service https
    firewall-cmd --permanent --zone public --add-service http --add-service https
    ```
=== "Almalinux 9"
    ```
    sudo firewall-cmd --zone public --add-service http --add-service https
    sudo firewall-cmd --permanent --zone public --add-service http --add-service https
    ```

=== "Debian 12"
    Firewall not enabled by default
@@ -562,33 +704,33 @@ This feature grants you the opportunity to use tab for completion on lnms comman
for normal linux commands.

```
ln -s /opt/librenms/lnms /usr/bin/lnms
cp /opt/librenms/misc/lnms-completion.bash /etc/bash_completion.d/
sudo ln -s /opt/librenms/lnms /usr/bin/lnms
sudo cp /opt/librenms/misc/lnms-completion.bash /etc/bash_completion.d/
```

## Configure snmpd

```
cp /opt/librenms/snmpd.conf.example /etc/snmp/snmpd.conf
sudo cp /opt/librenms/snmpd.conf.example /etc/snmp/snmpd.conf
```

```
vi /etc/snmp/snmpd.conf
sudo vi /etc/snmp/snmpd.conf
```

Edit the text which says `RANDOMSTRINGGOESHERE` and set your own community string.

```
curl -o /usr/bin/distro https://raw.githubusercontent.com/librenms/librenms-agent/master/snmp/distro
chmod +x /usr/bin/distro
systemctl enable snmpd
systemctl restart snmpd
sudo curl -o /usr/bin/distro https://raw.githubusercontent.com/librenms/librenms-agent/master/snmp/distro
sudo chmod +x /usr/bin/distro
sudo systemctl enable snmpd
sudo systemctl restart snmpd
```

## Cron job

```
cp /opt/librenms/dist/librenms.cron /etc/cron.d/librenms
sudo cp /opt/librenms/dist/librenms.cron /etc/cron.d/librenms
```

> NOTE: Keep in mind  that cron, by default, only uses a very limited
@@ -602,10 +744,10 @@ cp /opt/librenms/dist/librenms.cron /etc/cron.d/librenms
## Enable the scheduler

```
cp /opt/librenms/dist/librenms-scheduler.service /opt/librenms/dist/librenms-scheduler.timer /etc/systemd/system/
sudo cp /opt/librenms/dist/librenms-scheduler.service /opt/librenms/dist/librenms-scheduler.timer /etc/systemd/system/
systemctl enable librenms-scheduler.timer
systemctl start librenms-scheduler.timer
sudo systemctl enable librenms-scheduler.timer
sudo systemctl start librenms-scheduler.timer
```

## Copy logrotate config
@@ -615,9 +757,10 @@ become large and be rotated out.  To rotate out the old logs you can
use the provided logrotate config file:

```
cp /opt/librenms/misc/librenms.logrotate /etc/logrotate.d/librenms
sudo cp /opt/librenms/misc/librenms.logrotate /etc/logrotate.d/librenms
```


## Web installer

Now head to the web installer and follow the on-screen instructions.
@@ -643,6 +786,7 @@ That's it!  You now should be able to log in to
 you have configured HTTPS and taken appropriate web server hardening
 steps.


## Add the first device

We now suggest that you add localhost as your first device from within the WebUI.
