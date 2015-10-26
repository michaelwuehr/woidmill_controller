#########################################################
#About													#
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
#Raspberry Pi - SD Card Image							#
#########################################################

There is a prebuilt Rasbian image with WoidMill - Controller already running on it at port 80 for 115200 baud GRBL devices.  More information and a link to the .img can be found at http://www.michaelwuehr.de
The ethernet interface will get a DHCP address that you can ssh to.

username: pi
password: grod_schee_is



#########################################################
#GRBL baud rate settings								#
#########################################################

GRBL .9 uses 115200 baud rate while previous versions use 9600.
According to faster communication and support of the latest GRBL, default baud rate is 115200.
if you are using previous versions of GRBL, change config.serialBaudRate to 9600 in config.js.



#########################################################
#Installation - without SD Card Image					#
#########################################################

// Step 1:
git clone https://github.com/michaelwuehr/woidmill_controller.git
cd woidmill_controller
npm install

// Step 2:
Read http://www.hobbytronics.co.uk/raspberry-pi-serial-port

Set config.usettyAMA0 to 1 in config.js


#########################################################
#Configuration 											#
#########################################################

// optional:
edit config.js to change serial baud rate and web port



#########################################################
#Starting Server										#
#########################################################

// standalone
node server.js


// with forever
npm install -g forever
forever start server.js



#########################################################
#Access WoidMill - Controller							#
#########################################################

The default port in config.js is 80, you can change it by editing the file.

http://hostaddress:80/



#########################################################
#More about GRBL										#
#########################################################

https://github.com/grbl/grbl

https://github.com/grbl/grbl/wiki/Configuring-Grbl-v0.8

http://onehossshay.wordpress.com/2011/08/21/grbl-how-it-works-and-other-thoughts/


