---
title: "Let's Combine Keyboard Recommendations with LLMs so They Actually Understand What the Hell You’re Writing [DRAFT]"
description: "Markov Model"
date: "October 11 2025"
---

<img src="https://i.sstatic.net/w5J1V.png" alt="ilustrasi markov chain pada keyboard">
<br/>

In this blog we will create a fully context reccomendation using markov chain and LLM

According to [brilliant.org](https://brilliant.org/wiki/markov-chains/), A Markov chain is a mathematical system that experiences transitions from one state to another according to certain probabilistic rules. The defining characteristic of a Markov chain is that no matter how the process arrived at its present state, the possible future states are fixed. In other words, the probability of transitioning to any particular state is dependent solely on the current state and time elapsed. The state space, or set of all possible states, can be anything: letters, numbers, weather conditions, baseball scores, or stock performances.

Mari kita inisiasi riwayat dari kata yang sudah didapat, diberikan dataset kumpulan _sequence_ kata sebagai berikut

```text
Aku suka kamu, tapi kamu suka dia.
Kamu suka dia, tapi aku suka kamu.
Aku suka kamu.
Kamu suka dia.
Aku, kamu. Kamu, dia.
Tapi aku suka kamu. Tapi kamu suka dia.
Dia, kamu suka. Kamu, aku suka.
Tapi dia, kamu suka. Tapi kamu, aku suka.
Suka aku, suka kamu, suka dia.
Aku suka kamu.
Kamu suka dia.
Tapi.
Aku suka kamu, tapi kamu suka dia.
```

Dalam sebuah markov, penting untuk melihat persentase dari setiap kata yang ada, mari kita cari persentase dari setiap kata yang ada.
```python
import re
raw_text = """
Aku suka kamu tapi kamu suka dia
Kamu suka dia tapi aku suka kamu
Aku suka kamu
Kamu suka dia
Aku kamu Kamu dia
Tapi aku suka kamu Tapi kamu suka dia
Dia kamu suka Kamu aku suka
Tapi dia kamu suka Tapi kamu aku suka
Suka aku suka kamu suka dia
Aku suka kamu
Kamu suka dia
Tapi
Aku suka kamu tapi kamu suka dia
""".lower()


seq = list(set(raw_text.split())) # ubah kolom value komentar
prob = []
for i in range(len(seq)):
  prob.append({
      seq[i]:(len(list(filter(lambda x:x == seq[i],raw_text.split())))/len(raw_text.split()))*100
  })
print(prob)
```

Hasil dari perhitungan persentase sebagai berikut,
```json
[{'dia': 15.151515151515152},
 {'suka': 28.78787878787879},
 {'aku': 15.151515151515152},
 {'kamu': 28.78787878787879},
 {'tapi': 12.121212121212121}]
```
Hasil diatas mempunyai arti bahwa kata "tapi" mempunyai persentase kemunculan 12% disusul dengan kata "dia" dan "aku" mempunyai persentase kemunculan sebanyak 15%, lalu yang tertinggi adalah "suka" dan "kamu" dengan persentase 28%

Mari kita generate kombinasi per 2 kata
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

Dilakukan pentotalan tiap 
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