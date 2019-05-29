### Raspberry

See the corresponding [AWS Docs article](https://docs.aws.amazon.com/greengrass/latest/developerguide/module1.html). 


Prerequisites:
* The latest release of Raspbian.
* You have cert and key files in the current folder.
* RaspberryPi is accessible via SSH from your local workstation, use SSH to run all the commands marked as "At Raspberry".

#### Identity settings
At Raspberry:
```bash
sudo adduser --system ggc_user
sudo addgroup --system ggc_group
```

#### Synlinks settings
At Raspberry:
```bash
sudo -s
touch /etc/sysctl.d/98-rpi.conf
echo -e "\n\nfs.protected_hardlinks = 1\nfs.protected_symlinks = 1" >> /etc/sysctl.d/98-rpi.conf
reboot
```
After about a minute, connect to the Pi using SSH and then run the following command to confirm the change:

```bash
sudo sysctl -a 2> /dev/null | grep fs.protected
```
You should see `fs.protected_hardlinks = 1 and fs.protected_symlinks = 1`.

#### CGroup settings
```bash
sudo nano /boot/smdline.txt
```

Add to the same line
```
cgroup_enable=memory cgroup_memory=1
```

### Install Python Greengrass SDK @ RaspberryPi
At Raspberry:
```bash
sudo pip install greengrasssdk
sudo pip install RPi.GPIO
sudo pip install psutil
```

### Copy files to RaspberryPi

Download AWS IoT Greengrass Core Software distribution (`greengrass-linux-armv7l-1.9.0.tar.gz` or so).

At Raspberry:
```bash
cd /home/pi
mkdir install-gg
```

Locally (change the IP address and file names to the respective ones):
```bash
scp greengrass-linux-armv7l-1.9.0.tar.gz pi@10.0.1.14:/home/pi/install-gg
scp 63a8a448b5-certificate.pem.crt pi@10.0.1.14:/home/pi/install-gg
scp 63a8a448b5-private.pem.key pi@10.0.1.14:/home/pi/install-gg
scp 63a8a448b5-public.pem.key pi@10.0.1.14:/home/pi/install-gg
```

At Raspberry:
```bash
cd /home/pi/install-gg
sudo tar -xzvf greengrass-linux-armv7l-1.9.0.tar.gz -C /
sudo cp 63a8a448b5-certificate.pem.crt /greengrass/certs/63a8a448b5.cert.pem
sudo cp 63a8a448b5-private.pem.key /greengrass/certs/63a8a448b5.private.key
sudo cp 63a8a448b5-public.pem.key /greengrass/certs/63a8a448b5.public.key
cd /greengrass/certs/
sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
```


#### Greengrass Core Configuration
Edit the config file `/greengrass/config/config.json`. 
Set the following values in the configuration:

| Field | Meaning |
| ------------- | :------------- |
| `certPath` | Name of the `.pem.crt` file |
| `keyPath `        | Name of the `-private.pem.key` file |
| `thingArn`        | Core thing ARN. Can be found in "AWS IoT ~> Manage ~> Things ~> (choose the thing) ~> Details ~> `arn:...` |
| `iotHost`        | IoT Host URL. Can be found in "AWS IoT ~> Manage ~> Things ~> (choose the thing) ~> Interact ~> HTTPS ~> `....iot.us-west-2.amazonaws.com` |
| `ggHost`        | `greengrass-ats.iot.us-west-2.amazonaws.com` (change to the respective region) |
| `privateKeyPath`        | `file:///greengrass/certs/`+name of the `-private.pem.key` file |
| `certificatePath`        | `file:///greengrass/certs/`+name of the `.pem.crt` file |

Example:
```
{
  "coreThing" : {
    "caPath" : "root.ca.pem",
    "certPath" : "63a8a448b5.cert.pem",
    "keyPath" : "63a8a448b5.private.key",
    "thingArn" : "arn:aws:iot:us-west-2:531877574785:thing/my-raspberry-core-thing",
    "iotHost" : "aldkbv2oua7qj-ats.iot.us-west-2.amazonaws.com",
    "ggHost" : "greengrass-ats.iot.us-west-2.amazonaws.com",
    "keepAlive" : 600
  },
  "runtime" : {
    "cgroup" : {
      "useSystemd" : "yes"
    }
  },
  "managedRespawn" : false,
  "crypto" : {
    "principals" : {
      "SecretsManager" : {
        "privateKeyPath" : "file:///greengrass/certs/63a8a448b5.private.key"
      },
      "IoTCertificate" : {
        "privateKeyPath" : "file:///greengrass/certs/63a8a448b5.private.key",
        "certificatePath" : "file:///greengrass/certs/63a8a448b5.cert.pem"
      }
    },
    "caPath" : "file:///greengrass/certs/root.ca.pem"
  }
}

```

File permissions at Raspberry (recommended):
```
sudo chmod 0600 /greengrass/config/config.json
```

#### Start/stop the Greengrass Daemon
Start the service at Raspberry:
```bash
cd /greengrass/ggc/core/
sudo ./greengrassd start
```

To stop:
```bash
cd /greengrass/ggc/core/
sudo ./greengrassd stop
```

####  Logs
Logs are locaed in `/greengrass/ggc/var/log`.
As long as folders are protected, use `sudo -s` to navigate the folders. Use it carefully.