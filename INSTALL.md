Basics
--

provision a card:

    diskutil list
    diskutil unmountDisk /dev/diskn
    sudo dd bs=1m if=~/Downloads/2016-03-18-raspbian-jessie.img of=/dev/rdiskn

then

    sudo raspi-config
    expand file system, enable camera

reboot

    sudo apt-get remove --purge wolfram-engine -y
    sudo apt-get update && sudo apt-get upgrade -y

---

Broadcast AP
--

install AP and node from the radiodan repo

    git clone https://github.com/radiodan/provision
    cd provision
    git fetch origin
    git checkout -b minimal origin/minimal

    sudo ./provision iptables node wifi-connect

change name of network if you like (to 'bird')

    sudo pico /etc/systemd/system/wifi-connect.service

reboot

---

Install Collector
--

    cd
    git clone https://github.com/libbymiller/birdsnapper

    cd birdsnapper
    npm install

install supervisord for process management

    sudo update-rc.d dnsmasq defaults
    sudo update-rc.d dnsmasq enable

    sudo apt-get install supervisor -y

    cd
    cd whe
    sudo cp shared/supervisor.conf /etc/init.d/supervisor
    sudo cp collector/collector_supervisor.conf /etc/supervisor/conf.d/collector.conf

---

Install Snapper
--

install prerequisites

    sudo apt-get install libopencv-dev python-opencv -y
    sudo apt-get install libcurl4-openssl-dev -y

add to supervisor

    cd
    cd whe
    sudo cp shared/supervisor.conf /etc/init.d/supervisor
    sudo cp emitter/snapper/snapper_supervisor.conf /etc/supervisor/conf.d/snapper.conf
    sudo cp emitter/sender/image_sender_supervisor.conf /etc/supervisor/conf.d/image-sender.conf

---

Recompiling snapper
--

(you may not need to)

    sudo apt-get install cmake -y

    cd /opt/vc
    sudo git clone https://github.com/raspberrypi/userland.git
    
    cd /opt/vc/userland 
    sudo chown -R pi:pi .
    sed -i 's/DEFINED CMAKE_TOOLCHAIN_FILE/NOT DEFINED CMAKE_TOOLCHAIN_FILE/g' makefiles/cmake/arm-linux.cmake

    sudo mkdir build
    cd build
    sudo cmake -DCMAKE_BUILD_TYPE=Release ..
    sudo make
    sudo make install

    cd
    cd birdsnapper/emitter/snapper/
    cp -r /opt/vc/userland/host_applications/linux/apps/raspicam/*  .

    cp CMakeLists.txt.snapper CMakeLists.txt

    cmake .
    make

---

If Collector is a separate machine
--

edit wp_supplicant config

    sudo pico /etc/wpa_supplicant/wpa_supplicant.conf

to

    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1

    network={
        ssid="bird"
        proto=RSN
        key_mgmt=NONE
    }

---

Debugging
--

logs:

    sudo tail -f /var/log/supervisor/*
