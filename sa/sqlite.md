Install SQLAlchemy

```shell
pip install sqlalchemy
```

Prepare and run

```python
# imports
from sqlalchemy import create_engine, Boolean, Column, ForeignKey, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship

# in memory db
DATABASE_URL = "sqlite://"
engine = create_engine(DATABASE_URL, echo=True)

# create session
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# get Base
Base = declarative_base()

# create user table class
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)

# migrate
Base.metadata.create_all(bind=engine)

# instantiate a new user
user_db = User(email = "user@example.com", hashed_password="password", is_active=True)

# create session
db = SessionLocal()

# save user to db
db.add(user_db)
db.commit()

# reload user from db
db.refresh(user_db)

# get user id
print(user_db.id)
```
