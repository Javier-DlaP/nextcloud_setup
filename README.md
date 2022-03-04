# Nextcloud setup

## Docker installation

### Install some required packages first
```bash
sudo apt update
sudo apt install -y \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common
```

### Get the Docker signing key for packages
```bash
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -
```

### Add the Docker official repos
```bash
echo "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
     $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list
```

### Install Docker
```bash
sudo apt update
sudo apt install -y --no-install-recommends \
    docker-ce \
    cgroupfs-mount
```

## Configure Raspberry Pi as a Docker host

### Check disk attached to the Raspberry Pi
```bash
sudo blkid
```
```bash
ls -al /dev/disk/by-uuid/
```

### Mount the disk and create the nextcloud data directory
```bash
sudo mkdir /media/ssd
sudo mount /dev/sda1 /media/ssd
sudo mkdir /media/ssd/nextcloud
sudo chown -R www-data:www-data /media/ssd/nextcloud
sudo chmod -R 775 /media/ssd/nextcloud
```

### Mount every time it boots
```bash
sudo apt install -y vim
sudo vim /etc/fstab
```
Example:
```text
proc           /proc           proc    defaults        0       0
PARTUUID=d4ava109-01  /boot          vfat    defaults        0       2
PARTUUID=d4ava109-02  /              ext4    defaults        0       1
# Mounting new disk
PARTUUID=d4ava109-03  /media/ssd     ext4    defaults        0       2
```

### Download nextcloud from dockerhub
```bash
sudo docker pull nextcloud
```

### Create the nextcloud container
```bash
sudo docker run -d --name nextcloud -p 8080:80 -v /media/ssd/nextcloud:/data nextcloud
```

Open in browser: http://192.168.1.40:8080 and configure the nextcloud database.

### Configure nextcloud for the first time
```bash
sudo docker exec -it nextcloud bash -c "echo chown -R www-data:www-data /data"
sudo docker exec -it nextcloud bash -c "echo chmod -R 775 /data"
```

<!-- ### Configure access from local network
```bash
sudo docker exec -it nextcloud bash
cd config/
apt update
apt install -y vim
vim config.php
```
```php
array(
    0 => 'localhost:8080',
    1 => '192.168.1.40:8080',
)
```
```bash
exit
``` -->

## Extra steps

### Enable copy/paste from/to the Raspberry Pi
```bash
sudo apt-get update
sudo apt-get install -y openssh-server
sudo ufw allow 22
```

### Check the Raspberry Pi temperature
```bash
vcgencmd measure_temp
```

### Start the nextcloud container on startup
```bash
sudo vim /etc/rc.local
```

Add the following line:

```bash
sudo docker container start nextcloud
```
