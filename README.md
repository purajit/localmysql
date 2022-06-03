localmysql is a tool to easily spin up a locally running MySQL on any machine.

The instance can be easily configured and pre-populated.

# HAS THIS EVER HAPPENED TO YOU?
You're developing an application that uses MySQL. You want to create a new table, some fancy new APIs, all that jazz. You'd like to run some tests again and again, and make sure you start fresh each time. You know what makes that difficult? Having all those nasty datafiles lying around on your system. How could you ever be sure that you're starting with a clean database each time? Even if you get rid of the database, who knows? Maybe you changed some configs. Speaking of configs - you've definitely experienced how hard it is to keep in mind where to put your MySQL option files. Being nervous about it and constantly checking to make sure it's been picked up? Like, damn.

OK, now you've got your application setup. It's shiny, or matte - whatever floats your boat. And BAM - you need to update your application's MySQL version. Welp. You have no idea whether your tables will still work, or if anything needs to be changed. Maybe the way timestamps are managed has changed. Who knows? Or maybe you have two applications that use different MySQL versions. Normally, you'd have to keep two versions of MySQL installed - and trust me, one is bad enough. You'll probably end up just buying a new computer and starting from scratch. It doesn't need to be that way. You can just keep running `localmysql -v <version> -f` as many times as you'd like - once, twice, a hundred times, go nuts! Test all the versions you want, within seconds!

Then you go online and read all these fancy new features in MySQL v100. How would they work? How would things change? Would they perform as well as you hope? Well, again - simply run with the version you want, and you'll be able to run anything you'd like on this local MySQL v100. It's that easy. You know what you need to install? Just Docker. You know what you need to setup, or configure? Nothing.

Stop installing `mysql-server` on your computer. Stop being a chump. Install `localmysql`.

## If you like bullet points instead
* MySQL, especially, is a Kafkaesque thing to install. And the server is prone to getting into a bad state, and especially running in-test code can mess it up to a near-irreversible point
* No time spent creating the tools needed to be able to easily test and experiment. Reducing barrier to entry is big. Testing a new MySQL version is easy! Just set the `-v` option, and everything else remains the same!
* Much safer, easier, and intuitive startup and shutdown that won't put your computer or MySQL into a weird state
* [Not suggested, but] you could have MySQL integration/suite tests running in CI by having this container spun up

# Requirements
Docker, and that's about it!

# Installation
git clone https://github.com/purajit/localmysql.git
ln -Fs localmysql/localmysql /usr/local/bin/localmysql

## Updating
cd localmysql
git pull

# Usage
```# Help menu
localmysql

# Get available MySQL versions that can be used with localmysql
localmysql v

# Start localmysql with default version, no special configs
localmysql +

# Start localmysql with specific version, no special configs
localmysql + -v 5.7

# Start localmysql afresh - with no data carry over.
# If localmysql is already running, this will shut it down first
localmysql + -f

# Start localmysql with specific configurations. Specify the folder that contains the configs
# These configurations can include *.sql files to setup the database/run other setup SQL scripts
# (executed alphabetically)
#                              and *.cnf files to configure mysqld
#                              and a .env file that provides environment variables to override
#                                  settings like port, version, etc
# Note: the percona:5.6 container is broken and does not allow configuring [mysqld]. A patch is needed
localmysql + -c ./service/localmysqlconfigdir/

# Get a MySQL client connected to this localmysql instance
localmysql c

# Get a list of each non-empty table and how many rows they have. Useful to know whether
#   whether the database is clean or not
localmysql ts

# Get a bash prompt within the localmysql container, in case you want to dig into it more
localmysql b

# Stop local MySQL
localmysql -```

# Troubleshooting
## listen tcp 0.0.0.0:3306: bind: address already in use
This means you likely have another copy of MySQL (unrelated to localmysql) running. You can stop it with `sudo service mysql stop`. You can also completely remove the MySQL installation with the instructions below

# How to purge all other MySQL installations
This is highly recommended to keep your dev environment nice and clean
```sudo service mysql stop
sudo killall -KILL mysql mysqld_safe mysqld
sudo apt-get --yes purge mysql-server mysql-client
sudo apt-get --yes autoremove --purge
sudo apt-get autoclean
sudo deluser --remove-home mysql
sudo delgroup mysql
sudo rm -rf /etc/apparmor.d/abstractions/mysql /etc/apparmor.d/cache/usr.sbin.mysqld /etc/mysql /var/lib/mysql
/var/log/mysql* /var/log/upstart/mysql.log* /var/run/mysqld
sudo updatedb```

# TODOs
* allow tool to be installed via apt/brew
* flesh out documentation and steps for multiple platforms
* currently, there is a nasty sleep in the startup logic that is required because mysqld starts twice - once with `--initialize`, and once to actually startup the daemon. There doesn't seem to be a good way to differentiate between them, because connections are allowed in both cases, and queries can still run (but get interrupted inbetween). Hopefully there's something better.
* maybe better error messages for non-existent versions/files/used ports/etc
* configurability of passwords and other settings
