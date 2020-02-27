# nginx deployment and tips

Great guide from:

* [How to Install LEMP Stack (Nginx, MariaDB, PHP7.2) on Ubuntu 18.04 LTS](https://www.linuxbabe.com/ubuntu/install-lemp-stack-nginx-mariadb-php7-2-ubuntu-18-04-lts)
* [How to Install phpMyAdmin with Nginx (LEMP) on Ubuntu 18.04 LTS](https://www.linuxbabe.com/ubuntu/install-phpmyadmin-nginx-lemp-ubuntu-18-04-lts)

## Installing

### Setting hostname

    # hostname picard.prestatus.com
    
Verify on `/etc/hosts`

    127.0.0.1 picard.prestatus.com picard localhost

### Timezone

    # dpkg-reconfigure tzdata

### .inputrc

    # vim .inputrc

```bash
"\e[1~": beginning-of-line
"\e[2~": yank
"\e[3~": delete-char
"\e[4~": end-of-line
"\e[5~": history-search-backward
"\e[6~": history-search-forward
$if term=xterm
"\e[2;5~": yank
"\e[3;5~": delete-char
"\e[5;5~": history-search-backward
"\e[6;5~": history-search-forward
$endif
```

Add contents to `/etc/skel/.inputrc` 

### Update packages

Reboot instance before updating packages.

I prefer using `aptitude`

    # apt-get install aptitude
    # aptitude update
    # aptitude safe-upgrade

### Install Nginx web server

    # aptitude install nginx
    
Auto start Nginx when Ubuntu is booted

    # systemctl enable nginx

Start Nginx with:

    # systemctl start nginx
    
Check status

    # systemctl status nginx

Verify that `/usr/share/nginx/html` belongs to `www-data` (Nginx user) as the owner of web directory. By default it’s owned by the root user.

    # chown www-data:www-data /usr/share/nginx/html -R
    

