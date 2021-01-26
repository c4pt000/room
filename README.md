Install under Ubuntu 14.04
==========================

Basic System Setup
------------------

Required software:

* GIT
* MySQL Server 5.x
* Python 2.7
* Apache Web Server 2.x
* Memcached
* Build tools
* Python libraries

First install all required software:

    apt-get install mysql-server git apache2 memcached build-essential python-twisted \
                    python-mysqldb python-simplejson python-memcache python-libxml2 \
                    python-libxslt1

Our implementation of python-poker-network software uses /dev/random. Check it
if it produce a lot of data by default (cat /dev/random) - you need 1-5 Kb/s
minumum. If this do not happen you need to install rng-tools:

    apt-get install rng-tools

And configure it to use initial data from /dev/urandom. To do this edit file
/etc/default/rng-tools and insert following line at the end of the file:

    HRNGDEVICE=/dev/urandom

Restart rng-tools by:

    service rng-tools restart

Create new virtual host config file:

    vim /etc/apache2/sites-available/room.conf

With similar configuration:

    <VirtualHost *:80>
      ServerName room

      <Proxy *>
        Order deny,allow
        Allow from all
      </Proxy>
      ProxyPass /POKER_REST http://localhost:19384/POKER_REST retry=1
      ProxyPass / http://localhost:3000/ retry=1
    </VirtualHost>

Enable new settings:

    a2ensite room.conf
    a2enmod proxy proxy_http
    service apache2 restart

Alternative Install Poker Network Software
-----------------------------------------
```
sudo apt-get -y update; 

sudo apt-get -y install build-essential checkinstall git-core mysql-server \
                        python-pip python-twisted python-mysqldb \
                        python-mysqldb-dbg memcached python-memcache \
                        python-simplejson python-cjson python-cjson-dbg \
                        python-libxml2 python-libxml2-dbg python-lxml \
                        python-lxml-dbg python-libxslt1 python-libxslt1-dbg \
                        python-pypoker-eval gettext; 
sudo pip install git+git://github.com/pokermania/reflogging.git; 
sudo pip install git+git://github.com/pokermania/pokerdistutils.git; 
sudo pip install git+git://github.com/pokermania/pokerengine.git; 
sudo pip install git+git://github.com/pokermania/pokerpackets.git; 
git clone https://github.com/pokermania/pokernetwork.git pokernetwork; cd pokernetwork; sudo python setup.py configure -s mysql.root_user.password=password; sudo python setup.py configure -b; sudo python setup.py install; cd ..;
```



Install and configure Room
--------------------------

Install Module::Install from Ubuntu repositories:

    apt-get install libmodule-install-perl


Install Catalyst and Catalyst::Devel and other libs from Ubuntu repositories:

    sudo apt-get install libcatalyst-devel-perl libcatalyst-perl \
    libcrypt-ssleay-perl libobject-signature-perl


Next you will need to install all dependencies for this project. Dependencies 
will be download from CPAN. Since there are a lot of them it may make sense 
to enable auto installation of dependencies. To do this, simply bring up a 
CPAN shell:

    perl -MCPAN -e shell


And run these two commands in the CPAN shell:

    o conf prerequisites_policy follow
    o conf commit


To install all dependencies:

    perl Makefile.PL
    sudo make installdeps

Check output to see any errors. In my case I had to force-install DBIx::Class::FrozenColumns
due failing some tests:

    sudo cpan -fi DBIx::Class::FrozenColumns

Copy room-sample.conf to room.conf and edit it inserting correct login/passwords
for pythonpokernetwork database, bitcoind, twitter, GA, etc.

Import new schema:

    mysql -u root pythonpokernetwork < schema_dump.sql


Replace stock Python Poker Network source code with Room's code
---------------------------------------------------------------

Find where Python Poker Network and Python Poker Engine installed:

    find /usr -name "pokerclient.py"
    find /usr -name "pokerclient.py"

In my case it was /usr/lib/python2.7/dist-packages and /usr/share/pyshared. 
Rename existing pokernetwork and pokerengine folders into something else 
and create soft links to Room's code:

    cd /usr/lib/python2.7/dist-packages
    sudo mv pokerengine pokerengine.old
    sudo mv pokernetwork pokernetwork.old
    sudo ln -s ~/projects/Room/lib/ppn/pokerengine pokerengine
    sudo ln -s ~/projects/Room/lib/ppn/pokernetwork pokernetwork

And repeat for /usr/share/pyshared.

Install Python bitstring and oauth2 packages:

    sudo apt-get install python-pip 
    sudo pip install bitstring 
    sudo pip install oauth2

And restart poker server:

    sudo /etc/init.d/python-poker-network restart


Start Room
----------

From project's home folder run:

    ./script/room_server.pl -r -f 

Navigate to http://room/ or whatever domain you created for this project.
