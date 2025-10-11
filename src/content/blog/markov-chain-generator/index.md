---
title: "Membuat Rekomendasi Keyboard Markov Chain"
description: "Markov Model"
date: "October 11 2025"
---

<img src="https://i.sstatic.net/w5J1V.png">


given dataset
```bash
aku suka sama kamu tapi kamu suka sama dia
```

create a full nonsense sequence
```bash
aku suka sama kamu tapi kamu suka sama dia
```

get a percentation
```python
raw = "aku suka sama kamu tapi kamu suka sama dia"
seq = list(set(raw.split()))
prob = []
for i in range(len(q)):
  prob.append({
      q[i]:(len(list(filter(lambda x:x == q[i],seq.split())))/len(seq.split()))*100
  })
prob
```
```json
[{'dia': 11.11111111111111},
 {'aku': 11.11111111111111},
 {'kamu': 22.22222222222222},
 {'sama': 22.22222222222222},
 {'tapi': 11.11111111111111},
 {'suka': 22.22222222222222}]
```
that means dia = 11%

```python
combination = []
seq_list:list = seq.split()
for i in range(len(seq_list)):
  if i+1 > len(seq_list)-1:
    break
  combination.append((seq_list[i],seq_list[i+1]))
```

```json
[('aku', 'suka'),
 ('suka', 'sama'),
 ('sama', 'kamu'),
 ('kamu', 'tapi'),
 ('tapi', 'kamu'),
 ('kamu', 'suka'),
 ('suka', 'sama'),
 ('sama', 'dia')]
```

```python
d = {}
d_ = {}
for x in p_trans:
  if x[0] not in d:
    d[x[0]] = []
  d[x[0]].append(x[1])
  d_[x[0]] = len(d[x[0]])
```

```json
{'aku': 1, 'suka': 2, 'sama': 2, 'kamu': 2, 'tapi': 1}
```

```py
for i in p_trans:
  comb = list(filter(lambda x:x==i,p_trans))
  chain.append({i:len(comb)/d_[i[0]]})
```

```json
[{('aku', 'suka'): 1.0},
 {('suka', 'sama'): 1.0},
 {('sama', 'kamu'): 0.5},
 {('kamu', 'tapi'): 0.5},
 {('tapi', 'kamu'): 1.0},
 {('kamu', 'suka'): 0.5},
 {('suka', 'sama'): 1.0},
 {('sama', 'dia'): 0.5}]
```