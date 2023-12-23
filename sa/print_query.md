
```python
import sqlalchemy as sa
from sqlalchemy.dialects import postgresql

stmt = sa.select(...)

print(stmt.compile(dialect=postgresql.dialect(), compile_kwargs={"literal_binds": True}))
```
