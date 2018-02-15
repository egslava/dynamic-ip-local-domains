# dynamic-ip-local-domains
An instruction: how to get 2 .local domains work with 1 dynamic IP server

# dynamic-ip-local-domains
An instruction: how to get 2 .local domains work with 1 dynamic IP server

## Given
- 2 Wi-Fi networks
- _Server_ machine on Ubuntu 16.04.3 LTS “xenial” with two web-services on 0.0.0.0:80
- _Consumer_ laptop with OS X
## Want
- domain1.local, domain2.local associated with the web-services even though IPs are dynamical.
- when two laptops are in the same network (doesn’t matter which one) – _Consumer_ should be able to find _Server_’s web-services.
## Solution
On _Server_ machine only.

0. Install Avahi (actually, already installed)
1. Configure the main domain with Avahi.
In `/etc/avahi/avahi-daemon.conf` uncomment and edit these two lines:
```
[server]
host-name=domain1
domain-name=local
```
and run `sudo service avahi-daemon restart`
2. Install avahi-aliases
```bash
mkdir temp && cd temp
wget https://github.com/hmalphettes/avahi-aliases/archive/master.zip
unzip master.zip
cd avahi-aliases-master/
sudo ./install.sh
cd .. && gvfs-trash temp
```
3. Configure avahi-aliases to start when avahi-daemon is started. Create a file:
`/etc/init.d/avahi-publish-aliases`:
```bash
#!/bin/sh
# kFreeBSD do not accept scripts as interpreters, using #!/bin/sh and sourcing.
if [ true != "$INIT_D_SCRIPT_SOURCED" ] ; then
    set "$0" "$@"; INIT_D_SCRIPT_SOURCED=true . /lib/init/init-d-script
fi
### BEGIN INIT INFO
# Provides:          skeleton
# Required-Start:    $remote_fs $syslog $avahi
# Required-Stop:     $remote_fs $syslog 
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Avahi Publish Aliases
# Description:       Runs Avahi-Publish-Aliases
#                    placed in /etc/init.d.  This example start a
#                    single forking daemon capable of writing a pid
#                    file.  To get other behavoirs, implemend
#                    do_start(), do_stop() or other functions to
#                    override the defaults in /lib/init/init-d-script.
### END INIT INFO

# Author: Viacheslav Egorenkov <egslava@gmail.com>
#
# Please remove the "Author" lines above and replace them
# with your own name if you copy and modify this script.

DESC="Avahi Publish Aliases Service"
DAEMON=/usr/bin/avahi-publish-aliases
```
And make it run automatically:
`sudo update-rc.d avahi-publish-aliases default`

4. Configure a second domain: `sudo avahi-add-alias domain2.local`

5. remove the alias when it’s not needed anymore: `sudo avahi-remove-alias domain2.local`
