---
layout: default
title: Broken Dokuwiki Hogfather instance after updating LXC host from Debian Buster to Bullseye
--- 


I upgraded an unprivileged Linux Container (LXC) from Debian buster to debian Bullsye on a Proxmox host that has been recently updated from 6.4 to 7.1.

Post upgrade, Dokuwiki was not loading and it took a while to login to the server over ssh.

## Issues

* Dokuwiki is not loading (nginx served)
* ssh/terminal logins take 15-20 seconds

## Troubleshooting!

### Lets start with logs

Review **systemctl status**

```
systemctl status
* docs
    State: degraded
     Jobs: 0 queued
   Failed: 4 units
...
```

```
systemctl --failed
  UNIT                          LOAD   ACTIVE SUB    DESCRIPTION
* sys-kernel-config.mount       loaded failed failed Kernel Configuration File System
* modprobe@drm.service          loaded failed failed Load Kernel Module drm
* systemd-logind.service        loaded failed failed User Login Management
* systemd-journald-audit.socket loaded failed failed Journal Audit Socket
```

**journalctl -b**

```
Dec 13 11:52:09 docs systemd[153]: systemd-logind.service: Failed to set up mount namespacing: /run/systemd/unit-root/proc: Permission denied
Dec 13 11:52:09 docs systemd[1]: Starting Permit User Sessions...
Dec 13 11:52:09 docs systemd[153]: systemd-logind.service: Failed at step NAMESPACE spawning /lib/systemd/systemd-logind: Permission denied
Dec 13 11:52:09 docs systemd[1]: systemd-logind.service: Main process exited, code=exited, status=226/NAMESPACE
Dec 13 11:52:09 docs systemd[1]: systemd-logind.service: Failed with result 'exit-code'.
Dec 13 11:52:09 docs systemd[1]: Failed to start User Login Management.
```
### Research: systemd/logind/namespaces/permissions

Lets start with `systemd.logind.service`. That seems fairly critical.

`sudo journalctl -xe | grep -E '(Failed|exited with)`

journalctl -xe reveals numerous namespacing permission denied errors like this.
`Failed to set up mount namespacing: /run/systemd/unit-root/proc: Permission denied`

After some google searching, I learn that this can show up on upgraded proxmox unpriviledged containers because new versions of apparmor are more restrictive about what services LXC are allowed to load/write to?? I am not 100% sure if this is the correct phrasing or understanding of the issue. I will need to do more reading.

But, it turns out that can be resolved by enabling nesting for existing LXC on proxmox. This is turned on by default for new containers in proxmox 7, but was not on by default on older 6.x containers. Will need to see if this is a security risk.

* [https://discuss.linuxcontainers.org/t/apparmor-blocks-systemd-services-in-container/9812]()
* [https://github.com/systemd/systemd/issues/17866]()
* [https://forum.proxmox.com/threads/systemctl-status-degraded-in-unprivileged-container.52667/]()

### Fix: Enable proxmox nested container feature

On proxmox host, checkbox **PVE->Container->Options->Features->Nesting**

OR

Edit `/etc/pve/lxc/<lxcID>.conf` and add line: `features: nesting=1`

Reboot the container

Review logs again. All errors gone except for this interesting tidbit. 

`systemd[1]: nginx.service: Failed to parse PID from file /run/nginx.pid: Invalid argument`

Ok that gives me a head start on the next issue.

### Research: Nginx served dokuwiki failing to load

**Checklist:**

1. Nginx service is healthy.
1. Nginx process itself seems to be running and the default website works. (can get default nginx page at server IP address)
1. Nginx served dokuwiki website is not loading, which indicates that there may be something wrong with the sites-available config for dokuwiki.

Review dokuwiki config in `/etc/nginx/sites-available`

```
server {
    listen 80;
    listen [::]:80;

    root /var/www/docs.schlimmers.space;
    index index.php index.html index.htm;

    server_name docs.schlimmers.space;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }
}
```
**Config checklist**

* listen on port 80 is working. (Confirmed port is open, firewall rule is configured and default nginx site is served)
* root and index are unchanged.
* server_name is unchanged. (confirmed system hostname is the same)
* ***location ~\.php$ is pointing to an old version of php-fpm module for fastcgi!*** Debian bullseye includes an upgrade from **php 7.3 to php 7.4** 

#### Fix: Update sites-available config

So I edit the config and in location ~ \\.php I bump php7.3-fpm to 7.4: `fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;`

Restart nginx with `systemctl restart nginx.service` and refresh the site in the browser. Dokuwiki now loads as expected.

Reboot the container one more time just for completeness sake. Logs in `journalctl -xe` are now clean.

Celebrate with coffeeeeeeeee!!!

## Lessons??

I should have reviewed the software prerequisites for dokuwiki -***before***- upgrading from Buster to Bullseye to see if any dokuwiki dependencies were getting updated in Bullseye. 

Most likely, I still would have not forseen the php-fpm version issue in Dokuwiki's nginx sites-available config, but it still would have narrowed down the possible issues a bit. I definitely lucked out that the `nginx.pid: invalid argument` error was showing up in **journalctl -xe** while I was troubleshooting the lxc/permissions/namespace/systemd/logind/apparmor errors.