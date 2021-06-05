
[title(emptyline_above_is_important)]: # (Python: defaultdict)

# defaultdict
If you're tired of encountering **KeyError**s when accessing a non-existent key in a `dict`, and instead want to have a predefined default value returned when accessing the existent key, `defaultdict` might be for you.


## The problem
Suppose you want to keep track of the scores of people in a game.  
And now suppose you directly access the dict using a key that doesn't exist in the `dict`, you get a **KeyError**.

```pytho
scores_dict = {}

# We expect that scores_dict['John'] is a list
scores_dict['John'].append(10)

# KeyError: John is not a key inside scores_dict
```


## Simple workaround
The native solution would be to just declare the key `John` inside `scores_dict` beforehand so as to avoid the **KeyError**.

```python
scores_dict = {}

if 'John' not in scores_dict:
    scores_dict['John'] = []
    
scores_dict['John'].append(10)
print(scores_dict)   # { 'John' : [10] }
```


## Using defaultdict
By using `defaultdict`, we pass a function as an argument, so that we are able to define what the default value will be returned in place of the **KeyError**.

```python
from collections import defaultdict

# Remember, you must pass a function, not a value
scores_dict = defaultdict(list)

scores_dict['John'].append(10)
# scores_dict is first evaluated, and returns a list before appending the value 10 to that list

print(scores_dict)   # { 'John' : [10] }
```


## Arbitrary default values
`list` is actually a constructor, and calling `list()` returns the empty list `[]`.  
But what if we wan't to set arbitrary values of our own? We use `lambda`

```python
from collections import defaultdict

# We are passing a function, that returns a value
names_dict = defaultdict(lambda _: 'James')  

names_dict[007] += " Bond"
# names_dict[007] is first evaluated, and returns 'James'

names_dict[009] = "Moneypenny"

print(names_dict)   # { 007: 'James Bond', 009: "Moneypenny"}
```


We've just saved ourselves a few lines, thanks to the syntactic sugar of Python. But more importantly, we are able to predictably set a default value for every new key in the `dict`.
