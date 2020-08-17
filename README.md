# FreeTAKServer_support

FTS is normally run from the command line. These support files are intended to have FTS autostart on debian type systems (raspian, etc) which utilize systemd and rsyslog.



## Setup

You will need to copy files to system directories, which will require use of sudo or be done as root. 

From the source directory:

- Edit fts.service and make any IP address or port adjustments needed for FTS. 

The server IP address is assumed to be the current host, which should normally be the case. 
The default CoT port is 8087, default DataPackage port is 8080. 

If you do edit the params, note that exact quote usage matters! 

- copy the fts.service file to /lib/systemd/system/:

    ```
    sudo cp fts.service /lib/systemd/system/
    ```

- Ensure it's permissions are reasonable:
sudo chmod 644 /lib/systemd/system/fts.service
- copy the fts.conf file to /etc/rsyslog.conf and ensure permissions are also reasonable
    ```
    sudo cp fts.conf /lib/systemd/system/

    sudo chmod 644 /lib/systemd/system/fts.conf
    ```

- Reload rsyslog:  
    ```
    sudo systemctl restart rsyslog
    ```

- Make sure there are no syslog errors:
    ```
    tail /var/log/syslog
    ```

- Start FTS via system ctl:

    ```
    sudo systemctl daemon-reload 

    sudo systemctl start fts
    ```

- Again, make sure there are no syslog errors:

    ```
    tail /var/log/syslog
    ```

This will tell you if there were issues with your systemd config. 

- Then see if FTS is now logging in /var/log:
    ```
    ls -l /var/log/fts*
    ```

There should be an fts.log file, which has stdout messages from FTS

    ```
    tail -f /var/log/fts.log
    ```

- You'll want to setup logrotate to rotate the log files
    ```
    sudo cp fts.logrotate /etc/logrotate.d

    sudo chmod 644 /etc/logrotate.d/fts.logrotate
    ```

Cron runs logrotate nightly, so it will pickup the config file and start rotating the FTS console log. 

- Normal FTS logging can also be found in it's log directories. For PIP installs on debian it's normally in /usr/local/lib:

    ```
    ls -l /usr/local/lib/python*/dist-packages/FreeTAKServer/controllers/logs

    tail -f /usr/local/lib/python*/dist-packages/FreeTAKServer/controllers/logs/FTS.debug.log
    ```

## Usage
You can now manage FTS via systemd using commands like:
- Stop:
    ```
    sudo systemctl stop fts
    ```

- Start / Restart:
    ```
    sudo systemctl restart fts
    ```

## Python versions
Most debian based systems require invoking python as python3, which is how these scripts are configured. 

If your system has only python3, you may need to edit the service file to use python rather than python3. 

## Logging

These scripts are configured to log using syslog, which will nicely rotate files, etc. 

Note that FTS currently echo's CoT XML's to stdout, which is then logged in /var/syslog/fts.log. This 
can be quite verbose. If this becomes problematic you can edit the fts.conf file in /etc/rsyslogd.d
by commenting the current log line and uncommenting the one that logs to /dev/null. 

Ideally, we will find a way to redirect the FTS output separately, or set the log level to lower. 
