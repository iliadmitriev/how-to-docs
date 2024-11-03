# SQLAlchemy + Async

## install

Install SQLAlchemy, greenlet and databases async drivers

```shell
pip3 install sqlalchemy greenlet asyncpg asyncmy pymysql aiosqlite
```

## create databases

create docker instances of databases

```shell
cat > .env_postgres << _EOF
POSTGRES_DB=user
POSTGRES_USER=user
POSTGRES_PASSWORD=secret
_EOF

docker run -d --name user-postgres --hostname user-postgres \
           --env-file .env_postgres -p 5432:5432 postgres:13.4-alpine3.14

cat > .env_mysql << _EOF
MYSQL_ROOT_PASSWORD=rootsecret
MYSQL_DATABASE=user
MYSQL_USER=user
MYSQL_PASSWORD=secret
_EOF

docker run -d --name user-mariadb --hostname user-mariadb \
           --env-file .env_mysql -p 3306:3306 mariadb
```

## Run test scrypt

uncomment one of DATABASE_URL you wish to test

```python
import asyncio

from sqlalchemy.ext.asyncio import create_async_engine
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import URL, sessionmaker
from sqlalchemy import Boolean, Column, ForeignKey, Integer, String
from sqlalchemy.orm import relationship

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(length=120), unique=True, index=True)
    hashed_password = Column(String(length=150))
    is_active = Column(Boolean, default=True)

# db url
DATABASE_URL = URL.create(
    drivername="postgresql+asyncpg",
    host="localhost",
    port=5432,
    username="item",
    password="secret",
    database="item"
)

#DATABASE_URL = "postgresql+asyncpg://user:secret@192.168.10.1:5432/user"
#DATABASE_URL = "mysql+asyncmy://user:secret@192.168.10.1:3306/user"
#DATABASE_URL = "sqlite+aiosqlite://?cache=shared"
#DATABASE_URL = "sqlite+aiosqlite:///user.db?cache=shared"


# async engine
engine = create_async_engine(DATABASE_URL, echo=True)
# async session
async_session = sessionmaker(engine, expire_on_commit=False, autoflush=False, class_=AsyncSession)

# main coroutine
async def async_main():
    # migrate
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

    # create user
    user_db = User(email = "user@example.com", hashed_password="password", is_active=True)


    async with async_session() as session:
        async with session.begin():
            # add user
            session.add(user_db)
        # commit session
        await session.commit()
        # refresh user from db
        await session.refresh(user_db)

    print(user_db.id)

# run coroutine
asyncio.run(async_main())
```
