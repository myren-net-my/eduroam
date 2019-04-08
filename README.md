# eduroam Installation

We will install freeradius v3 from the script.

1. Update and refesh the CentOS package repositories to ensure we receive the most up-to-date packages:

       # yum -y update
              
2. Allow radius and radsec protocol in firewalld

       # firewall-cmd --permanent --add-port=1812/udp
       # firewall-cmd --permanent --add-port=1813/udp
       # firewall-cmd --permanent --add-port=2083/tcp
                
3. Reload the firewalld to make changes

       # firewall-cmd --reload
                
4. Install git to download the script :

       # yum -y install git
       # git clone https://github.com/myren-net-my/eduroam.git 
                
5. Edit file as below :

       # chmod -R 755 eduroam
       # cd eduroam
       # vi config.sh
       
       ## proxy.conf
       ## for example (kkselayang.edu.my OR polibanting.edu.my)
       export LOCAL_SCHOOL_REALM="institution.edu.my"
       export PARENT_IP_ADDRESSES=("203.80.16.18")

6. Run installation script

       # ./install.sh 
                
7. Start freeradius service

       # systemctl start radiusd
