# eduroam Installation

We will install freeradius v3 from the script.

1. Update and refesh the CentOS 7 package repositories to ensure we receive the most up-to-date packages:

       yum -y update
              
2. Allow radius and radsec protocol in firewalld

       firewall-cmd --permanent --add-port=1812/udp
       firewall-cmd --permanent --add-port=1813/udp
       firewall-cmd --permanent --add-port=2083/tcp
       firewall-cmd --permanent --add-port=80/tcp
       firewall-cmd --permanent --add-port=443/tcp
                
3. Reload the firewalld to make changes

       firewall-cmd --reload
       
4. Set SElinux to permissive mode

       setenforce 0
       sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
       
4. Install application and download the script to install the certificate and radius service :

       yum -y install git
       git clone https://github.com/letsencrypt/letsencrypt
       chmod -R 755 letsencrypt
       cd letsencrypt
       ./letsencrypt-auto certonly -d idp.XXXX.edu.my
       
            How would you like to authenticate with the ACME CA?
            - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
            1: Spin up a temporary webserver (standalone)
            2: Place files in webroot directory (webroot)
            - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
            Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 1
            Plugins selected: Authenticator standalone, Installer None
            Enter email address (used for urgent renewal and security notices) (Enter 'c' to
            cancel): admin@email.com

            - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
            Please read the Terms of Service at
            https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
            agree in order to register with the ACME server at
            https://acme-v02.api.letsencrypt.org/directory
            - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
            (A)gree/(C)ancel: A

            - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
            Would you be willing to share your email address with the Electronic Frontier
            Foundation, a founding partner of the Let's Encrypt project and the non-profit
            organization that develops Certbot? We'd like to send you email about our work
            encrypting the web, EFF news, campaigns, and ways to support digital freedom.
            - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
            (Y)es/(N)o: Y
            Obtaining a new certificate
            Performing the following challenges:
            http-01 challenge for idp.XXXX.edu.my
            Waiting for verification...
            Cleaning up challenges

		IMPORTANT NOTES:
		 - Congratulations! Your certificate and chain have been saved at:
		   /etc/letsencrypt/live/idp.XXXX.net.my/fullchain.pem
		   Your key file has been saved at:
		   /etc/letsencrypt/live/idp.XXXX.edu.my/privkey.pem
		   Your cert will expire on 2019-07-10. To obtain a new or tweaked
		   version of this certificate in the future, simply run
		   letsencrypt-auto again. To non-interactively renew *all* of your
		   certificates, run "letsencrypt-auto renew"
                
7. Install all the required packages

	   cd /root/
	   yum groupinstall "Development Tools"
	   yum install -y libtalloc-devel libtool libtool-ltdl-devel net-snmp-devel net-snmp-utils readline-devel libpcap-devel libcurl-devel openldap-devel python-devel mysql-devel sqlite-devel unixODBC-devel freetds-devel samba4-devel json-c-devel wget
	   
8. Download and extract the freeradius v3 package

	   wget ftp://ftp.freeradius.org/pub/freeradius/freeradius-server-3.0.19.tar.gz
	   tar zxf freeradius-server-3.0.19.tar.gz
	   
9. Build and install the package

	   cd freeradius-server-3.0.19
	   ./configure --prefix=/opt/freeradius
	   make
	   make install
	
10. Configure IDP server

	    cd /root/
	    git clone https://github.com/myren-net-my/eduroamv2
	    chmod -R 755 eduroamv2
	    cd eduroamv2
	    ./setup_irs
	    
	    ======= Setup .my IRS configuration =======
	    Input your reaml (e.g. 'university.edu.my) : XXXX.edu.my
	    Input your secret key (e.g. 'eduroamkey') : XXXXXXX
	    Input your Freeradius 3 installation directory (e.g. '/opt/freeradius/') : /opt/freeradius
	    Input your host certificate private key file (e.g. '/etc/letsencrypt/live/idp.XXXX.edu.my/privkey.pem') : 
	    Input your host certificate public key file (e.g. '/etc/letsencrypt/live/idp.XXXX.edu.my/cert.pem') :

11. Test run

	    cd /opt/freeradius/sbin/
	    ./radiusd -X
	    
	    =====the result=====
	    
	    	Listening on auth address * port 1812 bound to server default
		Listening on acct address * port 1813 bound to server default
		Listening on auth address 127.0.0.1 port 18120 bound to server inner-tunnel
		Listening on proxy address * port 52246
		Ready to process requests
		
	    ======Press Ctl-C to cancel======
