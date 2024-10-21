# this script is a simple sequence of commands that I ran to create a boinc server, it may not work
by: https://github.com/BOINC/boinc/wiki/Create-a-BOINC-server-(cookbook)

# prereq:
- `docker run -d --name=boinc -p 80:80 ubuntu tail -f /dev/null`
- `docker exec -it boinc bash`


```bash
apt update && apt upgrade -y

adduser boincadm  # in my case password "1"
apt install sudo
usermod -aG sudo boincadm
su boincadm   # use this command if you are logged out from shell
cd ~
```

# checkpoint: adduser; you can just skip this part
- `docker stop boinc`
- `docker commit boinc boinc_ck1_adduser`
- I entered the following 2 commands because I forgot to export the port
- `docker rm boinc`
- `*commands from prereq*`

# mysql-server libmysqlclient-dev  php-mysql
```bash
sudo apt install -y libssl-dev cron lsb-release git apache2 autoconf pkg-config libtool libcurl4-openssl-dev python3-pip python-is-python3 libapache2-mod-php php-cli php-gd php-xml nano wget
wget https://dev.mysql.com/get/mysql-apt-config_0.8.29-1_all.deb  #https://dev.mysql.com/downloads/repo/apt/
sudo dpkg -i mysql-apt-config_* #select 4 in question
sudo apt update
sudo apt install mysql-server libmysqlclient-dev php-mysql   #i set password same as sain in cookbook: foobar99
```

# setup db
```bash
sudo pip install -U pip
pip install mysqlclient
sudo service mysql start
sudo update-rc.d mysql defaults
sudo mysql_secure_installation   #i set password same as sain in cookbook: foobar99
```

```bash
sudo mysql
    CREATE USER 'boincadm'@'localhost' IDENTIFIED BY 'foobar99';
    GRANT ALL ON *.* TO 'boincadm'@'localhost';
```

# checkpoint: db; you can just skip this part
- `docker stop boinc`
- `docker commit boinc boinc_ck2_db`

# config apache
```bash
sudo usermod -a -G boincadm www-data
sudo service apache2 restart    #or sudo systemctl restart apache2
```
try visit http://localhost should give you default apache page

# install boinc
```bash
cd
git clone https://github.com/BOINC/boinc.git
cd boinc
./_autosetup
./configure --disable-client --disable-manager --enable-apps
make
```

# create project
```bash
cd ~/boinc/tools
./make_project --db_passwd foobar99 --url_base http://localhost test
cd ~/projects/test
sudo cp test.httpd.conf /etc/apache2/sites-enabled
sudo apache2ctl restart

crontab -e
# add this line:
## */5 * * * * cd /home/boincadm/projects/test; /home/boincadm/projects/test/bin/start --cron
bin/xadd
cd ~/projects/test/html/ops
htpasswd -c .htpasswd boincadm # type your password, for example: "1"

# replace text for PROJECT and COPYRIGHT_HOLDER
cd ~/projects/test/html/project
nano project.inc
```

# checkpoint: site; you can just skip this part
- `docker stop boinc`
- `docker commit boinc boinc_ck3_site`

# go to site and register
# add project in your boinc client

# lets add test application
```bash
cd ~/projects/test
cp ~/boinc/apps/uppercase_in templates
cp ~/boinc/apps/uppercase_out templates
```

```bash
nano ~/projects/test/config.xml
# add this in the <daemons> section:
      <daemon>
          <cmd>sample_trivial_validator --app uppercase</cmd>
      </daemon>
      <daemon>
          <cmd>sample_assimilator --app uppercase</cmd>
      </daemon>
```

# Create an app version
```bash
cd ~/projects/test/apps
mkdir -p uppercase/1.0/x86_64-pc-linux-gnu

cd ~/projects/test/apps/uppercase/1.0/x86_64-pc-linux-gnu
cp ~/boinc/apps/uppercase .
cp ~/boinc/apps/version.xml .

cd ~/projects/test
bin/update_versions

cd ~/projects/test
bin/stop
bin/start

# and testing...
cd ~/projects/test
nano infile     #add some mixed-case text
bin/demo_submit uppercase infile
```