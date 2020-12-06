# Sound Improvements

## PulseAudio Improvements

Default linux config provides average sound experience. To improve sound quality on linux, certain pulseaudio parameters can be changed. 

Recommended Parameteric values:
* default-sample-format = float32le
* resample-method = soxr-vhq
* realtime-priority = 9
* default-sample-rate = 48000
* alternate-sample-rate = 44100

The following commands automatically apply the above recommended values to the pulseaudio file named daemon.conf:

~~~
sudo sed -i 's/default-sample-format = s16le/default-sample-format = float32le/g' /etc/pulse/daemon.conf
sudo sed -i 's/resample-method = speex-float-1/resample-method = soxr-vhq/g' /etc/pulse/daemon.conf
sudo sed -i 's/realtime-priority = 5/realtime-priority = 9/g' /etc/pulse/daemon.conf
sudo sed -i 's/default-sample-rate = 44100/default-sample-rate = 48000/g' /etc/pulse/daemon.conf
sudo sed -i 's/alternate-sample-rate = 48000/alternate-sample-rate = 44100/g' /etc/pulse/daemon.conf
~~~

## PulseAudio Bluetooth Proprietary Modules (AptX, AptX HD, LDAC..)
Adds AptX, AptX HD, LDAC and AAC support in Linux. Greatly improves sound quality.

GitHub Repo Link - https://github.com/EHfive/pulseaudio-modules-bt

### Installation Instructions

#### Arch

~~~
yay -S --noconfirm pulseaudio-modules-bt
~~~

#### Debian

~~~
sudo add-apt-repository ppa:berglh/pulseaudio-a2dp
sudo apt update
sudo apt install pulseaudio-modules-bt libldac
~~~
