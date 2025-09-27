---
title: "A practical guide to building agents OpenAI"
description: "an instruction from OpenAI to create a AI Agent"
date: "Sept 27 2025"
---

> This post is part of my journey; Integrate Data Warehouse with AI Agent for Smart Analysis: Study Case ITERA Data Mart, 1/90 posts everyday, so wish me luck!

Artikel ini akan membahas terkait bagaimana cara merancang dan membuat AI Agent berdasarkan panduan dari OpenAI

_OpenAI, Inc. is an American artificial intelligence organization headquartered in San Francisco, California. It aims to develop "safe and beneficial" artificial general intelligence, which it defines as "highly autonomous systems that outperform humans at most economically valuable work"._

<iframe width="100%" height="500" src="https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf"></iframe>

------

# Part 1 - Apa itu AI Agent?

dalam buku panduan OpenAI terkait perancangan _AI Agent_ menyebutkan bahwa AI Agent adalah sistem yang berjalan secara mandiri dan autonomus pada tools yang sudah ditentukan.

> Agents are systems that independently accomplish tasks on your behalf.

> Think as workflow + LLM = AI Agent

Pada dasarnya, Agent dapat mengerjakan suatu alur pekerjaan user dengan bantuan kecerdasan ilmu dari LLM itu sendiri.
1. AI Agent menggunakan LLM untuk mengatur eksekusi alur kerja dan membuat keputusan.
2. AI Agent mempunyai akses pada _tools_ eksternal, dengan tujuan menjalankan aksi sesuai dengan _prompt_ dari user, dan ajaibnya, AI Agent dapat memilih tools secara dinamis sesuai dengan kebutuhan user, berkat bantuan kecerdasan dari LLM itu sendiri.

# Part 2 - Jadi, kapan seharusnya Anda membuat AI Agent?

Pada workflow yang tidak terlalu kompleks, seharusnya tidak memerlukan AI Agent untuk mengerjakan workflow tersebut. AI Agent dibutuhkan ketika proses workflow yang ada memerlukan pemikiran yang mendalam untuk mengambil sebuah keputusan.

**Studi Kasus: Payment Fraud Analysis**

Payment Fraud adalah Transaksi yang bersifat curang atau tidak atas persetujuan pemilik akun.
Basic workflow yang mungkin dilakukan pada sistem deteksi kecurangan pada pembayaran adalah sebagai berikut
1. Pengecekan identitas user
2. Pengecekan transaksi berdasarkan kriteria yang sudah ditentukan

Disinilah terdapat sebuah _miss_, bagaimana jika ternyata ada seseorang yang dapat mengakali sebuah _rules_ atau aturan yang ada, dan bisa lolos? disinilah AI Agent bekerja memperkaya fitur pada workflow dengan bantuan kecerdasan pada LLM.

Workflow yang sudah di-"enhance" oleh AI Agent pada sistem deteksi kecurangan pada pembayaran adalah sebagai berikut
1. Pengecekan identitas pengguna.
2. Pengecekan transaksi berdasarkan kriteria yang telah ditentukan (rule-based).
3. Evaluasi konteks transaksi berdasarkan riwayat dan kebiasaan pengguna. (AI Agent)
4. Pertimbangan terhadap pola-pola halus yang sulit ditangkap dengan aturan biasa. (AI Agent)
5. Penalaran layaknya seorang investigator berpengalaman untuk menggabungkan berbagai indikator. (AI Agent)
6. Identifikasi aktivitas mencurigakan meskipun tidak ada aturan yang dilanggar secara jelas. (AI Agent)

dengan bantuan AI Agent, workflow 3-5 dapat dilakukan secara dinamis kepada sistem.

# Part 3 - Membangun Pondasi AI Agent