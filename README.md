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
       
4. Install application and download the script to install the certificate :

       yum -y install git
       git clone https://github.com/letsencrypt/letsencrypt
       chmod -R 755 letsencrypt
       cd letsencrypt
       ./certbot-auto certonly --standalone --email name@yourmail.com -d idp.XXXX.edu.my
       
       	=====the result=====
            
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
	   yum groupinstall -y "Development Tools"
	   yum install -y libtalloc-devel libtool libtool-ltdl-devel wget
	   
8. Download and extract the freeradius v3 package

	   wget ftp://ftp.freeradius.org/pub/freeradius/freeradius-server-3.0.18.tar.gz
	   tar -zxf freeradius-server-3.0.18.tar.gz
	   
9. Build and install the package

	   cd freeradius-server-3.0.18
	   ./configure --prefix=/opt/freeradius
	   make
	   make install
	
10. Configure IDP server

	    cd /root/
	    git clone https://github.com/myren-net-my/eduroamv2
	    cd eduroamv2
	    chmod 755 setup_irs
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

12. Add local user

	    cd /opt/freeradius/etc/raddb
	    vi users
	    
	    ========
	    add test account below
	    ========
	    
	    test    Realm == "XXXX.edu.my", Cleartext-Password := "test1234"
	    
13. Add client / AP / controller details

	    vi clients.conf
	    
	    # Your Institution AP  example        #
		client AP1-traningroom {
			ipaddr   =  xx.xx.xx.xx
			secret  =   XXXX
			nas_type = other
			Operator-Name = 1XXXX.edu.my
			add_cui = yes
			require_message_authenticator = no
			shortname =  your-ap1
		}

14. Edit files proxy.conf and clients.conf, comment client flr2 and edit client flr1 to this value. To start edit press i, to save press ESC then :wq

	    cd /opt/freeradius/etc/raddb/
	    
	    vi clients.conf
	    
	    	client flr1 {
		ipaddr          = 203.80.16.18
		secret          = myr3n
		shortname     = flr1
		nas_type       = other
		virtual_server = eduroam
		Operator-Name  = 1nro-lab.myren.net.my
		}

		#client flr2 {
		#       ipaddr          = 119.40.121.26
		#        secret          = myr3n
		#        shortname       = flr2
		#        nas_type        = other
		#        virtual_server = eduroam
		#        Operator-Name  = 1nro2.eduroam.my
		#}
		
	    =============================================	
	    vi proxy.conf
	    
	    	# Radius-Proxy Server of National Roaming Operator
		home_server flr1 {
			ipaddr                  = 203.80.16.18
			port                     = 1812
			secret                  = myr3n
			status_check         = status-server
		}
		#home_server flr2 {
		#        ipaddr                  = 119.40.121.26
		#        port                     = 1812
		#        secret                  = myr3n
		#        status_check         = status-server
		#}
		
		home_server_pool eduroam {
			type                         = fail-over
			home_server             = flr1
		#        home_server            = flr2
			nostrip
		}
