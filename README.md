# Enabling_DAVIS-TCP
A how-to on how to enable TCP-Davis on your Linux Server, an alternative to Google's BBR congestion control but focusing on low latency over throughput.

---

<b>INTRODUCTION</b>

On network, a lower latency usually results in faster response times, and a faster response improves the quality of experience for the end users using the service you are providing

I am of the opinion that the quality of experience should take precedence even to the detriment of other considerations

When the network is saturated, latency goes up, contributing to worse quality of experience 

Congestion control algorithms are used to keep the service operating even under saturated network conditions (heavy load, etc), most of them focus on keeping the network efficient (achieving most throughput while trying to keep latency down), the most well known of the new algorithms is BBR, developed by Google, which google claims to have improved YouTube network throughput by 4% globally only by virtue of adoption

DAVIS TCP uses the same principles applied on BBR, but focusing on keeping latency down, even to the detriment of throughput, which in my opinion does more in improving the quality of experience.

---

<b>CONFIGURATION</b>

Tested on Ubuntu 20.04.x and Debian 10


<b>DEBIAN 10</b>

You will need a Linux Kernel version higher than 5.4.x, trying to compile on 4.9.x (The previous LTS kernel before 5.4.x) will fail

Install the new Kernel

    sudo nano /etc/apt/sources.list

    Include those lines at the end of the file, save and exit
    deb http://deb.debian.org/debian buster-backports main
    deb-src http://deb.debian.org/debian buster-backports main

    sudo apt update

    This will install the new kernel, the tools to compile the module, and git
    sudo apt install -t buster-backports linux-image-amd64 build-essential git


Restart to load the new Kernel

    sudo shutdown -r now


Now its necessary to install the kernel headers, needed to compile the module

    sudo apt install linux-headers-`uname -r`


Install TCP Davis

    cd /usr/src && sudo git clone https://github.com/lambda-11235/tcp_davis && cd tcp_davis
    
    sudo make -C "/usr/src/linux-headers-`uname -r`" M=/usr/src/tcp_davis modules
    
    This creates a file named tcp_davis.ko in the folder


Load module into the Kernel

    sudo insmod tcp_davis.ko


Check if module is loaded

    sudo lsmod | grep davis


If the module was loaded correctly, make the changes persistent

    sudo nano /etc/sysctl.conf

    Add this line at the end of the file, save and exit
    net.ipv4.tcp_congestion_control=davis

    Apply the new configuration
    sudo sysctl -p

    Copy the module file from the git folder to system
    sudo cp tcp_davis.ko /lib/modules/`uname -r`

    Enable module on startup
    sudo nano /etc/modules
	
    Add tcp_davis to the end of the file, save and exit
    tcp_davis

    Run depmod
    sudo depmod
    

Now restart the system and check if TCP-Davis was loaded on startup

     sudo sysctl net.ipv4.tcp_congestion_control


Now you can if you want, recover some space by removing the packages used to compile the module, assuming that you wont need them in the future

    sudo apt remove -y linux-headers-`uname -r` build-essential git && sudo apt autoremove -y && sudo apt autoclean -y && sudo rm -rf /usr/src/*

---

<b>UBUNTU 20.04</b>

Ubuntu comes with kernel 5.4.x, kernel headers and a lot of preinstalled packages so the only package you need to install is build-essential

    sudo apt install build-essential
    
    cd /usr/src && sudo git clone https://github.com/lambda-11235/tcp_davis && cd tcp_davis
    
    sudo make -C "/usr/src/linux-headers-`uname -r`" M=/usr/src/tcp_davis modules
    
    sudo cp tcp_davis.ko /lib/modules/`uname -r`
    
    sudo depmod
    
    sudo nano /etc/modules
    
    add this line at the end and save
    tcp_davis
    
    sudo nano /etc/sysctl.conf
    
    add this line at the end and save
    net.ipv4.tcp_congestion_control=davis
    
    Now restart the system and check if TCP-Davis was loaded on startup
    sudo sysctl net.ipv4.tcp_congestion_control

---

<b>CONCLUSION</b>

In conclusion, TCP-Davis should help your service maintain lower latency under network saturation, specially interesting for interactive services or services that are sensible to latency fluctuations, with the trade off that you spent throughput to do so, which in my opinion is an excellent deal.

---

<b>REFERENCES</b>

TCP-Davis Paper - https://arxiv.org/pdf/2012.14996.pdf</br>
TCP-Davis Github - https://github.com/lambda-11235/tcp_davis</br>
TCP-BBR Google Announcement - https://cloud.google.com/blog/products/networking/tcp-bbr-congestion-control-comes-to-gcp-your-internet-just-got-faster</br>
Information about the bufferbloat effect - https://www.bufferbloat.net/projects/</br>
Other congestion control algorithms - https://en.wikipedia.org/wiki/TCP_congestion_control</br>
