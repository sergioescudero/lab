dietpi@DietPi:~$ lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT
NAME         SIZE FSTYPE MOUNTPOINT
sda          1.8T
└─sda1       1.8T ntfs   /home/shares/public/media
mmcblk0     29.8G
├─mmcblk0p1 43.9M vfat   /boot
└─mmcblk0p2 29.8G ext4   /


### how to

mkdir -p /home/shares/public/media

ext4 / ext3 / ext2
mount /dev/sda1 /home/shares/public/media

NTFS
mount -t ntfs-3g /dev/sda1 /home/shares/public/media

exFAT
mount -t exfat /dev/sda1 /home/shares/public/media


Make the Mount Persistent (Auto-mount on Boot)

$ blkid /dev/sda1 
/dev/sda1: LABEL="TOSHIBA EXT" BLOCK_SIZE="512" UUID="843671A53671993E" TYPE="ntfs" PARTUUID="8289d5c9-01"

nano /etc/fstab
UUID=843671A53671993E  /home/shares/public/media  ntfs-3g  defaults,noatime,uid=1000,gid=1000  0  0


Before rebooting

mount -a

no errors - good

Set permissions

chown -R dietpi:dietpi /home/shares/public/media 
chmod -R 775 /home/shares/public/media 


