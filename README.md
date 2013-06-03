# Nagios or Icinga Event broker for Canopsis #

## Requirements ##

Debian like (Debian, Ubuntu ...):

    apt-get install build-essential git-core


Redhat like (Centos ..):

    yum groupinstall "Development Tools"
    yum install git-core


## Download and Build ##

via git:

    git clone https://github.com/capensis/canopsis-nagios.git
    cd canopsis-nagios
    make

via HTTP:

    wget --no-check-certificate  https://github.com/capensis/canopsis-nagios/tarball/master -O canopsis-nagios.tgz
    tar xfz canopsis-nagios.tgz
    cd canopsis-*
    make

## Install ##

Copy NEB in your Nagios/Icinga installation:

    sudo cp neb2amqp.o /usr/local/nagios/bin/


## Configure ##

Edit your Nagios/Icinga configuration (nagios.cfg):

    event_broker_options=-1
    broker_module=/usr/local/nagios/bin/neb2amqp.o name=Central host=localhost

Options:

    host =          AMQP Server (127.0.0.1)
    port =          AMQP Port (5672)
    userid =        AMQP login (guest)
    password =      AMQP password (guest)
    virtual_host =  AMQP Virtual host (canopsis)
    exchange_name = AMQP Exchange (canopsis.events)
    name =          Poller name (Central)
    connector =     Connector name (nagios) (you can type "icinga" for icinga)
    max_size =      Maximum message size to send to the AMQP bus (8192)
    cache_file =    File in which faulty messages are stored (/usr/local/nagios/var/canopsis.cache)
                    (note: if we cannot read/create the file, the cache will
                    only run in memory)
    cache_size =    Number of messages to store in cache (1000)
    autosync =      Delay in seconds between two automatic sync of the cache into 'cache_file'.
                    If < 0 disable autosync (note: the cache will always be stored when the module
                    is unloaded). If = 0 cache every time (this is not recommended as it may consumes
                    lot of I/O) (default: 60)
    autoflush =     Delay in seconds between two automatic flush of the cache into the AMQP bus
                    if it is available (60)
    rate =          Delay in ms between two messages when depiling (5)
    flush =         Number of messages to send when depiling (-1: means it is calculated at runtime)
    purge =         If 'true', purge cache at startup. /!\ This will increase Nagios' startup time. (false)

If nagios.cfg is generated by other program, you can try to add in your nagios init script:

    CPS_NEB=$prefix/bin/neb2amqp.o
    CPS_NAME=$(hostname -s)
    CPS_SERVER="odeon.root.local"
    if [ -e $CPS_NEB ]; then
            grep "$CPS_NEB" $NagiosCfgFile 1> /dev/null
            if [ $? -ne 0 ]; then
                echo "Add NEB: '$CPS_NEB' to configuration file '$NagiosCfgFile'"
                echo "broker_module=$CPS_NEB name=$CPS_NAME host=$CPS_SERVER" >> $NagiosCfgFile
            fi
    fi
