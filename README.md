# idomatic-python
*Transforming Code into Beautiful, Idiomatic Python*

Notes from Raymond Hettinger's talk at pycon US 2013 [video](https://www.youtube.com/watch?v=OSGv2VnC0go), [slides](https://speakerdeck.com/pyconslides/transforming-code-into-beautiful-idiomatic-python-by-raymond-hettinger-1).

## Looping over a Range of Numbers

```python
for i in [0, 1, 2, 3, 4, 5]:
    print(i ** 2)
```

```python
# better
for i in range(6):
    print(i ** 2)
```

## Looping over a Collection

```python
colors = ['red', 'green', 'blue', 'yellow']
```

```python
for i in range(len(colors)):
    print(colors[i])
```

```python
# better
for color in colors:
    print(color)
```

## Looping Backwards

```python
colors = ['red', 'green', 'blue', 'yellow']
```

```python
for i in range(len(colors)-1, -1, -1):
    print(colors[i])
```

```python
# better
for color in reversed(colors):
    print(color)
```

## Looping over a Collection and Indices

```python
colors = ['red', 'green', 'blue', 'yellow']
```

```python
for i in range(len(colors)):
    print(i, '--->', colors[i])
```

```python
# better
for i, color in enumerate(colors):
    print(i, '--->', color)
```

- It's fast and beautiful and saves you from tracking the individual indices and incrementing them.
- Whenever you find yourself manipulating indices [in a collection], you're probably doing it wrong.

## Looping over Two Collections

```python
names = ['raymond', 'rachel', 'matthew']
colors = ['red', 'green', 'blue', 'yellow']
```

```python
n = min(len(names), len(colors))
for i in range(n):
    print(names[i], '--->', colors[i])
```

```python
# better
for name, color in zip(names, colors):
    print(name, '--->', color)
```

## Looping in Sorted Order

```python
colors = ['red', 'green', 'blue', 'yellow']
```

```python
# forward sorted order
for color in sorted(colors):
    print(color)
```

```python
# backwards sorted order
for color in sorted(colors, reverse=True):
    print(color)
```

## Custom Sort Order

```python
colors = ['red', 'green', 'blue', 'yellow']
```

```python
def compare_length(c1, c2):
    if len(c1) < len(c2): return -1
    if len(c1) > len(c2): return 1
    return 0

print(sorted(colors, cmp=compare_length))
```

- It is slow and unpleasant to write. Also, comparison functions are no longer available in Python 3.

```python
# better
print(sorted(colors, key=len))
```

## Call a Function until a Sentinel Value

```python
blocks = []
while True:
    block = f.read(32)
    if block == '':
        break
    blocks.append(block)
```

```python
# better
blocks = []
for block in iter(partial(f.read, 32), ''):
    blocks.append(block)
```

- `iter` takes two arguments. The first you call over and over again and the second is a sentinel value.

## Distinguishing Multiple Exit Points in Loops

```python
def find(seq, target):
    found = False
    for i, value in enumerate(seq):
        if value == target:
            found = True
            break
    if not found:
        return -1
    return i
```

```python
# better
def find(seq, target):
    for i, value in enumerate(seq):
        if value == target:
            break
    else:
        return -1
    return i
```

- Inside of every `for` loop is an `else`.

## Looping over Dictionary Keys

```python
d = {'matthew': 'blue', 'rachel': 'green', 'raymond': 'red'}
```

```python
for k in d:
    print(k)
```

```python
for k in d.keys():
    if k.startswith('r'):
        del d[k]
```

When should you use the second and not the first?

- When you're mutating the dictionary.
- If you mutate something while you're iterating over it, you're living in a state of sin and deserve what ever happens to you.

`d.keys()` makes a copy of all the keys and stores them in a list. Then you can modify the dictionary.
Note: in Python 3 to iterate through a dictionary you have to explicitly write: `list(d.keys())` because `d.keys()` returns a "dictionary view" (an iterable that provide a dynamic view on the dictionary’s keys). See [documentation](https://docs.python.org/3/library/stdtypes.html#dict-views).

## Looping over Dictionary Keys and Values

```python
# not very fast, has to re-hash every key and do a lookup
for k in d:
    print(k, '--->', d[k])
```

```python
# better
for k, v in d.items():
    print(k, '--->', v)
```

- `items()` is better as it returns an iterator.

## Construct a Dictionary from Pairs

```python
names = ['raymond', 'rachel', 'matthew']
colors = ['red', 'green', 'blue']
```

```python
d = dict(zip(names, colors))
# {'matthew': 'blue', 'rachel': 'green', 'raymond': 'red'}
```

## Counting with Dictionaries

```python
colors = ['red', 'green', 'red', 'blue', 'green', 'red']
```

```python
# simple and basic way to count
d = {}
for color in colors:
    if color not in d:
        d[color] = 0
    d[color] += 1
# {'blue': 1, 'green': 2, 'red': 3}
```

```python
# better
d = {}
for color in colors:
    d[color] = d.get(color, 0) + 1
```

```python
# more modern but has several caveats, better for advanced users
d = collections.defaultdict(int)
for color in colors:
    d[color] += 1
```

## Grouping with Dictionaries

```python
names = ['raymond', 'rachel', 'matthew', 'roger',
         'betty', 'melissa', 'judith', 'charlie']
```

```python
# grouping by name length
d = {}
for name in names:
    key = len(name)
    if key not in d:
        d[key] = []
    d[key].append(name)
# {5: ['roger', 'betty'], 6: ['rachel', 'judith'], 7: ['raymond', 'matthew', 'melissa', 'charlie']}
```

```python
d = {}
for name in names:
    key = len(name)
    d.setdefault(key, []).append(name)
```

```python
# better
import collections
d = collections.defaultdict(list)
for name in names:
    key = len(name)
    d[key].append(name)
```

## Is a Dictionary `popitem()` atomic?

```python
d = {'matthew': 'blue', 'rachel': 'green', 'raymond': 'red'}
```

```python
while d:
    key, value = d.popitem()
    print(key, '-->', value)
```

- `popitem` is atomic so you don't have to put locks around it to use it in threads.

## Linking Dictionaries

```python
defaults = {'color': 'red', 'user': 'guest'}

import argparse, os
parser = argparse.ArgumentParser()
parser.add_argument('-u', '--user')
parser.add_argument('-c', '--color')
namespace = parser.parse_args([])
command_line_args = {k:v for k, v in vars(namespace).items() if v}
```

```python
# use defaults, override them with environment variables, override them with command line arguments
d = defaults.copy()
d.update(os.environ)
d.update(command_line_args)
```

```python
# better
import collections
d = collections.ChainMap(command_line_args, os.environ, defaults)
```

## Improving Clarity

- Positional arguments and indicies are nice
- Keywords and names are better
- The first way is convenient for the computer
- The second corresponds to how human’s think

## Clarify Function Calls with Keyword Arguments

```python
twitter_search('@obama', False, 20, True)
```

```python
# better
twitter_search('@obama', retweets=False, numtweets=20, popular=True)
```

- This is slightly (microseconds) slower but is worth it for the code clarity and developer time savings.

## Clarify Multiple Return Values with `namedtuples`

```python
import doctest
doctest.testmod()
# (0, 4)  # is this good or bad? it's not clear.
```

```python
# better
from collections import namedtuple
TestResults = namedtuple('TestResults', ['failed', 'attempted'])

doctest.testmod()
# TestResults(failed=0, attempted=4)
```

- A named tuple is a subclass of tuple so they still work like a regular tuple, but are more friendly.

## Unpacking Sequences

```python
p = 'Raymond', 'Hettinger', 0x30, 'python@example.com'
```

```python
fname = p[0]
lname = p[1]
age = p[2]
email = p[3]
```

```python
# better, tuple unpacking
fname, lname, age, email = p
```

## Updating Multiple State Variables

```python
def fibonacci(n):
    x = 0
    y = 1
    for i in range(n):
        print(x)
        t = y
        y = x + y
        x = t
```

- `x` and `y` are state, and state should be updated all at once or in between lines that state is mis-matched and a common source of issues.
- Ordering matters.
- It's too low-level.

```python
# better
def fibonacci(n):
    x, y = 0, 1
    for i in range(n):
        print(x)
        x, y = y, x + y
```

- It is more high-level, doesn't risk getting the order wrong and is fast.

## Simultaneous State Updates

```python
tmp_x = x + dx * t
tmp_y = y + dy * t
tmp_dx = influence(m, x, y, dx, dy, partial='x')
tmp_dy = influence(m, x, y, dx, dy, partial='y')
x = tmp_x
y = tmp_y
dx = tmp_dx
dy = tmp_dy
```

- The `influence` function here is just an example function. The important part is how to manage updating multiple variables at once.

```python
# better
x, y, dx, dy = (x + dx * t,
                y + dy * t,
                influence(m, x, y, dx, dy, partial='x'),
                influence(m, x, y, dx, dy, partial='y'))
```

## Efficiency

- An optimization fundamental rule
- Don’t cause data to move around unnecessarily
- It takes only a little care to avoid O(n**2) behavior instead of linear behavior
- Don't move data around unecessarily.

## Concatenating Strings

```python
names = ['raymond', 'rachel', 'matthew', 'roger',
         'betty', 'melissa', 'judith', 'charlie']
```

```python
s = names[0]
for name in names[1:]:
    s += ', ' + name
print(s)
```

```python
# better
print(', '.join(names))
```

## Updating Sequences

```python
names = ['raymond', 'rachel', 'matthew', 'roger',
         'betty', 'melissa', 'judith', 'charlie']

del names[0]
names.pop(0)
names.insert(0, 'mark')
```

- The last two lines are signs you're using the wrong data structure

```python
# better
import collections
names = collections.deque(['raymond', 'rachel', 'matthew', 'roger',
               'betty', 'melissa', 'judith', 'charlie'])

del names[0]
names.popleft()
names.appendleft('mark')
```
## Decorators and Context Managers

- Helps separate business logic from administrative logic
- Clean, beautiful tools for factoring code and improving code reuse
- Good naming is essential.
- Remember the Spiderman rule: With great power, comes great responsibility!

## Using Decorators to Factor-out Administrative Logic

```python
def web_lookup(url, saved={}):
    if url in saved:
        return saved[url]
    page = urllib.urlopen(url).read()
    saved[url] = page
    return page
```

- It mixes business / administrative logic and is not reusable.

```python
# better
@cache
def web_lookup(url):
    return urllib.urlopen(url).read()
```

- Since Python 3.2 there is a decorator for this in the [standard library](https://docs.python.org/3/library/functools.html): [`functools.lru_cache`](https://docs.python.org/3/library/functools.html#functools.lru_cache).

## Factor-out Temporary Contexts

```python
# saving the old, restore the new
old_context = getcontext().copy()
getcontext().prec = 50
print(Decimal(355) / Decimal(113))
setcontext(old_context)
```

```python
# better
with localcontext(Context(prec=50)):
    print(Decimal(355) / Decimal(113))
```

## How to Open and Close Files

```python
f = open('data.txt')
try:
    data = f.read()
finally:
    f.close()
```

```python
# better
with open('data.txt') as f:
    data = f.read()
```

## How to use Locks

```python
# make a lock
lock = threading.Lock()
```

```python
lock.acquire()
try:
    print('Critical section 1')
    print('Critical section 2')
finally:
    lock.release()
```

```python
# better
with lock:
    print('Critical section 1')
    print('Critical section 2')
```

## Factor-out Temporary Contexts

```python
try:
    os.remove('somefile.tmp')
except OSError:
    pass
```

```python
# better
from contextlib import suppress
with suppress(OSError):
    os.remove('somefile.tmp')
```

To make your own `suppress` context manager in the meantime:

```python
@contextmanager
def ignored(*exceptions):
    try:
        yield
    except exceptions:
        pass
```

## Factor-out Temporary Contexts

```python
# temporarily redirect standard out to a file and then return it to normal
with open('help.txt', 'w') as f:
    oldstdout = sys.stdout
    sys.stdout = f
    try:
        help(pow)
    finally:
        sys.stdout = oldstdout
```

```python
# better
from contextlib import redirect_stdout
with open('help.txt', 'w') as f:
    with redirect_stdout(f):
        help(pow)
```

To roll your own `redirect_stdout` context manager:

```python
@contextmanager
def redirect_stdout(fileobj):
    oldstdout = sys.stdout
    sys.stdout = fileobj
    try:
        yield fileobj
    finally:
        sys.stdout = oldstdout
```

## Concise Expressive One-Liners

Two conflicting rules:

- Don’t put too much on one line
- Don’t break atoms of thought into subatomic particles

Raymond’s rule:

- One logical line of code equals one sentence in English

## List Comprehensions and Generator Expressions

```python
# tell what you do
result = []
for i in range(10):
    s = i ** 2
    result.append(s)
print(sum(result))
```

```python
# better, tell what you want
print(sum(i ** 2 for i in range(10)))
```
