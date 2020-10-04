# This is the Build and Deploy how-to for Aeon on Android ARMv8 devices

First we need to create a Linux VM if you haven’t already or just use linux as your main OS if that is your style.
For this specific build with binaries I used Ubuntu 18.10. Please note that some of the dependencies will be different
if you are using 16.04 or other linux flavors. This build was done with the latest v0.12.9.0 release. Many of the build maker files and docker files were updated for the latest versions for the 12.9 release so we can build this now. 

## Building AeonD on Docker with Ubuntu 18.10

Install all Linux Dependencies for Aeon CLI:
```
sudo apt update && sudo apt install build-essential cmake git pkg-config libboost-all-dev libssl-dev libzmq3-dev libunbound-dev libsodium-dev libminiupnpc-dev libunwind8-dev liblzma-dev libreadline6-dev libldns-dev libexpat1-dev doxygen graphviz libpcsclite-dev libnorm-dev
```

Pull a fresh recursive pull of Aeon CLI code:
```
git clone --recursive https://github.com/aeonix/aeon.git
```

Update your folders:
```
cd Aeon && git submodule init && git submodule update
```

Checkout the latest version of code (or whatever version you want to build post 12.9):
```
Git checkout v0.12.9.0-aeon
```
Now we have Aeon CLI downloaded and ready to roll, we need to install Docker so can we build it all. Please note that
this was done on Ubuntu 18.10 so the specific packages may be different on older or newer systems. 

First we need to install docker:

```
sudo apt-get update

sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update

sudo apt-get install docker.io
```

Now that docker is installed we need to add a user group to it:

```
sudo groupadd docker

sudo usermod -aG docker $YOURUSERNAME. (Note: YOURUSERNAME should be your account user name, not YOURUSERNAME)

Sudo reboot
```

After reboot you can do a quick check to see if docker is working:
```
Docker run hello-world
```

If you get an error like this:
```
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock:
```
The docker group is not working properly, you can google for this solution or drop to root with sudo su


Install ABD (Android Debug Bridge):
```
sudo apt install adb
```

Next we need to build the Binaries. This will take some time especially if using a VM:
```
cd utils/build_scripts/ && docker build -f android64.Dockerfile -t aeon-android .  (Wait quite some time for it to build)
```
After it builds we need to create the files docker built:
```
docker create -it --name aeon-android aeon-android bash 
```
Now we need to copy the files from the docker to the proper folder in linux:
```
cd ../../
docker cp aeon-android:/opt/android/aeon/build/release/bin .
```

Now check that the binaries have been copied out to the docker container:
```
ls bin/   (Or check manually in the folder)
```

You should see these files listed: 

Aeon-blockchain-usage, aeon-blockchain-blackball, aeon-blockchain-export, aeon-blockchain-import, aeond, aeon-wallet-cli, aeon-wallet-rpc

Now we need to copy all of our files to our android phone so we can use them.
Plug your Android device in and make sure developer mode is enabled. Here is a quick how-to for setting your phone up for developer mode and debug mode for USB. - https://www.xda-developers.com/quickly-install-adb/
Also install ADB (android studio, 1.2GB for full package with android emulator if you want that)
Just the cli tools here: https://developer.android.com/studio/releases/platform-toolscd

With adb installed on your computer, confirm you can see your device attached to your computer via USB cable (you may need to authorize your computer on your Android screen):
```
adb devices
```
If the device shows and no errors are reported, copy the binaries to the sdcard:
```
adb push ./bin /data/local/tmp
```
If you download “Terminal Emulator” from the play store you can cd to find your way to the correct folder location via your device.

## Run aeonD via Android Debug Bridge (ADB)

There are different ways of running cross compiled apps directly. Some phones do not let you write to the init.d files so you can't set aeonD to start on startup. If you have a rooted device, you should be able to place the files in and run in the /sdcard folder (your "My Files" Internal storage folder). 
Open a terminal on your desktop after connecting your device to your PC and run these commands:

```
adb shell

cd /data/local/tmp/bin

./aeond --data-dir /data/local/tmp/bin
```

You should now have aeond running successfully on an Android device from your computer and adb bridge!
You can open another adb shell, change directory back to the aeon binaries and run the aeon-wallet-cli or aeon-wallet-rpc applications.

If you want to run this from your device, you need to install a terminal emulator such as ``ADB Shell`` or 
``Terminal Emulator`` from the Google Play store. You will also need to give permissions to the folder you are running the 
Daemon and Wallet files to through ABD shell. You may need root privileges for daemon to work if you get ``permission denied`` messages. This means you need to root your device. 

Please test and run these binaries. If enough community interest, this may be added to the official binaries eventually.
