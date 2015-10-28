#########################################################
#About
#########################################################

WoidMill - Controller is a free web-based controlling software for any GRBL based CNC mills and equivalent CNC hardware
For further information about WoidMill - Controller please refer to http://www.michaelwuehr.de/
This free software is based on grblweb project of Andrew Hodel <andrewhodel@gmail.com> http://www.xyzbots.com

Copyright 2015 Michael Wuehr <wmc@michaelwuehr.de> under the GNU AFFERO GENERAL PUBLIC LICENSE Version 3

You must share the source of your project and notify the original author via email in plain english or german if you include or use this code, even if it is included or used as a library on the web.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.

If you would like to include this code in a project which is not licensed under the AGPL V3, please contact the author at wmc@michaelwuehr.de



#########################################################
#Raspberry Pi - SD Card Image			
#########################################################

There is a prebuilt Rasbian image with WoidMill - Controller already running on it at port 80 for 115200 baud GRBL devices.  More information and a link to the .img can be found at http://www.michaelwuehr.de
The ethernet interface will get a DHCP address that you can ssh to.

username: pi

password: woidmill



#########################################################
#GRBL baud rate settings
#########################################################

GRBL .9 uses 115200 baud rate while previous versions use 9600.
According to faster communication and support of the latest GRBL, default baud rate is 115200.
if you are using previous versions of GRBL, change config.serialBaudRate to 9600 in config.js.



#########################################################
#Installation - without SD Card Image	
#########################################################

// Step 1:

install precompiled node.js v0.10.28 on Raspberry Pi

```
cd /opt
sudo wget http://nodejs.org/dist/v0.10.28/node-v0.10.28-linux-arm-pi.tar.gz
```

extract packages and remove packed ones

```
sudo tar xfvz node-v0.10.28-linux-arm-pi.tar.gz
sudo rm node-v0.10.28-linux-arm-pi.tar.gz
sudo mv node-v0.10.28-linux-arm-pi node/
```

open .profile and add the following line to end of file

```
nano ~/.profile
```

```
export PATH=$PATH:/opt/node/bin
```

reload .profile

```
source ~/.profile
```

// Step 2:

clone woidmill_controller repository to home folder

```
cd /home/pi
git clone https://github.com/michaelwuehr/woidmill_controller.git
```

// Step 3:

install woidmill_controller

```
cd woidmill_controller
npm install
```

// Step 4:

settings correct rights to allow port 80 handling

```
sudo apt-get install libcap2-bin
sudo setcap cap_net_bind_service=+ep /opt/node/bin/node
```

Woidmill - Controller should be set up correctly now



#########################################################
#Configuration
#########################################################

// optional:
edit config.js to change serial baud rate and web port



#########################################################
#Starting Server	
#########################################################

// standalone

```
cd /home/pi/woidmill_controller
node server.js
```

// with forever

```
npm install -g forever
cd /home/pi/woidmill_controller
forever start server.js
```

#########################################################
#Starting Server automatically at start-up	
#########################################################

// Step 1:

create file `node-run´

```
sudo nano /opt/node/bin/node-run
``` 

// Step 2:

fill file with following content:

```
#!/bin/sh
#
# Runs node.js against script, logging to a logfile. We have to do
# this because there's no way to call node directly and have start-stop-daemon
# redirect stdout to a logfile.
#

LOGFILE=/home/pi/woidmill_controller/node.log
NODE=/opt/node/bin/node
WORKING_DIR=/home/pi/woidmill_controller 
JAVASCRIPT=server.js

#pidfile must be first argument
PIDFILE=$1

#cd into working dir:
cd ${WORKING_DIR}
# Fork off node into the background and log to a file
${NODE} ${JAVASCRIPT} >>${LOGFILE} 2>&1 </dev/null &
 
# Capture the child process PID
CHILD="$!"
# write pid file:
echo $CHILD > $PIDFILE
```

// Step 3:

make executable

```
sudo chown root:root /opt/node/bin/node-run
sudo chmod +x /opt/node/bin/node-run
```

// Step 4:

create init.d script

```
sudo nano /etc/init.d/node
```

fill file with following:

```
#! /bin/sh
### BEGIN INIT INFO
# Provides:          node
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Starts node.js Server
# Description:       Starts node.js Server
#                  
### END INIT INFO

# PATH should only include /usr/* if it runs after the mountnfs.sh script
PATH=/sbin:/usr/sbin:/bin:/usr/bin:/usr/local/bin
NAME=node
DAEMON=/opt/node/bin/node-run
PIDFILE=/var/run/$NAME.pid
DAEMON_ARGS=$PIDFILE
SCRIPTNAME=/etc/init.d/$NAME
# Exit if the package is not installed
[ -x "$DAEMON" ] || exit 0

# Read configuration variable file if it is present
# [ -r /etc/default/$NAME ] && . /etc/default/$NAME

# Load the VERBOSE setting and other rcS variables
. /lib/init/vars.sh

# Define LSB log_* functions.
# Depend on lsb-base (>= 3.2-14) to ensure that this file is present
# and status_of_proc is working.
. /lib/lsb/init-functions

#
# Function that starts the daemon/service
#
do_start()
{
    # Return
    #   0 if daemon has been started
    #   1 if daemon was already running
    #   2 if daemon could not be started
    start-stop-daemon --start --quiet --pidfile $PIDFILE --exec $DAEMON --test > /dev/null \
        || return 1
    start-stop-daemon --start --quiet --pidfile $PIDFILE --exec $DAEMON $DAEMON_ARGS \
        || return 2
    # Add code here, if necessary, that waits for the process to be ready
    # to handle requests from services started subsequently which depend
    # on this one.  As a last resort, sleep for some time.
}

#
# Function that stops the daemon/service
#
do_stop()
{
    # Return
    #   0 if daemon has been stopped
    #   1 if daemon was already stopped
    #   2 if daemon could not be stopped
    #   other if a failure occurred
    start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PIDFILE
    RETVAL="$?"
    # Many daemons don't delete their pidfiles when they exit.
    rm -f $PIDFILE
    return "$RETVAL"
}



case "$1" in
  start)
    [ "$VERBOSE" != no ] && log_daemon_msg "Starting $DESC" "$NAME"
    do_start
    case "$?" in
        0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
        2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
    esac
    ;;
  stop)
    [ "$VERBOSE" != no ] && log_daemon_msg "Stopping $DESC" "$NAME"
    do_stop
    case "$?" in
        0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
        2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
    esac
    ;;
  status)
    status_of_proc "$DAEMON" "$NAME" && exit 0 || exit $?
    ;;
  restart|force-reload)
    #
    # If the "reload" option is implemented then remove the
    # 'force-reload' alias
    #
    log_daemon_msg "Restarting "$NAME""
    do_stop
    case "$?" in
      0|1)
        do_start
        case "$?" in
            0) log_end_msg 0 ;;
            1) log_end_msg 1 ;; # Old process is still running
            *) log_end_msg 1 ;; # Failed to start
        esac
        ;;
      *)
        # Failed to stop
        log_end_msg 1
        ;;
    esac
    ;;
  *)
    echo "Usage: $SCRIPTNAME {start|stop|status|restart|force-reload}" >&2
    exit 3
    ;;
esac

:
```

// Step 5:

adjust rights

``` 
sudo chmod +x /etc/init.d/node
sudo chown root:root /etc/init.d/node
```

// Step 6:

update-rc:

```
sudo update-rc.d node defaults
```

Woidmill - Controller should start at boot and should be started and stopped by following commands:

Starting:

```
sudo service node start
```

Stopping:

```
sudo service node stop
```



#########################################################
#Access WoidMill - Controller	
#########################################################

The default port in config.js is 80, you can change it by editing the file.

http://hostaddress:80/



#########################################################
#More about GRBL	
#########################################################

https://github.com/grbl/grbl

https://github.com/grbl/grbl/wiki/Configuring-Grbl-v0.8

http://onehossshay.wordpress.com/2011/08/21/grbl-how-it-works-and-other-thoughts/


