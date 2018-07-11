#getting pppd “must be root” after setting whole fs with chmod 777
By running chmod 777, you removed the setuid bit on pppd. To restore it, you need to run

chmod 4755 /usr/sbin/pppd
Note that by running a recursive chmod 777 in this manner, you’ve probably broken other permissions, and you’ve certainly made your system rather insecure.


