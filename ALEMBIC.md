# Alembic creation and setup from a scratch

---

1. install virtual environment and activate it

```shell
cd $PROJECT_ROOT
python3 -m venv venv
source venv/bin/activate
```

2. create package dir and files

```shell
mkdir profiles
touch profiles/__init__.py
touch setup.py
touch README.md
```

this will result in

```shell
./
├── README.md
├── profiles
│   └── __init__.py
└── setup.py
```

3. modify `setup.py` file (minimum)

```shell
from setuptools import setup, find_packages
from os.path import join, dirname

setup(
    name='profiles',
    version='1.0.0',
    packages=find_packages(),
    long_description=open(join(dirname(__file__), 'README.md')).read(),
)
```

4. edit `README.md`
5. create and edit `profiles/models.py` file

```python
from sqlalchemy import Column, Integer, String, Date
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()


class Profile(Base):
    __tablename__ = 'profile'

    id = Column('id', Integer, primary_key=True, autoincrement=True)
    firstname = Column('firstname', String(100))
    surname = Column('surname', String(100))
    user_id = Column('user_id', Integer, index=True, unique=True)
    birthdate = Column('birthdate', Date)
    avatar = Column('avatar', String(200))
```

6. install pip packages and editable package

```shell
pip install alembic sqlalchemy psycopg2
pip install -e .
```

7. generate alembic setup

```shell
cd profiles
alembic init alembic
```

8. create and run test postgresql instance

```shell
docker run -d -p 5432:5432 -e POSTGRES_DB=profile \
    -e POSTGRES_USER=profile -e POSTGRES_PASSWORD=profilesecret \
    --name profile-postgres --hostname profile-postgres \
    postgres:alpine
```

9. edit alembic settings `alembic/env.py` lines 10, 20
   (setup models metadata and database connection)

https://github.com/iliadmitriev/profile/blob/ae101a57d25eb08b91dc6e6416c819c8cdbf7e2d/alembic/env.py#L17-L18

```python
...
config = context.config
config.set_main_option('sqlalchemy.url', 'postgres://profile:profilesecret@localhost:5432/profile')
...
from profiles.models import Base
target_metadata = Base.metadata
...
```

10. generate alembic revision

```shell
alembic revision --autogenerate -m "create table profile"
```

this will create revision file `profile/alembic/versions/{revision_hash}_create_table_profile.py`
check this file

11. show sql script of revisions

```shell
alembic upgrade head --sql
```

12. upgrade database (migrate)

```shell
alembic upgrade head
```

### Useful links

1. [Setuptool packaging](https://packaging.python.org/tutorials/packaging-projects/)
2. [Writing setup.py script](https://docs.python.org/3/distutils/setupscript.html)
3. [SQLAlchemy ORM tutorial](https://docs.sqlalchemy.org/en/13/orm/tutorial.html)
4. [Alembic tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html)
5. [Alembic autogenerate revisions](https://alembic.sqlalchemy.org/en/latest/autogenerate.html)
