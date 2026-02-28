---
title: "Loop over rows in pandas"
description: "Re-learning pandas in a deeper way"
date: "Feb 28 2026"
---

# Loop over rows

For example, you may have a dataset where one column contains a comma-separated list—for instance, the `hobbies` column:

| id | name   | hobbies                          |
|----|--------|----------------------------------|
| 1  | virdio | listen to music, reading, sleeping |
| 2  | juan   | sleeping, fishing, eating          |
| 3  | dani   | reading, ski, gym                  |

and your goal is to transform that table into a more structured format like this:

| id | name   | hobbies                          |
|----|--------|----------------------------------|
| 1  | virdio | listen to music                  |
| 1  | virdio | reading                          |
| 1  | virdio | sleeping                         |
| 2  | juan   | fishing                          |
| 2  | juan   | eating                           |
| 3  | dani   | reading                          |
| 3  | dani   | ski                              |
| 3  | dani   | gym                              |

## Two approaches you can use

1. Vectorized operations 

Using `.apply(func)`, we can process each row without writing an explicit `for` loop:

```python
data['hobbies']=data['hobbies'].str.split(',').apply(
    lambda items:
    list(
        map(lambda item:item.strip(),items)
    )
)
```
This is generally the best practice.

2. Looping with a `for` loop

```python
data['hobbies'] = data['hobbies'].astype(object)

for idx in data.index:
    items = str(data.at[idx, 'hobbies']).split(',')
    cleaned = [i.strip() for i in items]
    data.at[idx, 'hobbies'] = cleaned
```

This approach is more time-consuming and often redundant.


Both work, but the final step is to explode the list into rows:
```python
data = data.explode('hobbies')
```

This creates a more granular and specific table.