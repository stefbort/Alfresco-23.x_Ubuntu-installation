# Not completed: WORKIN IN PROGRESS !!

-----

# Full Installation Alfresco Community 23 Guide from ZIP package

A didatic guide to full installation of Alfresco 23 Community from a ZIP package.

# Requisite
- OS: Ubuntu Server 24.04 LTS
- Java: OpenJDK 17
- Application server: Apache Tomcat 10.1
- Database: MariaDB 11.3
- Database Connector: MariaDB JDBC connector 2.7.12
- Message Broker: ActiveMQ 6.1.2
- Document Processor: LibreOffice 24.2
- Image Processor: ImageMagick 6.9
- Reverse Proxy; Nginx 1.24


# Procedure
To get started, prepare your production server by installing the prerequisite software (JRE, database, and message broker) before continuing.

1. Installing third-party software;
2. installing ActiveMQ;
3. creating DB for Alfresco;
4. installing Tomcat;
5. installing Alfresco Community 23;
6. installing Alfresco Module Package (AMP);
7. installing Alfresco Transformer Service (ATS);
8. first test.


## 1. Install third-party software
- After installed Ubuntu 24 update and upgade

```
sudo apt update
sudo apt full-upgrade -y
```

- install third-party software

```
sudo apt install -y htop neofetch wget apt-transport-https curl openssl
sudo apt install -y openjdk-17-jdk openjdk-17-jdk-headless openjdk-17-jre openjdk-17-jre-headless
sudo apt install -y imagemagick
sudo apt install -y libreoffice
sudo apt install -y libimage-exiftool-perl
```

- Install MariaDB
```
sudo curl -o /etc/apt/trusted.gpg.d/mariadb-keyring.pgp 'https://mariadb.org/mariadb_release_signing_key.pgp'

sudo echo "# MariaDB 11.3 repository list - created 2024-06-02 21:39 UTC
# https://mariadb.org/download/
X-Repolib-Name: MariaDB
Types: deb
# deb.mariadb.org is a dynamic mirror if your preferred mirror goes offline. See https://mariadb.org/mirrorbits/ for details.
# URIs: https://deb.mariadb.org/11.3/ubuntu
URIs: https://mirror.mva-n.net/mariadb/repo/11.3/ubuntu
Suites: mantic
Components: main main/debug
Signed-By: /etc/apt/trusted.gpg.d/mariadb-keyring.pgp" > /etc/apt/sources.list.d/mariadb.sources

sudo apt update
sudo apt install -y mariadb-server
```

 + enable and run MariaDB

```
systemctl enable mariadb --now
systemctl status mariadb
```

 + initialize MariaDB

```
mariadb-secure-installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] y
Enabled successfully!
Reloading privilege tables..
 ... Success!


You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] y
New password: 				YourSecureMainPassword
Re-enter new password: 		YourSecureMainPassword
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```


## 2. Installing ActiveMQ
(https://www.howtoforge.com/how-to-install-apache-activemq-on-ubuntu-20-04/)

- Download package

```
mkdir SW && cd SW
wget -O apache-activemq-6.1.2-bin.tar.gz "https://www.apache.org/dyn/closer.cgi?filename=/activemq/6.1.2/apache-activemq-6.1.2-bin.tar.gz&action=download"

tar xfz apache-activemq-6.1.2-bin.tar.gz -C /opt/
mv /opt/apache-activemq-6.1.2 /opt/activemq

sudo groupadd -r activemq
sudo useradd -d /opt/activemq -M -r -g activemq -s /usr/sbin/nologin activemq

sudo chown activemq:activemq -R /opt/activemq

sudo nano /usr/lib/systemd/system/activemq.service
```

```
[Unit]
Description=Apache ActiveMQ
After=network.target
[Service]
Type=forking
User=activemq
Group=activemq

ExecStart=/opt/activemq/bin/activemq start
ExecStop=/opt/activemq/bin/activemq stop

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
sudo systemctl enable activemq --now
```

## 3. Creating DB for Alfresco
- Run query

```
mariadb -u root -e "CREATE DATABASE alfresco CHARACTER SET utf8 COLLATE utf8_general_ci;"
mariadb -u root -e "CREATE USER alfresco@localhost IDENTIFIED BY 'alfrescoPass';"
mariadb -u root -e "GRANT ALL ON alfresco.* TO alfresco@localhost IDENTIFIED BY 'alfrescoPass';"
mariadb -u root -e "FLUSH PRIVILEGES;"
mariadb -u root -e "SHOW DATABASES;"
```

- Update MariaDB server configuration

```
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

```
[ ... ]
#max_connections         = 100
max_connections         = 275
[ ... ]
```

- restart MariaDB server

```
sudo systemctl restart mariadb
```


## 4. Installing Tomcat
- Download Tomcat 10.1.24

```
cd ~ && cd SW
mkdir /opt/alfresco
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.24/bin/apache-tomcat-10.1.24.tar.gz

tar xfz apache-tomcat-10.1.24.tar.gz -C /opt/alfresco
mv /opt/alfresco/apache-tomcat-10.1.24 /opt/alfresco/tomcat

chmod +x /opt/alfresco/tomcat/bin/*.sh
rm /opt/alfresco/tomcat/bin/*.bat

sudo nano /usr/lib/systemd/system/alfresco.service
```

```
[Unit]
Description=Alfresco Platform
After=network.target
[Service]
Type=forking
User=alfresco
Group=alfresco

ExecStart=/opt/alfresco/tomcat/bin/startup.sh
ExecStop=/opt/alfresco/tomcat/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
```

- Add MariaDB JDBC connector in Tomcat

```
cd ~ && cd SW
wget https://dlm.mariadb.com/3752064/Connectors/java/connector-java-2.7.12/mariadb-java-client-2.7.12.jar
sudo cp mariadb-java-client-2.7.12.jar /opt/alfresco/tomcat/lib/
```

- Delete all in ``webapps``

```
rm /opt/alfresco/tomcat/webapps/*
```


## 5. Installing Alfresco Community 23
- Download Alfresco 23 and Alfresco SSL Generator

```
cd ~ && cd SW
mkdir alfresco 

wget https://nexus.alfresco.com/nexus/service/local/repositories/releases/content/org/alfresco/alfresco-content-services-community-distribution/23.2.1/alfresco-content-services-community-distribution-23.2.1.zip -O alfresco/alfresco-content-services-community-distribution-23.2.1.zip
wget -O alfresco-ssl-generator.zip https://github.com/Alfresco/alfresco-ssl-generator/archive/refs/heads/master.zip

unzip alfresco-ssl-generator.zip

cd alfresco
unzip ../alfresco-content-services-community-distribution-23.2.1.zip

mv ../alfresco-ssl-generator-master/ssl-tool /opt/alfresco/
mv web-server/webapps/* /opt/alfresco/tomcat/webapps/
mv web-server/shared /opt/alfresco/tomcat/
mv web-server/lib/* /opt/alfresco/tomcat/lib/
cp -R web-server/conf/Catalina /opt/alfresco/tomcat/conf/
mkdir -p /opt/alfresco/data /opt/alfresco/logs /opt/alfresco/modules/{platform,share} /opt/alfresco/tomcat/shared/classes/lib
mv keystore /opt/alfresco/data/
mv amps /opt/alfresco/
mv bin /opt/alfresco/
mv licenses /opt/alfresco/

rm -R web-server
```

- update alfresco.xml and share.xml

```
nano /opt/alfresco/tomcat/conf/Catalina/localhost/alfresco.xml

[ ... ]
<PostResources base="/opt/alfresco/modules/platform"
[ ... ]

nano /opt/alfresco/tomcat/conf/Catalina/localhost/share.xml

[ ... ]
<PostResources base="/opt/alfresco/modules/share"
[ ... ]
```

- Generate certificates for mutual TLS

```
cd /opt/alfresco/ssl-tool/
bash ./run.sh -keystorepass keystore -truststorepass keystore
```

- copy certificate

```       
cp -R /opt/alfresco/ssl-tool/keystores/* /opt/alfresco/data/keystore/.
cp -R /opt/alfresco/ssl-tool/certificates /opt/alfresco/data/keystore/.
cp -R /opt/alfresco/ssl-tool/ca /opt/alfresco/data/keystore/.
```

- edit /opt/alfresco/tomcat/conf/catalina.properties and integrate ``shared.loader=``

```
nano /opt/alfresco/tomcat/conf/catalina.properties
```

```
[ ... ]
shared.loader="${catalina.base}/shared/classes","${catalina.base}/shared/lib/*.jar"
[ ... ]
```

- create ``/opt/alfresco/tomcat/bin/catalina.sh``

```
nano /opt/alfresco/tomcat/bin/setenv.sh
```

```
#
# Set to certificate

JAVA_OPTS="$JAVA_OPTS -Dencryption.keystore.type=JCEKS -Dencryption.cipherAlgorithm=DESede/CBC/PKCS5Padding -Dencryption.keyAlgorithm=DESede -Dencryption.keystore.location=/opt/alfresco/data/keystore/metadata-keystore/keystore -Dmetadata-keystore.password=mp6yc0UD9e -Dmetadata-keystore.aliases=metadata -Dmetadata-keystore.metadata.password=oKIWzVdEdA -Dmetadata-keystore.metadata.algorithm=DESede"


#
# Setting for Alfresco JVM

JAVA_OPTS="-Xms1G -Xmx5G $JAVA_OPTS "
```

```
chmod +x /opt/alfresco/tomcat/bin/setenv.sh
```

- update ``server.xml``

```
nano /opt/alfresco/tomcat/conf/server.xml
```

```
[ ... ]
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxParameterCount="1000"
               URIEncoding="UTF-8"
               maxHttpHeaderSize="32768"
            />
[ ... ]
```

- Set the owner

```
sudo groupadd -r alfresco
sudo useradd -d /opt/alfresco -M -r -g alfresco -s /usr/sbin/nologin alfresco

sudo chown alfresco:alfresco -R /opt/alfresco
```


## 6. Installing Alfresco Module Package (AMP)
(https://docs.alfresco.com/content-services/community/install/zip/amp/)

- Run ``apply_amps.sh`` as ``alfresco`` user

```
sudo su - -s /bin/bash alfresco
cd /opt/alfresco/bin
bash apply_amps.sh
exit
```


## 7. Installing Alfresco Transformer Service (ATS)
(https://docs.alfresco.com/transform-service/latest/)

- Download Alfresco PDF Render

```
cd ~ && cd SW
wget https://nexus.alfresco.com/nexus/service/local/repositories/releases/content/org/alfresco/alfresco-pdf-renderer/1.2/alfresco-pdf-renderer-1.2-linux.tgz
```
- publish ``alfresco-pdf-renderer``

```
sudo tar xfz alfresco-pdf-renderer-1.2-linux.tgz -C /opt/alfresco/bin/
sudo chmod -x /opt/alfresco/bin/alfresco-pdf-renderer
```

- download Alfresco Transform Core

```
wget https://nexus.alfresco.com/nexus/service/local/repositories/releases/content/org/alfresco/alfresco-transform-core-aio-boot/2.7.0-A1/alfresco-transform-core-aio-boot-2.7.0-A1.jar
```

- publish Alfresco Transform Core

```
sudo cp alfresco-transform-core-aio-boot-2.7.0-A1.jar /opt/alfresco/bin/
```

- make the script to run service di avvio-configurazione

```
sudo nano /opt/alfresco/bin/alfrescots.sh
```

```
#!/bin/bash

#
# Set global variables
#
SERVICE_NAME=LTS_-_Local_Tranfromation_Service
LOCAL_TRANSFORM_SERVICE_HOME=/opt/alfresco
PID_PATH_NAME=$LOCAL_TRANSFORM_SERVICE_HOME/tomcat/temp/LTS.pid 

#
# Set Variables of run application
#

# -DPDFRENDERER_EXE
TS_PDFRENDERER=/opt/alfresco/bin/alfresco-pdf-renderer

# -DLIBREOFFICE_HOME
TS_OFFICE=/usr/lib/libreoffice

# -DIMAGEMAGICK_ROOT -DIMAGEMAGICK_DYN
TS_IMAGEMAGICK=/usr/bin

# -DIMAGEMAGICK_EXE
TS_IMAGEMAGICK_CONVERT=/usr/bin/convert

# -DIMAGEMAGICK_CODERS
TS_IMAGEMAGICK_CONVERT_CODERS=/usr/lib/x86_64-linux-gnu/ImageMagick-6.9.12/modules-Q16/coders

# -DIMAGEMAGICK_CONFIG
TS_IMAGEMAGICK_CONVERT_CONFIG=/usr/lib/x86_64-linux-gnu/ImageMagick-6.9.12/config-Q16

# alfresco-transform-core-aio-boot-2.7.0-A1.jar
TS_JAR=/opt/alfresco/bin/alfresco-transform-core-aio-boot-2.7.0-A1.jar


#
# Function Start Transform Service
#
TsStart()
{
nohup java -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=80 \
    -DPDFRENDERER_EXE="$TS_PDFRENDERER" \
    -DLIBREOFFICE_HOME="$TS_OFFICE" \
    -DIMAGEMAGICK_ROOT="$TS_IMAGEMAGICK" \
    -DIMAGEMAGICK_DYN="$TS_IMAGEMAGICK" \
    -DIMAGEMAGICK_EXE="$TS_IMAGEMAGICK_CONVERT" \
    -DIMAGEMAGICK_CODERS="$TS_IMAGEMAGICK_CONVERT_CODERS" \
    -DIMAGEMAGICK_CONFIG="$TS_IMAGEMAGICK_CONVERT_CONFIG" \
    -DACTIVEMQ_URL="failover:(tcp://localhost:61616)?timeout=3000" \
    -jar $TS_JAR \
     /tmp 2>> /dev/null >>/dev/null \
     & echo $! > $PID_PATH_NAME  
}

#
# Function Stop Transform Service
#
TsStop()
{
    # PID=$(cat $PID_PATH_NAME);
    # printf "\n $SERVICE_NAME stoping ... \n" 
    /bin/kill $PID;         
    # printf "\n $SERVICE_NAME stopped ...\n" 
    # rm $PID_PATH_NAME       
}


echo "Process id path: $PID_PATH_NAME"

case $1 in 
start)
  echo "Starting $SERVICE_NAME ..."
  if [ ! -f $PID_PATH_NAME ]; then
    TsStart
    printf "\n $SERVICE_NAME started ...\n"
  else
    printf "\n $SERVICE_NAME is already running ...\n"
  fi
;;

stop)
  if [ -f $PID_PATH_NAME ]; then
        PID=$(cat $PID_PATH_NAME);
        printf "\n $SERVICE_NAME stoping ... \n"
        TsStop
        printf "\n $SERVICE_NAME stopped ...\n"
        rm $PID_PATH_NAME
  else          
         printf "\n $SERVICE_NAME is not running ...\n"
  fi    
;;
    
restart)  
  if [ -f $PID_PATH_NAME ]; then
        #
        # Process Stop
        PID=$(cat $PID_PATH_NAME);
        printf "\n Stoping $SERVICE_NAME ... \n"
        TsStop
        printf "\n Stopped $SERVICE_NAME ...\n"
        rm $PID_PATH_NAME
        #
        # Process Start
        echo "Starting $SERVICE_NAME ..."
        TsStart
        printf "\n Started $SERVICE_NAME ...\n"
    else
        printf "\n $SERVICE_NAME is not running ...\n"
    fi
 esac
```

- set file mode

```
sudo chmod +x /opt/alfresco/bin/alfrescots.sh
```

- create a systemd unit

```
sudo nano /usr/lib/systemd/system/alfrescots.service
```

```
[Unit]
Description=Alfresco Transform Service
After=network.target

[Service]
Type=forking
Restart=always

User=alfresco
Group=alfresco

ExecStart=/opt/alfresco/bin/alfrescots.sh start
ExecStop=/opt/alfresco/bin/alfrescots.sh stop

[Install]
WantedBy=multi-user.target
```

- re-load systemd units

```
sudo systemctl daemon-reload
```


## 8. First test
- Create ``alfresco-global.properties``

```
sudo nano /opt/alfresco/tomcat/shared/classes/alfresco-global.properties
```

```
# The server mode. Set value here
# UNKNOWN | TEST | BACKUP | PRODUCTION
system.serverMode=TEST

dir.license.external=/opt/alfresco/licenses

dir.root=/opt/alfresco/data
dir.keystore=/opt/alfresco/data/keystore

# Solr setup
#index.subsystem.name=solr6
#solr.secureComms=https
#solr.port=8983

# MariaDB setting
db.name=alfresco
db.username=alfresco
db.password=alfrescoPass
db.port=3306
db.host=127.0.0.1
db.pool.max=275
db.driver=org.mariadb.jdbc.Driver
db.url=jdbc:mariadb://127.0.0.1:3306/alfresco?useUnicode=yes&characterEncoding=UTF-8

user.name.caseSensitive=true
domain.name.caseSensitive=false
domain.separator=


#ActiveMQ setup
messaging.broker.url=failover:(tcp://localhost:61616)?timeout=3000

#Context generator
alfresco.context=alfresco
alfresco.host=vm08.kbsb.loc
alfresco.port=8080
alfresco.protocol=http
share.context=share
share.host=vm08.kbsb.loc
share.port=8080
share.protocol=http

imap.server.enabled=false
alfresco.rmi.services.host=0.0.0.0
smart.folders.enabled=false
#smart.folders.enabled=true
#smart.folders.model=alfresco/model/smartfolder.xml
#smart.folders.model.labels=alfresco/messages/smartfolder-model"

#This property is default true, here it it for information purpose.
local.transform.service.enabled=true
localTransform.core-aio.url=http://localhost:8090/
```

- set rigth owner

```
sudo chown alfresco:alfresco -R /opt/alfresco
```

- first run (**NB**: manualy proces)

```
systemctl start mariadb
systemctl start activemq
systemctl start alfrescots
systemctl start alfresco
```

- with a web broweser connect to ``http://< alfresco IP >:8080/``, autenticate with default account \(Username: **admin**   Password: **admin**\) and check and test evryting <br>
  **NB**: search system is not running.

- stop alfresco stack

```
systemctl stop alfresco
systemctl stop alfrescots
systemctl stop mariadb
systemctl stop activemq
```

**NOT COMPLETE**

**Woking in progress**

Coming soon: installing Alfresco Search System, installing the reverse proxy and makeing a Systemd unit to launch the Alfresco Stack.
