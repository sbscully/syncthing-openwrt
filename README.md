# syncthing-openwrt
Init script and usage for AArch64 based OpenWrt devices.

This document is only applicable for devices with an AArch64 chip, and is only
tested on OpenWrt 18.06.1 on a Raspberry Pi 3 Model B.

Usage
=====

1. Login to your OpenWrt device via SSH.

2. Download and extract the Syncthing tarball, and copy the syncthing
   executable to `/usr/bin` or any other location that are in your `PATH`
   environment variable. (Please replace the URL and file names with the
   latest version. You can find the link on https://syncthing.net .)
   ```shell
   $ curl -kLO https://github.com/syncthing/syncthing/releases/download/v0.14.51/syncthing-linux-arm64-v0.14.51.tar.gz
   $ tar -zxf syncthing-linux-arm64-v0.14.51.tar.gz
   $ cd syncthing-linux-arm64-v0.14.51
   $ cp syncthing /usr/bin/
   ```

3. Run Syncthing for the first time.
   ```shell
   syncthing
   ```
   When you see a message about your Node ID that looks like this:
   ```
   [2EQK3] 15:47:15 OK: Ready to synchronize default (read-write)
   [2EQK3] 15:47:15 INFO: Node 2EQK3ZR77PTBQGM44KE7VQIQG7ICXJDEOK34TO3SWOVMUL4QFBHA is "server1" at [dynamic]
   ```
   You can press Ctrl-C to stop it.

4. If you don't want the Syncthing Web GUI to be accessible from WAN, you can
   skip 5 and 6.

5. Modify the Syncthing configuration file.
   ```shell
   $ vi ~/.config/syncthing/config.xml
   ```
   Look for a section that looks like this:
   ```xml
   <gui enabled="true" tls="false">
      <address>127.0.0.1:8384</address>
   </gui>
   ```
   Change it to:
   ```xml
   <gui enabled="true" tls="false">
      <address>0.0.0.0:8384</address>
   </gui>
   ```

6. Open TCP port 8384 to WAN. Add the following lines in
   `/etc/config/firewall`:
   ```
   config rule
            option enabled '1'
            option target 'ACCEPT'
            option src 'wan'
            option proto 'tcp'
            option dest_port '8384'
            option name 'Syncthing Web'
   ```

7. Download `/etc/init.d/syncthing` from this repository and copy it to
   `/etc/init.d` and make it executable.
   ```shell
   $ curl -k https://raw.githubusercontent.com/sbscully/syncthing-openwrt/master/etc/init.d/syncthing
   $ cp syncthing /etc/init.d
   $ chmod +x /etc/init.d/syncthing
   ```
8. Enable and start the syncthing service.
   ```shell
   $ /etc/init.d/syncthing enable
   $ /etc/init.d/syncthing start
   ```

9. Open http://your-openwrt-device-address:8384 on your browser.

10. If you have allowed access to your Syncthing Web GUI from WAN, make sure
    to turn on "GUI Authentication Password" and "Use HTTPS for GUI" in the
    settings.

11. Open ports 22000/TCP and 21027/UDP to WAN, and set up port forward for
    port 22000/TCP. Add the following lines in `/etc/config/firewall`:
    ```
    config rule
             option target 'ACCEPT'
             option src 'wan'
             option proto 'tcp'
             option dest_port '22000'
             option name 'Syncthing TCP'

    config rule
             option enabled '1'
             option target 'ACCEPT'
             option src 'wan'
             option proto 'udp'
             option dest_port '21027'
             option name 'Syncthing UDP'

    config redirect
             option enabled '1'
             option target 'DNAT'
             option src 'wan'
             option dest 'lan'
             option proto 'tcp'
             option src_dport '22000'
             option dest_ip '192.168.1.1'
             option dest_port '22000'
             option name 'Syncthing'
    ```
