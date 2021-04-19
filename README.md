# Enabling_DAVIS-TCP
A how-to on how to enable TCP-Davis on your Linux Server, an alternative to Google's BBR congestion control but focusing on low latency over throughput.

<b>INTRODUCTION</b>

On network, a lower latency usually results in faster response times, and a faster response improves the quality of experience for the end users using the service you are providing

I am of the opinion that the quality of experience should take precedence even to the detriment of other considerations

When the network is saturated, latency goes up, contributing to worse quality of experience 

Congestion control algorithms are used to keep the service operating even under saturated network conditions (heavy load, etc), most of them focus on keeping the network efficient (achieving most throughput while trying to keep latency down), the most well known of the new alorithms is BBR, developed by Google, which google claims to have improved YouTube network throughput by 4% globally ony by virtue of adoption

DAVIS TCP uses the same principles applied on BBR, but focusing on keeping latency down, even to the detriment of throughput, which in my opinion does more in improving the quality of experience.

---

<b>CONFIGURATION</b>

You will need a Linux Kernel version higher than 5.4.x, trying to compile on 4.9.x (The previous LTS kernel before 5.4.x) will fail

In the next example I'm using Debian 10 as a base, the latest stable Debian version, Debian 11 was not released yedt at the time of this writing

Install the new Kernel

    sudo nano /etc/sources.list

    Include those lines at the end of the file, save and exit
    deb http://deb.debian.org/debian buster-backports main
    deb-src http://deb.debian.org/debian buster-backports main

    sudo apt update

    sudo apt install -t buster-backports linux-image-amd64

    sudo apt install linux-headers-$(uname -r)

Install TCP Davis

    sudo apt install git
    
    git clone https://github.com/lambda-11235/tcp_davis && cd tcp_davis

    make
    
Load module into the Kernel

    insmod tcp_davis.ko

Check if module is loaded

    lsmod | grep davis
    
    sysctl net.ipv4.tcp_congestion_control

If the module was loaded correctly, make the changes persistent

    sudo nano /etc/sysctl.conf

    Add this line at the end of the file, save and exit
    net.ipv4.tcp_congestion_control=davis

    Apply the new configuration
    sudo sysctl -p

    Copy the module file from the git folder to system
    cp tcp_davis.ko /lib/modules/`uname -r`

    Enable module on startup
    sudo nano /etc/modules
    Add tcp_davis to the end of the file, save and exit
    tcp_davis

    Run depmod
    sudo depmod
    

Now restart the system and check if TCP-Davis was loaded on startup
    sysctl net.ipv4.tcp_congestion_control

---

<b>CONCLUSION</b>

In conclusion, TCP-Davis should help your service maintain lower latency under network saturation, sppecially interesting for interactive services os services that are sensible to latency fluctuations, with the tradeoff that you spent troughtput to do so, which in my opinion is an excellent deal.

---

<b>REFERENCES</b>

TCP-Davis Paper - https://arxiv.org/pdf/2012.14996.pdf
TCP-Davis Github - https://github.com/lambda-11235/tcp_davis
TCP-BBR Google Announcement - https://cloud.google.com/blog/products/networking/tcp-bbr-congestion-control-comes-to-gcp-your-internet-just-got-faster
Information about the bufferbloat effect - https://www.bufferbloat.net/projects/
Other congestion control algorithms - https://en.wikipedia.org/wiki/TCP_congestion_control
