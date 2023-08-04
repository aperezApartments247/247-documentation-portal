# Introduction
- This guide will help in getting a local CMS environment set up on [Ubuntu 20.04.1 LTS](https://releases.ubuntu.com/20.04/)
- This is working as of 06/03/2022

# Tools
* [PostgreSQL 11](https://www.postgresql.org/download/linux/ubuntu/)
* [Pyenv](https://github.com/pyenv/pyenv-installer) - A python version manager
* [Pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv) - A python virtualenv manager
* [Pip](https://linuxize.com/post/how-to-install-pip-on-ubuntu-20.04/#installing-pip-for-python-2) - Python Package Installer
* [RabbitMQ](https://www.rabbitmq.com/download.html) - Required for celery
* [MailHog](https://github.com/mailhog/MailHog) - SMTP


# Pre-Setup
- Add these lines at the bottom of your `~/.bashrc`
```
# Pyenv
export PYENV_ROOT="$HOME/.pyenv"
export PYENV_SHIMS="$PYENV_ROOT/shims"
export PYENV_BIN="$PYENV_ROOT/bin:$PYENV_SHIMS"
export PYENV_VIRTUALENV_DISABLE_PROMPT=1
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"

# Get unique paths
PATH="$PYENV_BIN:$PATH"
export PATH=`echo $PATH | tr ':' '\n' | sort | uniq | tr '\n' ':' | sed 's/.$//'`
```

# Pre-Requisites
- Install these packages 
```
$ sudo apt install build-essential libbz2-dev libpq-dev libreadline-dev libsqlite3-dev libssl-dev zlib1g-dev
```

# Installations
**PostgreSQL 11**
```
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install postgresql-11 postgresql-11-postgis-2.5 postgresql-11-postgis-2.5-scripts postgresql-client-11 postgresql-client-common postgresql-common
 
To verify: 
$ psql -V
psql (PostgreSQL) 11.14
```

**Python 2.7**
```
$ curl https://pyenv.run | bash
$ exec $SHELL

$ pyenv install 2.7-dev
$ pyenv global 2.7-dev

To verify: 
$ pyenv -v
pyenv 1.2.15

$ python2 -V
Python 2.7.18
```

**Python 2.7 environment**
```
$ pyenv virtualenv 2.7-dev cms-env

To verify:
$ pyenv virtualenvs
2.7-dev/envs/cms-env (created from /home/<username>/.pyenv/versions/2.7-dev)
cms-env (created from /home/<username>/.pyenv/versions/2.7-dev)
```

# Setup
_The folder structure defined in this guide will be specific to this setup. Any path modifications will be your own responsibility._ 
- **PSQL**
  - Create your database, user, and password.
    - _Remember the user and password for your `local.py`_
    ```
    $ sudo -u postgres createuser -Plsdr $USER
    $ sudo -u postgres createdb $USER
    $ sudo -u postgres createdb cms
    ```
  - Add the extensions
    ```
    $ psql -d cms -c "CREATE EXTENSION postgis"
    $ psql -d cms -c "CREATE EXTENSION hstore"
    ```
- **CMS**
  - Repo
  ```
    $ mkdir -p ~/git/cms
    $ git clone git@github.com:Apartments24-7/247.git ~/git/cms/src
  ```
  - Packages
  ```
    $ cd ~/git/cms/src
    $ pyenv activate cms-env
    $ pip install -r requirements.txt
  ```
  - Fieldkeys
  ```
    $ mkdir -p ~/git/cms/src/fieldkeys
    $ cd ~/git/cms/src/
    $ keyczart create --location=fieldkeys --purpose=crypt
    $ keyczart addkey --location=fieldkeys --status=primary --size=256
  ```
  - Pull latest db in to your local machine
    - _Note: **TS_USER** will have to be created by a TomServo admin_
  ```
    $ mkdir -p ~/git/cms/db
    $ TS_USER=<your_tom_servo_username>
    $ FIND_DB='find /data/db -name "`ls -St /data/db | head -1`"'
    $ LATEST_CMS_DB=$(ssh -p 2224 $TS_USER@office.apartments247.com $FIND_DB)
    $ scp -P 2224 $TS_USER@office.apartments247.com:$LATEST_CMS_DB ~/git/cms/db/.
  ```
  - Import the local db
  ```
    $ cd ~/git/cms/db
    $ zcat $(ls -St | head -1) | psql -U $USER -f - cms
  ```
  - Reimport the local db
  ```
    $ dropdb cms
    $ createdb cms
    $ cd ~/git/cms/db
    $ zcat $(ls -St | head -1) | psql -U $USER -f - cms
  ```
  - `local.py` setup
    - `$ touch ~/git/cms/src/247/settings/local.py` 
    - Get an example [here](https://github.com/Apartments24-7/247/wiki/CMS-Dev-Setup:-local.py)

# Common commands
- CMS Environment
  - `pyenv activate cms-env`
- CMS Path
  - `cd ~/git/cms/src/247`
- CMS Runserver
  - `python manage.py runserver`
- CMS Migrate
  - `python manage.py migrate --noinput`
- CMS Shell
  - `python manage.py shell_plus --quiet`
- MailHog
  - `MailHog &`
- RabbitMQ
  - `rabbitmq-server -q`
- CMS Celery
  - `celery -A celery_init worker --loglevel=debug --purge`
- PSQL
  - `sudo service postgresql start`