sudo apt install mdadm
sudo mdadm --create --verbose /dev/md0 --level=6 --raid-devices=5 /dev/sd{d, e, a, b, c}
sudo mkfs.ext4 -F /dev/md0
sudo mkdir -p /diskRAID
sudo mount /dev/md0 /diskRAID df -hT
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
echo '/dev/md0 /diskRAID ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab

sudo cat /proc/mdstat # Проверка работы