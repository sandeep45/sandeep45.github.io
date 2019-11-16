update and upgrade - 
```
sudo apt update
sudo apt upgrade
```

install `vim` unless you want to go crazy - `sudo apt-get install vim`. this will fix the madness of the built in `vi`

change pi hostname to something easy and also assign it a fixed local IP. I named mine `pi4`
verify by `hostname -I` & `hostname`

setup `afp` between mac and pi to easily see and move files accross platforms. it should be setup to allow mounting of the home and tmp directory. 
https://pimylifeup.com/raspberry-pi-afp/

```
[Homes]
basedir regex = /home

[tmp]
path = /tmp
```

Use `command + k` in finder to mount pi by its address `afp://pi4`. You should get an option with both directories available. Mount them both.

have ssh setup between the two machines with public keys on each other so you can ssh in either direction without having to key in username/password everytime. 
https://medium.com/@sandeeparneja/how-to-ssh-from-a-mac-to-raspberrypi-without-entering-the-password-every-time-afd769ecfb6

install the camera properly with cables int the right direction. enable it config and verify that it works

run `raspistill -o image.jpg` and confirm that the file.

Setup a directory in RAM
https://medium.com/@sandeeparneja/how-to-make-tmp-directory-use-ram-over-filesystem-d50bc1966aee

set aws - `aws configure` and `printenv` to confirm that env variables are there
whats missing can be added to `~/.profile`
add AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_DEFAULT_REGION, AWS_DEFAULT_OUTPUT to .bashrc

setup github profile and keys

setup permission so global packages can be installed - `sudo chown -R $USER /usr/local/lib`
install nvm & setup .nvmrc
 
install pip and virtualenv - https://virtualenv.pypa.io/en/stable/userguide/

```
sudo pip install --upgrade pip
sudo pip install virtualenv
python3 -m venv env
source ./env/bin/activate
deactivate
``` 

```
pip install opencv-contrib-python
pip install boto3
pip install pynt
pip install pytz 
```
```
cd ~/code/amazon-rekognition-video-analyzer/env/lib/python3.7/site-packages/cv2
ldd cv2.cpython-37m-arm-linux-gnueabihf.so | grep 'not found'
apt-file search pk_name
sudo apt install pk_name
```

https://blog.piwheels.org/how-to-work-out-the-missing-dependencies-for-a-python-package/

Install NVM, Node and .nvmrc




















