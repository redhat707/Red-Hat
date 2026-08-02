# Elinditjuk a rendszert, majd a Boot loader-nél nyomunk egy 'E' betűt hogy bejöjjön a Grub indítóparancs.
# A sor végére beillesztjük ezt : rw init=/bin/bash majd ctrl + x
<img width="1133" height="597" alt="image" src="https://github.com/user-attachments/assets/17f50b76-e8c8-48f9-96f0-76c15a0bde20" />

# Ezután a következőket kell tenni : csatolni kell a fájlrendszert mert read only : ' mount -o remount,rw / '

# passwd

# touch /.autorelabel

# exec /sbin/init
<img width="1133" height="597" alt="image" src="https://github.com/user-attachments/assets/2ac1cd77-35a6-4dac-bc74-1cba36b1ef08" />

