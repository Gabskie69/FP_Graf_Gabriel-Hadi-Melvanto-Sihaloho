# FP_Graf_Gabriel-Hadi-Melvanto-Sihaloho

# 🤖 E-Commerce Chatbot & Customer Insights Knowledge Graph

Dokumen ini menjelaskan rancangan filosofi, skema data, dan panduan langkah-demi-langkah dalam membangun **Knowledge Graph** berbasis **Neo4j** untuk pemetaan perilaku pelanggan (*customer behavior*), interaksi chatbot, serta hasil konversi pembelian pada platform e-commerce.

---

## 🗺️ 1. Graph Overview & Design Philosophy

Knowledge Graph ini dirancang untuk memodelkan hubungan dinamis antara pelanggan, perilaku penjelajahan (*browsing*), interaksi dengan chatbot, produk yang diminati, serta hasil akhir dari proses transaksi. 

Dengan struktur graf, sistem ini mampu menjawab pertanyaan analitis kompleks seperti:
* 📉 *"Pelanggan mana saja yang berpotensi churn berdasarkan pola penjelajahan tinggi tanpa adanya riwayat pembelian?"*
* 🎯 *"Kategori produk apa yang paling diminati oleh segmen pelanggan tertentu?"*
* 💬 *"Bagaimana interaksi chatbot memengaruhi tingkat konversi (purchase conversion) pelanggan?"*
* 👥 *"Pelanggan mana saja yang memiliki kemiripan pola perilaku (ML clustering)?"*

### 🧠 Mengapa Menggunakan Knowledge Graph untuk Chatbot?
Basis data tradisional melihat pelanggan sebagai baris-baris data yang terisolasi. Sebaliknya, **Knowledge Graph melihat mereka sebagai entitas yang saling terhubung**. 

Sebagai contoh, seorang pelanggan yang menjelajahi kategori *Electronics* dan menanyakan *"kapan estimasi waktu pengiriman"* kepada chatbot secara semantik akan langsung terhubung dengan pelanggan lain yang memiliki pola serupa dan berakhir melakukan pembelian. Konteks relasional yang kaya inilah yang memungkinkan chatbot memberikan respons yang **personalized** dan **graph-aware**.

---

## 🏷️ 2. Node Types (Entities)

Berikut adalah daftar Entitas (Node) beserta properti kunci yang digunakan di dalam graf:

| Node Label | Description | Key Properties |
| :--- | :--- | :--- |
| **`Customer`** | Merepresentasikan profil pembeli tunggal. | `id`, `age`, `gender`, `location`, `annual_income` |
| **`Product`** | Kategori atau item produk tertentu. | `category`, `avg_price` |
| **`Purchase`** | Sebuah event transaksi atau pembelian tunggal. | `date`, `price`, `category` |
| **`BrowsingSession`**| Aktivitas sesi penjelajahan produk oleh pelanggan. | `timestamp`, `category` |
| **`Review`** | Ulasan atau testimoni produk dari pelanggan. | `text`, `rating`, `sentiment` |
| **`ChatSession`** | Sesi interaksi percakapan dengan chatbot. | `session_id`, `intent`, `resolved` |
| **`Intent`** | Niatan atau klasifikasi query pengguna ke chatbot. | `name` *(e.g., track_order, refund, recommend)* |
| **`Location`** | Entitas geografis tempat tinggal pelanggan. | `city` |
| **`Segment`** | Klaster kelompok pelanggan berbasis Machine Learning. | `segment_id`, `label` |

---

## 🔗 3. Relationship Types (Edges)

Hubungan antar entitas didefinisikan melalui relasi (Edge) berbobot dan berproperti berikut:

| Relationship | From → To | Properties | Description |
| :--- | :--- | :--- | :--- |
| **`LIVES_IN`** | `Customer` → `Location` | — | Lokasi domisili pembeli. |
| **`MADE_PURCHASE`** | `Customer` → `Purchase` | `date` | Aktivitas transaksi yang dipicu pelanggan. |
| **`PURCHASED_PRODUCT`**| `Purchase` → `Product` | `price` | Item produk yang dibeli dalam transaksi. |
| **`BROWSED`** | `Customer` → `Product` | `timestamp`, `count` | Riwayat produk yang dilihat/dijelajahi. |
| **`WROTE_REVIEW`** | `Customer` → `Review` | `date` | Ulasan yang ditulis oleh pelanggan. |
| **`REVIEW_ABOUT`** | `Review` → `Product` | — | Target produk yang diulas. |
| **`HAD_CHAT`** | `Customer` → `ChatSession` | `timestamp` | Sesi obrolan pelanggan dengan AI. |
| **`EXPRESSED_INTENT`** | `ChatSession` → `Intent` | `confidence` | Intent yang terdeteksi oleh chatbot. |
| **`BELONGS_TO_SEGMENT`**| `Customer` → `Segment` | `embedding_distance` | Hasil klastering segmen pelanggan. |
| **`SIMILAR_TO`** | `Customer` ↔ `Customer` | `similarity_score` | Tingkat kemiripan perilaku antar pelanggan (ML). |
| **`FREQUENTLY_BUYS`** | `Customer` → `Product` | `frequency`, `total_spent` | Agregasi kebiasaan belanja pelanggan. |

---

## 📊 4. Graph Schema Diagram

Diagram di bawah ini menggambarkan arsitektur hubungan data secara menyeluruh:

```mermaid
(Location) <─[LIVES_IN]── (Customer) ──[MADE_PURCHASE]──> (Purchase) ──[PURCHASED_PRODUCT]──> (Product)
                              │                                                                      ▲
                              ├──[BROWSED]───────────────────────────────────────────────────────────┤
                              │                                                                      │
                              ├──[WROTE_REVIEW]──> (Review) ──[REVIEW_ABOUT]────────────────────────┘
                              │
                              ├──[HAD_CHAT]──> (ChatSession) ──[EXPRESSED_INTENT]──> (Intent)
                              │
                              ├──[BELONGS_TO_SEGMENT]──> (Segment)
                              │
                              └──[SIMILAR_TO]──> (Customer)  [ML-derived]
