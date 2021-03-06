Installation & Configuration
============================

Getting Started
---------------

Caravel is tested using Python 2.7 and Python 3.4+. Python 3 is the recommended version,
Python 2.6 won't be supported.


OS dependencies
---------------

Caravel stores database connection information in its metadata database.
For that purpose, we use the ``cryptography`` Python library to encrypt
connection passwords. Unfortunately this library has OS level dependencies.

You may want to attempt the next step
("Caravel installation and initialization") and come back to this step if
you encounter an error.

Here's how to install them:

For **Debian** and **Ubuntu**, the following command will ensure that
the required dependencies are installed: ::

    sudo apt-get install build-essential libssl-dev libffi-dev python-dev python-pip

For **Fedora** and **RHEL-derivatives**, the following command will ensure
that the required dependencies are installed: ::

    sudo yum upgrade python-setuptools
    sudo yum install gcc libffi-devel python-devel python-pip python-wheel openssl-devel

**OSX**, system python is not recommended. brew's python also ships with pip  ::

    brew install pkg-config libffi openssl python
    env LDFLAGS="-L$(brew --prefix openssl)/lib" CFLAGS="-I$(brew --prefix openssl)/include" pip install cryptography

**Windows** isn't officially supported at this point, but if you want to
attempt it, download `get-pip.py <https://bootstrap.pypa.io/get-pip.py>`_, and run ``python get-pip.py`` which may need admin access. Then run the following: ::

    C:\> pip install cryptography

    # You may also have to create C:\Temp
    C:\> md C:\Temp

Python virtualenv
-----------------
It is recommended to install Caravel inside a virtualenv. Python 3 already ships virtualenv, for
Python 2 you need to install it. If it's packaged for your operating systems install it from there
otherwise you can install from pip: ::

    pip install virtualenv

You can create and activate a virtualenv by: ::

    # virtualenv is shipped in Python 3 as pyvenv
    virtualenv venv
    . ./venv/bin/activate

On windows the syntax for activating it is a bit different: ::

    venv\Scripts\activate

Once you activated your virtualenv everything you are doing is confined inside the virtualenv.
To exit a virtualenv just type ``deactivate``.

Caravel installation and initialization
---------------------------------------
Follow these few simple steps to install Caravel.::

    # Install caravel
    pip install caravel

    # Create an admin user
    fabmanager create-admin --app caravel

    # Initialize the database
    caravel db upgrade

    # Create default roles and permissions
    caravel init

    # Load some data to play with
    caravel load_examples

    # Start the web server on port 8088
    caravel runserver -p 8088

    # To start a development web server, use the -d switch
    # caravel runserver -d


After installation, you should be able to point your browser to the right
hostname:port `http://localhost:8088 <http://localhost:8088>`_, login using
the credential you entered while creating the admin account, and navigate to
`Menu -> Admin -> Refresh Metadata`. This action should bring in all of
your datasources for Caravel to be aware of, and they should show up in
`Menu -> Datasources`, from where you can start playing with your data!

Configuration behind a load balancer
------------------------------------

If you are running caravel behind a load balancer or reverse proxy (e.g. NGINX
or ELB on AWS), you may need to utilise a healthcheck endpoint so that your
load balancer knows if your caravel instance is running. This is provided
at ``/health`` which will return a 200 response containing "OK" if the
webserver is running.


Configuration
-------------

To configure your application, you need to create a file (module)
``caravel_config.py`` and make sure it is in your PYTHONPATH. Here are some
of the parameters you can copy / paste in that configuration module: ::

    #---------------------------------------------------------
    # Caravel specific config
    #---------------------------------------------------------
    ROW_LIMIT = 5000
    CARAVEL_WORKERS = 16

    CARAVEL_WEBSERVER_PORT = 8088
    #---------------------------------------------------------

    #---------------------------------------------------------
    # Flask App Builder configuration
    #---------------------------------------------------------
    # Your App secret key
    SECRET_KEY = '\2\1thisismyscretkey\1\2\e\y\y\h'

    # The SQLAlchemy connection string to your database backend
    # This connection defines the path to the database that stores your
    # caravel metadata (slices, connections, tables, dashboards, ...).
    # Note that the connection information to connect to the datasources
    # you want to explore are managed directly in the web UI
    SQLALCHEMY_DATABASE_URI = 'sqlite:////path/to/caravel.db'

    # Flask-WTF flag for CSRF
    CSRF_ENABLED = True

    # Set this API key to enable Mapbox visualizations
    MAPBOX_API_KEY = ''

This file also allows you to define configuration parameters used by
Flask App Builder, the web framework used by Caravel. Please consult
the `Flask App Builder Documentation
<http://flask-appbuilder.readthedocs.org/en/latest/config.html>`_
for more information on how to configure Caravel.

Please make sure to change:

* *SQLALCHEMY_DATABASE_URI*, by default it is stored at *~/.caravel/caravel.db*
* *SECRET_KEY*, to a long random string

Database dependencies
---------------------

Caravel does not ship bundled with connectivity to databases, except
for Sqlite, which is part of the Python standard library.
You'll need to install the required packages for the database you
want to use as your metadata database as well as the packages needed to
connect to the databases you want to access through Caravel.

Here's a list of some of the recommended packages.

+---------------+-------------------------------------+-------------------------------------------------+
| database      | pypi package                        | SQLAlchemy URI prefix                           |
+===============+=====================================+=================================================+
|  MySQL        | ``pip install mysqlclient``         | ``mysql://``                                    |
+---------------+-------------------------------------+-------------------------------------------------+
|  Postgres     | ``pip install psycopg2``            | ``postgresql+psycopg2://``                      |
+---------------+-------------------------------------+-------------------------------------------------+
|  Presto       | ``pip install pyhive``              | ``presto://``                                   |
+---------------+-------------------------------------+-------------------------------------------------+
|  Oracle       | ``pip install cx_Oracle``           | ``oracle://``                                   |
+---------------+-------------------------------------+-------------------------------------------------+
|  sqlite       |                                     | ``sqlite://``                                   |
+---------------+-------------------------------------+-------------------------------------------------+
|  Redshift     | ``pip install sqlalchemy-redshift`` | ``redshift+psycopg2://``                        |
+---------------+-------------------------------------+-------------------------------------------------+
|  MSSQL        | ``pip install pymssql``             | ``mssql://``                                    |
+---------------+-------------------------------------+-------------------------------------------------+
|  Impala       | ``pip install impyla``              | ``impala://``                                   |
+---------------+-------------------------------------+-------------------------------------------------+
|  SparkSQL     | ``pip install pyhive``              | ``jdbc+hive://``                                |
+---------------+-------------------------------------+-------------------------------------------------+

Note that many other database are supported, the main criteria being the
existence of a functional SqlAlchemy dialect and Python driver. Googling
the keyword ``sqlalchemy`` in addition of a keyword that describes the
database you want to connect to should get you to the right place.


Caching
-------

Caravel uses `Flask-Cache <https://pythonhosted.org/Flask-Cache/>`_ for
caching purpose. Configuring your caching backend is as easy as providing
a ``CACHE_CONFIG``, constant in your ``caravel_config.py`` that
complies with the Flask-Cache specifications.

Flask-Cache supports multiple caching backends (Redis, Memcached,
SimpleCache (in-memory), or the local filesystem). If you are going to use
Memcached please use the pylibmc client library as python-memcached does
not handle storing binary data correctly. If you use Redis, please install
[python-redis](https://pypi.python.org/pypi/redis).

For setting your timeouts, this is done in the Caravel metadata and goes
up the "timeout searchpath", from your slice configuration, to your
data source's configuration, to your database's and ultimately falls back
into your global default defined in ``CACHE_CONFIG``.


Deeper SQLAlchemy integration
-----------------------------

It is possible to tweak the database connection information using the
parameters exposed by SQLAlchemy. In the ``Database`` edit view, you will
find an ``extra`` field as a ``JSON`` blob.

.. image:: _static/img/tutorial/add_db.png
   :scale: 30 %

This JSON string contains extra configuration elements. The ``engine_params``
object gets unpacked into the
`sqlalchemy.create_engine <http://docs.sqlalchemy.org/en/latest/core/engines.html#sqlalchemy.create_engine>`_ call,
while the ``metadata_params`` get unpacked into the
`sqlalchemy.MetaData <http://docs.sqlalchemy.org/en/rel_1_0/core/metadata.html#sqlalchemy.schema.MetaData>`_ call. Refer to the SQLAlchemy docs for more information.


Schemas (Postgres & Redshift)
-----------------------------

Postgres and Redshift, as well as other database,
use the concept of **schema** as a logical entity
on top of the **database**. For Caravel to connect to a specific schema,
there's a **schema** parameter you can set in the table form.


SSL Access to databases
-----------------------
This example worked with a MySQL database that requires SSL. The configuration
may differ with other backends. This is what was put in the ``extra``
parameter ::

    {
        "metadata_params": {},
        "engine_params": {
              "connect_args":{
                  "sslmode":"require",
                  "sslrootcert": "/path/to/my/pem"
            }
         }
    }


Druid
-----

* From the UI, enter the information about your clusters in the
  ``Admin->Clusters`` menu by hitting the + sign.

* Once the Druid cluster connection information is entered, hit the
  ``Admin->Refresh Metadata`` menu item to populate

* Navigate to your datasources

Note that you can run the ``caravel refresh_druid`` command to refresh the
metadata from your Druid cluster(s)


CORS
-----

The extra CORS Dependency must be installed:

    caravel[cors]


The following keys in `caravel_config.py` can be specified to configure CORS:


* ``ENABLE_CORS``: Must be set to True in order to enable CORS
* ``CORS_OPTIONS``: options passed to Flask-CORS (`documentation <http://flask-cors.corydolphin.com/en/latest/api.html#extension>`)

Upgrading
---------

Upgrading should be as straightforward as running::

    pip install caravel --upgrade
    caravel db upgrade

SQL Lab
-------
SQL Lab is a powerful SQL IDE that works with all SQLAlchemy compatible
databases out there. By default, queries are run in a web request, and
may eventually timeout as queries exceed the maximum duration of a web
request in your environment, whether it'd be a reverse proxy or the Caravel
server itself.

In the modern analytics world, it's not uncommon to run large queries that
run for minutes or hours.
To enable support for long running queries that
execute beyond the typical web request's timeout (30-60 seconds), it is
necessary to deploy an asynchronous backend, which consist of one or many
Caravel worker, which is implemented as a Celery worker, and a Celery
broker for which we recommend using Redis or RabbitMQ.

It's also preferable to setup an async result backend as a key value store
that can hold the long-running query results for a period of time. More
details to come as to how to set this up here soon.

Making your own build
---------------------

For more advanced users, you may want to build Caravel from sources. That
would be the case if you fork the project to add features specific to
your environment.::

    # assuming $CARAVEL_HOME as the root of the repo
    cd $CARAVEL_HOME/caravel/assets
    npm install
    npm run prod
    cd $CARAVEL_HOME
    python setup.py install

