/etc/samba/smb.conf


```
# Change this to the workgroup/NT-domain name your Samba server will part of
   workgroup = WORKGROUP
...

[2see]
   comment = Carpeta de inicio de la Raspberry Pi
   path = /home/shares/public/media/2see
   browseable = Yes
   writeable = Yes
   only guest = no
   create mask = 0777
   directory mask = 0777
   public = no
```

restart samba
```
systemctl restart smbd
```

