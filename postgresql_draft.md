b) PostgreSQL
-------------

**This is work in progress and doesn't yet work. There's [bug report](https://github.com/haiwen/seafile/issues/5) opened on Seafile side.**

Install PostgreSQL:

    sudo apt-get install -y postgresql-9.1 postgresql-client-9.1 python-psycopg2

Login to PostgreSQL:

    sudo su - postgres

PostgreSQL make use of system users -- we already created this one in previous step so we can carry on with user assignment and databases creation:

    psql
    postgres=# CREATE USER seafile WITH PASSWORD '$password';
    postgres=# CREATE DATABASE "ccnet-db" OWNER seafile;
    postgres=# CREATE DATABASE "seafile-db" OWNER seafile;
    postgres=# CREATE DATABASE "seahub-db" OWNER seafile;
    postgres=# \q
    exit

    # Remember to change $password to some real, secure value

### Configure ccnet to use PostgreSQL:

    cd /home/seafile
    sudo -u seafile -H vim seafile/ccnet/ccnet.conf

Append to file following configuration:

    [Database]
    ENGINE=postgresql
    HOST=localhost
    USER=seafile
    PASSWD=$password
    DB=ccnet-db
    UNIX_SOCKET=/var/run/postgresql/.s.PGSQL.5432
    
    # Remember to change $password to real value

### Configure seafile to use PostgreSQL:

    sudo -u seafile -H vim data/seafile.conf

Replace existing database section with following:

    [database]
    type=postgresql
    host=localhost
    user=seafile
    password=$password
    db_name=seafile-db
    unix_socket=/var/run/postgresql/.s.PGSQL.5432
    
    # Remember to change $password to real value

### Configure seahub to use PostgreSQL:

    sudo -u seafile -H vim seafile/seahub_settings.py

Append to file following configuration:

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'USER': 'seafile',
            'PASSWORD': '$password',
            'NAME': 'seahub-db',
            'HOST': '/var/run/postgresql/.s.PGSQL.5432',
        }
    }

    # Remember to change $password to real value

### Create DB structures:

    sudo -u seafile -H seafile/seafile-server-1.4.5/seafile.sh start
