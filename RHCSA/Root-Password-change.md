## Root Password Change

 Elinditjuk a rendszert, majd a Boot loader-nél nyomunk egy 'E' betűt hogy bejöjjön a Grub indítóparancs.
 A sor végére beillesztjük ezt : 
 
 1. **rw init=/bin/bash majd ctrl + x**
 
<img width="1133" height="597" alt="image" src="https://github.com/user-attachments/assets/17f50b76-e8c8-48f9-96f0-76c15a0bde20" />

 Ezután a következőket kell tenni : csatolni kell a fájlrendszert mert read only :
 
 2. **mount -o remount,rw /**

 3. **passwd**

 4. **touch /.autorelabel**

 5. **exec /sbin/init**
  
<img width="1135" height="584" alt="image" src="https://github.com/user-attachments/assets/ed210880-1aeb-4914-99f1-7ea621c60a3e" />


