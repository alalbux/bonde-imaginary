# BONDE Graphene

A GraphQL API for read and manipulate data on Bonde application.
Based on Python 3, Flask, Graphene and JWT authentication.

## Getting Started

First you'll need to get the source of the project. Do this by cloning the whole repository:

```
git clone https://github.com/nossas/bonde-graphene.git
cd bonde-graphene
```

It is good idea (but not required) to create a virtual environment for this project. We'll do this using [venv](https://docs.python.org/3/library/venv.html) module that built-in in Python3 to keep things simple, but you may also find something like [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) to be useful:

```
python3 -m venv env
source env/bin/activate
```

Now we can install our dependencies:

```
pip install -r requirements/development.txt
```

Run application on `http://127.0.0.1:3003/graphql`:

```
python manage.py runserver -p 3003
```

### Migrations


> **IMPORTANT:** To create a new migration, you need to create an object that inherits from `db.Model`, add the table name in the` SQLALCHEMY_MIGRATE_ONLY_TABLES`configuration that is in the file `bonde_graphene/database/__init__.py`


Creating a migration:

```
python manage.py db migrate -m "your migration name"
```

Performing migrations:

```
python manage.py db upgrade
```

For more information go to: https://github.com/miguelgrinberg/Flask-Migrate

### Run tests

```
DATABASE_URI=sqlite:///db.sqlite3 py.test -s -vv
```

With coverage:

```
DATABASE_URI=sqlite:///db.sqlite3 py.test -s -vv --cov=bonde_graphene tests/
```

### Configure your application

Here are some environment variables that you can use to configure your application and its default values:

- `DATABASE_URI`: postgres://monkey_user:monkey_pass@db.bonde.devel:5432/bonde
- `JWT_SECRET_KEY`: segredo123
- `JWT_EXPIRES`: 12