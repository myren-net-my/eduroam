# eduroam Installation

We will install freeradius v3 from the script.

1. Update and refesh the CentOS package repositories to ensure we receive the most up-to-date packages:

       # yum -y update
              
2. Allow radius and radsec protocol in firewalld

       # firewall-cmd --permanent --add-port=1812/udp
       # firewall-cmd --permanent --add-port=1813/udp
       # firewall-cmd --permanent --add-port=2083/tcp
       # firewall-cmd --permanent --add-port=80/tcp
       # firewall-cmd --permanent --add-port=443/tcp
                
3. Reload the firewalld to make changes

       # firewall-cmd --reload
                
4. Install application and download the script to install the certificate and radius service :

       # yum -y install git
       # git clone https://github.com/letsencrypt/letsencrypt
       # chmod -R 755 letsencrypt
       # cd letsencrypt
       # ./letsencrypt-auto certonly -d idp.XXXX.edu.my
       
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
		   /etc/letsencrypt/live/sso.myren.net.my/fullchain.pem
		   Your key file has been saved at:
		   /etc/letsencrypt/live/sso.myren.net.my/privkey.pem
		   Your cert will expire on 2019-07-10. To obtain a new or tweaked
		   version of this certificate in the future, simply run
		   letsencrypt-auto again. To non-interactively renew *all* of your
		   certificates, run "letsencrypt-auto renew"
                
7. Edit /etc/raddb/site-enabled/eap , replace private_key_file, certificate_file and CA_file with the following:

       private_key_file = /etc/letsencrypt/live/ins'X'.myren.net.my/privkey.pem
       certificate_file = /etc/letsencrypt/live/ins'X'.myren.net.my/cert.pem
       CA_file = /etc/letsencrypt/live/ins'X'.myren.net.my/chain.pem

8. Start freeradius service

       # systemctl enable radisud
       # systemctl start radiusd
