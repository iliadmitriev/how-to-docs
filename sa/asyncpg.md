# Install

```shell
pip install sqlalchemy\[asyncio\]
```

Create postgresql instanse with docker
```shell
# env file with variables
cat > .env << _EOF_
POSTGRES_HOST=192.168.10.1
POSTGRES_PORT=5432
POSTGRES_DB=auth
POSTGRES_USER=auth
POSTGRES_PASSWORD=authsecret
_EOF_
export $(cat .env | xargs)

docker run -d --name auth-postgres --hostname auth-postgres \
    -p 5432:5432 --env-file .env postgres:13.4-alpine3.14
```

# Prepare and run

```python
import asyncio

from sqlalchemy.ext.asyncio import create_async_engine
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from sqlalchemy import Boolean, Column, ForeignKey, Integer, String
from sqlalchemy.orm import relationship

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)

# postgresql db
DATABASE_URL = URL.create(
    drivername="postgresql+asyncpg",
    host="localhost",
    port=5432,
    username="auth",
    password="authsecret",
    database="auth"
)

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
