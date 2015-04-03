# Introduction #

The objective of this guide is to help any user to get a full running Sylverant PSO server running in a Windows environment using Cygwin.

To have success on it, you need a minimum Unix level.
I.e: know how to get a SVN copy, and how to use make tools and a little about MySQL.

Please note, that as [r500](https://code.google.com/p/sylverant/source/detail?r=500), Sylverant will compile with IPv6 support by default. Cygwin already supports IPv6 on Windows 7, and partially on XP. In this guide, we'll indicate you the necessary flags to compile only with IPv4 support.

**Last update:** 3 July 2012

# Steps #

You will need **Cygwin 1.7** or a newer version. Get the latest Cygwin here: http://cygwin.com/install.html

## Compile phase ##

1. Install Cygwin and add the following packages (with their dependencies). If more than one version is found, use the latest one:

**autoconf automake gcc-core gcc-g++ gettext-devel gnutls-devel libgnutls(26) libiconv libopenssl(098) libtool libxml2 (must be >=2.6) make openssl openssl-devel readline subversion**


2.1. MYSQL Server side:

You can install a MySQL Server in your normal environment (Windows I suppose...) and on the same machine or on a remote one. Using cygwin as server doesn't work.

2.2. MySQL Client side:

Get MySQL source code (tested with version 5.1.57).
http://dev.mysql.com/downloads/mysql/5.1.html

Extract sources to: /usr/local/

```
cd /usr/local/mysql-5.x
./configure --without-server --without-readline --without-libedit CFLAGS=-O2
make
make install
```

**Multi-language server support**

If you want to have your server messages in other languages than English, you need to install the **mini18n** project.
**Note** You need CMake >= 2.8 to generate the Makefiles (i.e. Win32 version).
```
cd /usr/local
svn co https://yabause.svn.sourceforge.net/svnroot/yabause/trunk/mini18n minii18n
cmake -G "MinGW Makefiles"
make
make install
```

3. Grab latest SVN version of sylverant:
```
cd /usr/local
svn co http://sylverant.googlecode.com/svn sylverant
```

4. Compile libsylverant (needed for the rest of packages).
```
cd sylverant/trunk/libsylverant
autoreconf --install
./configure
make all
make install
```

5. Compile login\_server (check IPv6 flag)
```
cd sylverant/trunk/login_server
autoreconf --install
CFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib" ./configure [--disable-ipv6]
make all
make install
```
Note: If you compiled mini18n, sylverant will create a l10n folder inside share/sylverant.

6. Compile patch\_server. (check IPv6 flag)
```
cd sylverant/trunk/patch_server (Needed for PSO PC Players).
autoreconf --install
CFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib" ./configure [--disable-ipv6]
make all
make install
```

7. Compile ship\_server.
```
cd sylverant/trunk/ship_server
touch config.rpath (seems to correct automake 1.10 bug)
autoreconf --install
CFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib" ./configure [--disable-ipv6]
make all
make install
```

8. Compile shipgate.
```
cd sylverant/trunk/shipgate
touch config.rpath 
autoreconf --install
CFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib" ./configure
make all
make install
```


## Configure phase ##

9. Create config structure:
  * Create **"sylverant"** folder inside /usr/local/share/
  * Create **"config"** folder inside /usr/local/share/sylverant
  * Edit and put your XML config files inside config folder (ship\_config.xml, quests.xml, gms.xml, patch\_config.xml, sylverant\_config.xml (copy over legits.xml from ship\_server) ).
    * Remember to update your XML with the latest DTD config rules: http://sylverant.net/dtd/
  * Create **"info"** folder.
  * Create **"logs"** folder.

  * Optional: Create text files inside info folder (about.txt, bugs.txt, bugreport.txt, motd.txt)
  * Optional: Create **"patches"** folder.

  * For BB: You will need a "blueburst" folder with extra data.

10. Edit XML data:

See [ConfiguringSylverantShips](ConfiguringSylverantShips.md) for information about how to configure Sylverant Ship Server.

See [ConfiguringSylverant](ConfiguringSylverant.md) for information about how to configure Sylverant Login Server and Shipgate.

See [ConfiguringQuests](ConfiguringQuests.md) for information about how to configure Sylverant Quests.

See [ConfiguringPatchServer](ConfiguringPatchServer.md) for information about how to configure Sylverant Patch Server.

11. Create the "pso" database and their tables in the **MySQL** server.

See [DatabaseLayout](DatabaseLayout.md) for information about the layout of the tables in Sylverant.

**Tip:** Use Cygwin to connect to your MySQL server like:
```
mysql -h 127.0.0.1 -uroot -psomepass (pso < psoscript.sql)
```

12. For every PSO PC user, you have to add it to the database:

```
mysql> INSERT INTO guildcards VALUES (guildcard,account_id);
mysql> INSERT INTO pc_clients VALUES (guildcard,"serial_number","access_key");
```

See [FAQs](FAQs.md) for more information about how to add a PSOPC player.

13. Remember to open your Router / Firewall ports! A range 9000-12100 TCP/UDP is more than enough for DC, PC and GC players.


## X.509 Certs ##

Please note, there are lots of good tutorials to understand and use the X.509 Certificates. Here it only shows a quick shortcut to get it working...

```
$ mkdir /home/user/sslCA
$ cd /home/user/sslCA
$ mkdir demoCA
$ cd demoCA
$ echo 1000 > serial
$ touch index.txt
$ mkdir certs private newcerts
$ cd ..
```

Create your own CA (for Shipgate). Put a Common Name!
```
$ openssl.exe req –new –x509 –days 3650 –extensions v3_ca 
-keyout demoCA/private/cakey.pem –out demoCA/cacert.pem 
-config /usr/ssl/openssl.cnf
```

Next step: perform a SSL request.
```
$ openssl.exe req -new -nodes -out sylverant-req.pem -keyout demoCA/private/sylverant-key.pem -config /usr/ssl/openssl.cnf
```

Finally, signing CSR.
```
$ openssl.exe ca -config /usr/ssl/openssl.cnf -out sylverant-cert.pem -infiles sylverant-req.pem
```

Your certificate is: sslCA/sylverant-cert.pem
Your key is: sslCA/demoCA/private/sylverant-key.pem
Your CA is: sslCA/demoCA/cacert.pem

Copy these files inside your config folder and edit sylverant\_config.xml

Get the SHA1 Fingerprint of our ship\_server cert:
```
$ openssl.exe x509 -noout -sha1 -fingerprint -in sylverant-cert.pem
```

Create a new registry in MySQL 'ship\_data' table:
mysql> INSERT INTO ship\_data (ship\_number,memo,sha1\_fingerprint) VALUES ('0','',UNHEX('BB34765179347A0823787BE03756AA75B0D9E0CC'));

Where 'BB34...' is your SHA1 fingerprint without the ':' symbols.

## Execute phase ##

Time to start our server!

**IMPORTANT** Execute the programs in the following order: login\_server.exe, shipgate.exe, ship\_server.exe, patch\_server.exe

By default, the programs execute in daemon mode. If you want to see log messages, use **--nodaemon** argument.

17. Start the **login\_server**:

```
$ login_server.exe --nodaemon
[2010:04:18: 16:46:42.515]: Reading quests...
[2010:04:18: 16:46:42.515]: Connecting to the database...
[2010:04:18: 16:46:42.515]: Opening Dreamcast ports for connections.
[2010:04:18: 16:46:42.515]: Opening PSO for PC ports for connections.
[2010:04:18: 16:46:42.515]: Opening PSO for Gamecube ports for connections.
...
```

18. Start the **shipgate**:

```
$ shipgate.exe --nodaemon
[2010:04:18: 16:46:42.515]: Connecting to the database...
[2010:04:18: 16:46:42.515]: Clearing online ships...
[2010:04:18: 16:46:42.515]: Clearing online_clients...
...
```

19. Start the **ship\_server**:

```
$ ship_server.exe --nodaemon
[2010:04:18: 16:04:58.567]: Configured parameters:
[2010:04:18: 16:04:58.567]: Shipgate IP: 192.168.0.2
[2010:04:18: 16:04:58.568]: Shipgate Port: 3455
[2010:04:18: 16:04:58.568]: Number of Ships: 1
[2010:04:18: 16:04:58.568]: Ship Name: A random name
[2010:04:18: 16:04:58.568]: Ship IPv4 Host: use.public.ip
[2010:04:18: 16:04:58.568]: Ship IPv6 Host: Autoconfig or None
[2010:04:18: 16:04:58.568]: Base Port: 12001
[2010:04:18: 16:04:58.568]: Blocks: 2
[2010:04:18: 16:04:58.568]: Default Lobby Event: 0
[2010:04:18: 16:04:58.568]: Default Game Event: 0
[2010:04:18: 16:04:58.568]: Menu: es
[2010:04:18: 16:04:58.568]: Flags: 0x00000000

[2010:04:18: 16:04:58.727]: Starting server for ship A random name...
[2010:04:18: 16:04:58.731]: A random name: Connecting to shipgate...
...
```

  * If it says: Connection refused
    * Shipgate is OFF (turn ON)
    * Server does not found the IP
    * Port is not correct (must match both shipgate ports from XML)

19. Optional: Start the **patch\_server**:

```
$ patch_server.exe --nodaemon
...
```