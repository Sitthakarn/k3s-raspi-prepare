# k3s-raspi-prepare
Prepare raspi for k3s cluster
Original tutorial from blog Alex ellis https://twitter.com/alexellisuk


## Prepare the RPi
### Step 1: Flash the OS to SD Card

I use raspbian stretch for RBpi3 and Buster for RBpi4

Flash the OS to the SD card

On MacOS you can usually type in: 
```
$ sudo touch /Volumes/boot/ssh
```
### Step 2: 
Power-up the device & customise it
Now power-up your device. It will be accessible on your network over ssh using the following command:

$ ssh pi@raspberrypi.local

Log in with the password raspberry and then type in sudo raspi-config.

Update the following:

#### 1. Set the GPU memory split to 16mb
#### 2. Set the hostname to whatever you want (write it down?)
#### 3. Change the password for the pi user
#### 4. Setting a static IP for each Raspberry Pi in your cluster.
```
$ sudo nano /etc/dhcpcd.conf
```

Change to this

```
#static IP configuration

interface eth0
static ip_address=192.168.1.15/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

#### 5. Copy over your ssh key
Do you have an ssh key?

$ ls -l ~/.ssh/id_rsa.pub
If that says file not found, then let's generate a key-pair for use with SSH. This means you can set a complicated password, or disable password login completely and rely on your public key to log into each RPi without typing a password in.

Hit enter to everything:

$ ssh-keygen
Finally run: ssh-copy-id pi@raspberrypi.local

#### 6. Enable container features
We need to enable container features in the kernel, edit /boot/cmdline.txt and add the following to the end of the line:
```
 cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
``` 

#### 7. Now reboot the device.

## Install OpenFaas

### Get the faas-cli
curl -SLsf https://cli.openfaas.com | sudo sh

### Forward the gateway to your machine
```
$ kubectl rollout status -n openfaas deploy/gateway

$ kubectl port-forward -n openfaas svc/gateway 8080:8080 &
```

### If basic auth is enabled, you can now log into your gateway:
```
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)

echo -n $PASSWORD | faas-cli login --username admin --password-stdin
```
```
faas-cli store deploy figlet
faas-cli list
```

### For Raspberry Pi
```
faas-cli store list \
 --platform armhf
```
```
faas-cli store deploy figlet \
 --platform armhf
```

### Find out more at:
### https://github.com/openfaas/faas

Thanks for using k3sup!
