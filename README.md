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

                # cd eduroam
                # vi config.sh
                
                

## radsecproxy Installation

We will install radsecproxy v1.7.1 from source as the yum package manager only provides version 1.6.9.

1.  Update and refresh the Ubuntu package repositories to ensure we receive the most up-to-date packages:

                $ s

