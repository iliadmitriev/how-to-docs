

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

#### Disassemble classes, methods, function and other

```python
def len_str(s: str) -> int:
    return len(s)
    
dis.dis(len_str)
  2           0 LOAD_GLOBAL              0 (len)
              2 LOAD_FAST                0 (s)
              4 CALL_FUNCTION            1
              6 RETURN_VALUE
```
