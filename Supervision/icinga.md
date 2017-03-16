Supervision - Icinga
==
<br/>
TODO

#### Pré-installation

```bash
* cmake >= 2.6
* GNU make (make)
* C++ compiler which supports C++11 (gcc-c++ >= 4.7 on RHEL/SUSE, build-essential on Debian, alternatively clang++)
* pkg-config
* OpenSSL library and header files >= 0.9.8 (openssl-devel on RHEL, libopenssl1-devel on SLES11,
libopenssl-devel on SLES12, libssl-dev on Debian)
* Boost library and header files >= 1.41.0 (boost-devel on RHEL, libboost-all-dev on Debian)
* GNU bison (bison)
* GNU flex (flex) >= 2.5.35
* recommended: libexecinfo on FreeBSD (automatically used when Icinga 2 is
               installed via port or package)
* optional: MySQL (mysql-devel on RHEL, libmysqlclient-devel on SUSE, libmysqlclient-dev on Debian);
            set CMake variable `ICINGA2_WITH_MYSQL` to `OFF` to disable this module
* optional: PostgreSQL (postgresql-devel on RHEL, libpq-dev on Debian); set CMake
            variable `ICINGA2_WITH_PGSQL` to `OFF` to disable this module
* optional: YAJL (yajl-devel on RHEL, libyajl-dev on Debian)
* optional: libedit (libedit-devel on CentOS (RHEL requires rhel-7-server-optional-rpms
            repository for el7 e.g.), libedit-dev on Debian)
* optional: Termcap (libtermcap-devel on RHEL, not necessary on Debian) - only
            required if libedit doesn't already link against termcap/ncurses
* optional: libwxgtk2.8-dev or newer (wxGTK-devel and wxBase) - only required when building the Icinga 2 Studio

```

#### Installation

```bash
# groupadd icinga
# groupadd icingacmd
# useradd -c "icinga" -s /sbin/nologin -G icingacmd -g icinga icinga

# usermod -a -G icingacmd www-data

$ mkdir build && cd build
$ cmake ..
$ make
$ make install

apt-get install icingaweb2
apt-get install icingacli
apt-get install icingaweb2-module-monitoring icingaweb2-module-doc

http://X.X.X.X/icingaweb2/authentication/login

# generation  d'un token sur le serveur icinga
icingacli setup token create

# Installation plugins nagios
apt-get install nagios-plugins

```

#### Commande pratique
```bash

icinga2 feature list  

collab-tab:~/icinga2-2.6.0/build# icinga2 feature list
Disabled features: command compatlog debuglog gelf graphite ido-mysql ido-pgsql influxdb livestatus opentsdb perfdata statusdata syslog
Enabled features: api checker mainlog notification


# Activation des fonctionnalités
# ido-mysql, pour l’enregistrement des données dans MySQL. Et la fonctionnalité command, qui permettra notamment à l’interface web d’envoyer des commandes à Icinga
icinga2 feature enable ido-mysql command

ln -s /usr/local/etc/init.d/icinga2 /etc/init.d/icinga2


mysql -u root -p icinga_ido < mysql.sql

```

#### Avec dashing
```bash
# Installation environnement Ruby
# http://linoxide.com/monitoring-2/setup-monitoring-dashing-icinga2/

apt-get install gnupg2
gpg2 --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -L https://get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh

rvm install 2.2.3
rvm use 2.2.3
rvm default 2.2.3
rvm autolibs enable
gem install bundler
gem install dashing

cf : https://nodejs.org/en/download/
apt-get install nodejs npm


cd /usr/share/
git clone https://github.com/Icinga/dashing-icinga2.git
cd dashing-icinga2/
bundle install --system
icinga2 feature enable api

vim /etc/icinga2/conf.d/api-users.conf
    object ApiUser "dashing" {

    password = "icingadashingondebian"
    permissions = [ "status/query", "objects/query/*" ]

    }


Dans /usr/share/dashing-icinga2

```
