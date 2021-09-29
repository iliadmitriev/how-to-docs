

#### CamelCase to snake_case

```python
import re

def camel_to_snake(name):
  name = re.sub('(.)([A-Z][a-z]+)', r'\1_\2', name)
  return re.sub('([a-z0-9])([A-Z])', r'\1_\2', name).lower()

```


#### Logging

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```
