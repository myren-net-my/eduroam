# Install FreeIPA

FreeIPA recommended system requirements
•	Server with 4gb ram 
•	CentOS 7.x server (64bit server)
•	2 vCPUs
•	Port 443 and 80 not used by another application
•	FQDN – Resolvable over public or private DNS server
•	10 GB Disk space

#System Software
Install the following software.
•	epel-release (Provides access to additional packages)
•	bind-utils (Provides bind utilities such as nslookup)
•	net-tools (Provides networking tools such as netstat)

      yum install –y epel-release bind-utils net-tools
      
Update all system software

      yum update –y
      
# Install the FreeIPA server packages

FreeIPA provides a range of services including a DNS service which we won’t be installing in this tutorial.

      yum install –y ipa-server
      
The software is now installed by requires configuring for your institution.

# Initial Configuration of the FreeIPA server

FreeIPA requires a DNS record to function. For this tutorial we will be using local entries /etc/hosts. This step is not necessary if there is a DNS server to resolve the hostname.

Find the IP Address of your machine

      ifconfig | grep inet
        inet 10.10.10.12  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 fe80::72eb:3067:4a49:5f17  prefixlen 64  scopeid 0x20<link>
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>

Ensure you have at least on network interface with an IP address of 10.10.10.12 for the APAN Training.

Set the hostname of the machine to apan-ldap.aaftest.xyz

      hostnamectl set-hostname apan-ldap.aaftest.xyz
      
 Confirm the hostname by running
 
      hostname
      
Run the IPS Server installation script. You will be prompted to answer a number of questions.

      ipa-server-install
				The log file for this installation can be found in /var/log/ipaserver-install.log
				==============================================================================
				This program will set up the IPA Server.

				This includes:
 					* Configure a stand-alone CA (dogtag) for certificate management
  				* Configure the Network Time Daemon (ntpd)
  				* Create and configure an instance of Directory Server
  				* Create and configure a Kerberos Key Distribution Center (KDC)
  				* Configure Apache (httpd)
  				* Configure the KDC to enable PKINIT

				To accept the default shown in brackets, press the Enter key.

				Do you want to configure integrated DNS (BIND)? [no]: no

				Enter the fully qualified domain name of the computer
				on which you're setting up server software. Using the form
				<hostname>.<domainname>
				Example: master.example.com.


				Server host name [apan-ldap.aaftest.xyz]: apan-ldap.aaftest.xyz

				The domain name has been determined based on the host name.

				Please confirm the domain name [aaftest.xyz]: aaftest.xyz

				The kerberos protocol requires a Realm name to be defined.
				This is typically the domain name converted to uppercase.

				Please provide a realm name [AAFTEST.XYZ]: AAFTEST.XYZ

				Certain directory server operations require an administrative user.
				This user is referred to as the Directory Manager and has full access
				to the Directory for system management tasks and will be added to the
				instance of directory server created for IPA.
				The password must be at least 8 characters long.

				Directory Manager password: ********
				Password (confirm): ********

				The IPA server requires an administrative user, named 'admin'.
				This user is a regular system account used for IPA server administration.

				IPA admin password: ********
				Password (confirm): ********
				
				The IPA Master Server will be configured with:
				Hostname:       apan-ldap.aaftest.xyz
				IP address(es): 10.10.10.12
				Domain name:    aaftest.xyz
				Realm name:     AAFTEST.XYZ

				Continue to configure the system with these values? [no]: yes


				The following operations may take some minutes to complete.
				Please wait until the prompt is returned.

				Configuring NTP daemon (ntpd)
  				[1/4]: stopping ntpd
  				[2/4]: writing configuration
  				[3/4]: configuring ntpd to start on boot
  				[4/4]: starting ntpd
				Done configuring NTP daemon (ntpd).
				Configuring directory server (dirsrv). Estimated time: 30 seconds

				……
				……

				Client configuration complete.
				The ipa-client-install command was successful

				Please add records in this file to your DNS system: /tmp/ipa.system.records.xvSk9n.db
				==============================================================================
				Setup complete

				Next steps:
        1. You must make sure these network ports are open:
                TCP Ports:
                  * 80, 443: HTTP/HTTPS
                  * 389, 636: LDAP/LDAPS
                  * 88, 464: kerberos
                UDP Ports:
                  * 88, 464: kerberos
                  * 123: ntp

        2. You can now obtain a kerberos ticket using the command: 'kinit admin'
           This ticket will allow you to use the IPA tools (e.g., ipa user-add)
           and the web user interface.

				Be sure to back up the CA certificates stored in /root/cacert.p12
				These files are required to create replicas. The password for these
				files is the Directory Manager password

Add firewall rules (firewalld)

Centos 7 servers generally install a firewall using the firewallD software.

				firewall-cmd --state
				
Will tell you if firewalld is installed and if it is running. To see what access is available from the Internet we use the public zone.

				firewall-cmd --zone=public --list-all
					public (active)
				  target: default
					icmp-block-inversion: no
				  interfaces: enp0s3
				  sources: 
				  services: ssh dhcpv6-client
				  ports: 
				  protocols: 
				  masquerade: no
				  forward-ports: 
				  source-ports: 
				  icmp-blocks: 
				  rich rules:

By default access is only available via ssh. This needs to be expanded to allow access to service provided by the IpA service. To add the required ports…

				firewall-cmd --zone=public --permanent --add-service={http,https,ldap,ldaps,kerberos,kpasswd,ntp}
				
These command only change the configuration of the firewall not the current state. You can restart the firewalld process to enable these rules.

				systemctl restart firewalld
				
				firewall-cmd --zone=public --list-services
						ssh dhcpv6-client http https ldap ldaps kerberos kpasswd ntp
						
# Extending the Schema

As we are mainly working with educational institutions users may need attributes from the eduPerson schema. Extending the IPA Ldap schema is very straight forward.
The file 60eduPerson.ldif hold the definition of the attributeTypes and objectClass definition for the eduPerson schema. To apply this schema to the LDAP server only requires coping of this file into the schema directory.

				cp 60eduPerson.ldif /etc/dirsrv/slapd-AAFTEST.XYZ /schema
		
Change the file ownership from root to dirsrv

				chown dirsrv:dirsrv 60eduperson.ldif
				
Then restart the IPA service.

				systemclt restart ipa
				
# Logging into the IPA dashboard

The FreeIPA service provides a web based dashboard that can be used to manage identities and objects. It also provides a management interface for the entire system.

https://apan-ldap.aaftest.xyz

The first login will use the password you entered during the initial configuration stage. 
