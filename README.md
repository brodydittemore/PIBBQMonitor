PIBBQMonitor
============

This is primarily a fork of @tilmil's PIBBQMonitor. I had issues with SQLite so I replaced it with Postgres. Additional contributions might come down the line.<br>

Raspberry PI with 3 meat thermometers<br>
Thanks to Paul Schow, Tomas Holderness, and Adafruit for the code started with for this project<br>
http://www.paulschow.com/2013/08/monitoring-temperatures-using-raspberry.html<br>
https://learn.adafruit.com/reading-a-analog-in-and-controlling-audio-volume-with-the-raspberry-pi/script<br>
https://github.com/talltom/PiThermServer<br>


Description
============
Raspberry PI project leveraging the MCP3008 ADC and Thermoworks TX-1001X-OP thermistor temperature probes to create a 3 sensor wireless BBQ Temperature monitor.<br>

Tested with Pi 2 running Weezy
logger.py - python script that reads ADC, calculates temp and logs to Postgres database<br>
alert.py - python script that monitors temperature against user defines thresholds and sends email alerts<br>
dbcleanup.sh - shell script to clear entries older than 48 hours<br>
build_db.sh - script to build Postgres databse<br>
/web/thermserv - node web server<br>
/web/index.html - web interface<br>

Dependencies
============
NodeJS<br>
Postgres<br>
node-static<br>
Twitter Bootstrap 3.2.0 (included in repository)<br>
Highcharts (included in repository)<br>
JQuery (included in repository)<br>
python-dev<br>
python-setuptools<br>
alsa-utils<br>
mpg321<br>

Setup
============
1.  Remove old node
    sudo rm -r bin.node /bin/node-waf include/node lib/node lib/pkgconfig.nodejs.pc share/man/man1/node.1
2.  Install node<br>
    curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
    sudo apt install nodejs
    Make sure node install was good by checking "sudo node -v". (tested with node version _____.<br>
3.  Install python and other packages: <br>
    sudo apt-get upgrade<br>
    sudo apt-get update<br>
    sudo apt-get install python-dev<br>
    sudo apt-get install python-setuptools<br>
    sudo easy_install rpi.gpio<br>
    sudo apt-get install alsa-utils<br>
    sudo apt-get install mpg321<br>
    sudo apt-get install npm<br>
4.  Install node dependencies with NPM<br>
    sudo npm install -g node-gyp<br>
    sudo npm install node-static<br>
5.  Install Postgres<br>
    sudo apt-get install postgresql-9.4<br>
    sudo cp /etc/postgresql/9.4/main/pg_hba.conf /etc/postgresql/9.4/main/pg_hba.conf.bak<br>
    sudo nano /etc/postgresql/9.4/main/pg_hba.conf<br>
    sudo cp /etc/postgresql/9.4/main/postgresql.conf /etc/postgresql/9.4/main/postgresql.conf.bak<br>
    sudo nano /etc/postgresql/9.4/main/postgresql.conf<br>
    sudo apt-get install pgadmin3<br>
    sudo -u postgres psql<br>
6.  Clone git repository to /home/pi.  i.e. logger.py should be in /home/pi directory<br>
    git clone git://github.com/brodydittemore/PIBBQMonitor.git<br>
    cp -R ~pi/PIBBQMonitor/* ~pi/<br>
7.  Run command to create DB<br>
    Build_db.sh<br>
8.  Disable any webserver that may already be running on port 80.<br>
    sudo update-rc.d apache2 disable<br>
9.  Setup the thermserv node app as a service<br>
    sudo cp /home/pi/thermserv_initfile /etc/init.d/thermserv<br>
    sudo chmod 755 /etc/init.d/thermserv<br>
    sudo update-rc.d thermserv defaults<br>
10.  Make cron scripts executable:<br>
    chmod +x /home/pi/logger.py<br>
    chmod +x /home/pi/dbcleanup.sh<br>
    chmod +x /home/pi/alert.py<br>
11. Config Postgresql and connection<br>
    sudo adduser postgres_user<br>
    sudo su - postgres_user<br>
    nano database.ini<br>
    nano config.py<br>
    sudo apt-get install python-psycopg2<br>
    sudo apt-get install build-dep<br>
    sudo apt-get install python-configparser<br>
12.  Execute "sudo crontab -e" and paste the following lines into your crontab.  Logger will log the temp to the DB every minute. The dbcleanup.sh will limit the DB to 24 hours worth of data:<br>
      */1 * * * * /home/pi/logger.py<br>
      0 * * * * /home/pi/dbcleanup.sh<br>
      */1 * * * * /home/pi/alert.py<br>
13. /etc/initd/thermserv start<br>
