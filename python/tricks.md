
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

#### Jinga one-liner test
```python
from jinja2 import Environment
import jinja2.filters

jinja2.filters.FILTERS['mul_by_10'] = lambda x: x * 10
tpl = Environment().from_string("{{ data.val | mul_by_10 }}")
tpl.render(data={'val': 10})

```

#### Access to clipboard data from python for MacOS

using MacOS commands `pbcopy` and `pbpaste`

```python
import subprocess

def getClipboardData():
    p = subprocess.Popen(['pbpaste'], stdout=subprocess.PIPE)
    retcode = p.wait()
    data = p.stdout.read()
    return data

def setClipboardData(data):
    p = subprocess.Popen(['pbcopy'], stdin=subprocess.PIPE)
    p.stdin.write(data)
    p.stdin.close()
    retcode = p.wait()
```

#### Flatten list

```python
l = [[1,2,3], [1,4,5], [7],[8]]
print(sum(l, list()))
```

```
[1, 2, 3, 1, 4, 5, 7, 8]
```

#### Profile python code

collect stats:
```python
import cProfile
with cProfile.Profile(builtins=False) as pr:
    main()
pr.dump_stats("tmp.stats")
```

vizualize:
```shell
brew install graphviz
pip3 install gprof2dot 
gprof2dot -f pstats tmp.stats | dot -Tsvg -o tmp.svg 
````

visualize with QCacheGrind:
```shell
pip3 install pyprof2calltree
pyprof2calltree -i tmp.stats -o tmp.calltree
brew install qcachegrind
qcachegrind tmp.calltree
```
