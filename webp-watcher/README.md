Produced by DigitalKog (https://www.digitalkog.co.uk)
Author: Stuart Cross
Released under GPLv3 License (https://www.gnu.org/licenses/gpl-3.0.en.html)

DESCRIPTION:
A Linux service that uses inotifywait to monitor folders for new 
jpg, jpeg, png files and creates a new .webp version using the original filename
with .webp on the end.
e.g.
myimage.jpg creates myimage.jpg.webp in the same location.
It also converts CMYK original images to RGB before Webp conversion.

Only tested on Ubuntu 18.04LTS, so far. Nothing special, so should work on other flavours/versions.

Probably only works correctly when started by root.

It will also scan the folders on each start/restart for any missed files,
and process them.
500K already converted files, will complete in less than a minute (AWS R5.Large).
It will take a while on first run, if none of your images have webp equivalents yet.
And spike your CPU!

OPERATION:
process ids are stored in /var/run/webp-watcher.pid.

processes ids in there are killed on restart/stop.

images are chown to the images parent folder owner/group

jpg by default create 80% quality webp
png by default create 90% quality webp

These percentages can only be modified within /etc/webp-watcher/webp-watcher.

Sometimes you have zero bytes images from previous disk full or failures (Wordpress)
you can uncomment these lines to log them:
#rm /etc/webp-watcher/zero_byte.txt
#echo "$filepath" >> /etc/webp-watcher/zero_byte.txt
(in /etc/webp-watcher/webp-watcher)

Folders to be watched require their full path stored in:
/etc/webp-watcher.conf
Each on a new line.

DEPENDENCIES:
This service is reliant on 2 dependencies that will need to be installed via APT:
cwebp - converts images to webp (sudo apt install webp)
convert - converts CMYK images to RGB (Part of imageMagick IIRC)

INSTALLATION:
Simply install dependencies.
Copy the start/stop service to /etc/init.d/webp-watcher
Create an /etc/webp-watcher.conf file
Copy the Bash script itself to /etc/webp-watcher.
Use /etc/init.d/webp-watcher {start|stop|restart|status}
Enjoy!

EXTRA:
Example /etc/webp-watcher.conf:
/var/www/www.site1.co.uk/images
/var/www/www.site2.co.uk/images

Hint: 'status' only reads the pid storage file, to actually confirm watchers are running use:
pidof inotifywait

PS: a standard Linux system only allows 8192 watched files. This will often need increasing.
cat /proc/sys/fs/inotify/max_user_watches
(will show you your current limit)

echo 10000000 | sudo tee /proc/sys/fs/inotify/max_user_watches
(will bump your limit temporarily to 10M - check your inode limit on the drive as your absolute max with df -hi)

echo fs.inotify.max_user_watches=10000000 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
(will set the limit permanently)
(Thanks: https://dev.to/rubiin/ubuntu-increase-inotify-watcher-file-watch-limit-kf4)

PS: 10M is our current production limit, without issues on AWS EC2 R5.Large
