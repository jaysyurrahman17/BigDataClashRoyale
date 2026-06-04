
```markdown
# Analisis Big Data Komunitas Clash Royale & AI Chatbot Terintegrasi RAG
> **Proyek Akhir Mata Kuliah Kecerdasan Web dan Big Data**
> **Departemen Teknik Komputer, Institut Teknologi Sepuluh Nopember (ITS)**

---

## 📌 Deskripsi Proyek
Proyek ini membangun sebuah ekosistem *data engineering* berskala besar yang berfungsi untuk melakukan *crawling* otomatis, analisis sentimen, klasifikasi topik, dan ekstraksi keluhan kritis secara *real-time* dari komunitas game **Clash Royale** di 4 kanal YouTube terkemuka (*Clash Royale Official, Orange Juice, B-rad, dan SirTag*). 

Sistem ini mengintegrasikan pipa data otomatis (*automated data pipeline*) dengan arsitektur **RAG (Retrieval-Augmented Generation)** menggunakan LLM kelas berat untuk melayani pertanyaan analitis pengguna/dosen melalui **Bot Telegram**. Seluruh ekosistem dikemas menggunakan **Docker Compose** dan diamankan jalurnya dengan **Ngrok Static Domain** agar dapat berjalan secara mandiri (*self-hosted*) di latar belakang sejak komputer dinyalakan.

---

## 🏗️ Arsitektur Sistem & Alur Data

Ekosistem ini terbagi menjadi dua sub-sistem utama yang bekerja secara independen namun terhubung pada database yang sama:

### 1. Hulu: Automated Ingestion Pipeline (Jalur Penambangan Data)
```text
[Schedule Trigger] ➡️ [RSS Video Read] ➡️ [YouTube Data API v3]
                                                  ⬇️ (Komentar Mentah)
[MongoDB Storage] ⬅️ [Edit Fields] ⬅️ [JS Cleansing] ⬅️ [Groq AI (Llama 3.1 8B)]

```

### 2. Hilir: Serving & RAG Chatbot Pipeline (Jalur Interaksi Bot)

```text
[Telegram User Chat] ➡️ [Telegram Trigger] ➡️ [MongoDB Pipeline Aggregation]
                                                        ⬇️ ($unionWith & $facet)
[Telegram Response] ⬅️ [Groq AI (Llama 3.3 70B)] ⬅️ [JS Data Flattening & Math]

```

---

## 🚀 Fitur Utama

1. **Multi-Channel Scraper Otomatis:** Menambang ribuan komentar secara berkala dari 4 kanal YouTube berbeda secara simultan tanpa duplikasi data.
2. **AI Multiclass Classification:** Mengklasifikasikan komentar secara otomatis menggunakan LLM ke dalam 3 dimensi data:
* **Sentimen:** Positif, Negatif, Netral.
* **Kategori Komentar:** Keluhan/Nerf, Hiburan, Strategi.
* **Topik/Kartu:** Mendeteksi entitas nama kartu (e.g., *Mega Knight, Electro Wizard, Firecracker*).
* **Metrik Numerik:** Memberikan skor emosi berupa Skala Frustrasi Global (0-10).


3. **Business Intelligence (BI) Dashboard:** Visualisasi data real-time menggunakan **Metabase** untuk memantau polarisasi sentimen, tren popularitas kartu, dan grafik *gauge* untuk tingkat frustrasi komunitas.
4. **Zero-Hallucination RAG Chatbot:** Bot Telegram analitis yang dibekali konteks data dari 4.500+ baris data MongoDB. Menggunakan teknik *separation of concerns* (matematika dihitung deterministik oleh JavaScript, nalar bahasa diproses oleh LLM).

---

## 🛠️ Spesifikasi Teknologi (Tech Stack)

* **Orkestrasi & Otomatisasi:** n8n v1.x (Workflow-driven automation)
* **Database NoSQL:** MongoDB v6.x (Skema fleksibel untuk menampung JSON hasil analisis AI)
* **Business Intelligence:** Metabase (Native Query & Interactive Dashboard)
* **Mesin Inferensi AI:** Groq Cloud API
* *Model Crawler:* `llama-3.1-8b-instant` (Komputasi cepat untuk klasifikasi massal)
* *Model Chatbot:* `llama-3.3-70b-versatile` (Penalaran bahasa tingkat tinggi untuk menjawab pertanyaan analitis)


* **Infrastruktur & Jaringan:** Docker Desktop, Docker Compose, Ngrok Tunneling (Static Dev Domain)
* **Bahasa Pemrograman / Scripting:** JavaScript (ES6+) untuk *data pre-processing* dan *flattening* di n8n.

---

## 📂 Struktur Repositori

```text
├── docker-compose.yml               # Konfigurasi container n8n, MongoDB, Metabase, dan Ngrok
├── .gitignore                       # Proteksi kredensial agar tidak ter-push ke publik
├── README.md                        # Laporan proyek akhir (File ini)
├── Bot Telegram.json                # Backup workflow n8n untuk RAG Chatbot Telegram
├── Crawler - B-rad Fix.json         # Backup workflow n8n untuk crawler channel B-rad
├── Crawler - Clash Royale Fix.json   # Backup workflow n8n untuk crawler channel Clash Royale Official
├── Crawler - Orange Juice Fix.json  # Backup workflow n8n untuk crawler channel Orange Juice (komentar_oj)
└── Crawler - Sirtag Fix.json        # Backup workflow n8n untuk crawler channel Sirtag

```

---

## 🔧 Panduan Instalasi & Pengoperasian

### 1. Setup File Konfigurasi

Buat file `docker-compose.yml` di direktori proyek, lalu gunakan konfigurasi multi-container berikut:

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n_app
    environment:
      - WEBHOOK_URL=[https://gradation-undertake-perish.ngrok-free.dev](https://gradation-undertake-perish.ngrok-free.dev)  # Menggunakan dev domain statis ngrok
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
    restart: always

  mongodb:
    image: mongo:latest
    container_name: mongodb_db
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    restart: always

  metabase:
    image: metabase/metabase:latest
    container_name: metabase_app
    ports:
      - "3000:3000"
    volumes:
      - metabase_data:/metabase.db
    restart: always

  ngrok:
    image: ngrok/ngrok:latest
    container_name: ngrok_tunnel
    restart: always
    environment:
      - NGROK_AUTHTOKEN=MASUKKAN_TOKEN_NGROK_KAMU_DI_SINI
    command: http --domain=gradation-undertake-perish.ngrok-free.dev n8n:5678
    depends_on:
      - n8n

volumes:
  n8n_data:
  mongodb_data:
  metabase_data:

```

### 2. Menjalankan Server Ekosistem

Buka terminal/PowerShell di direktori proyek, lalu eksekusi perintah Docker Compose:

```bash
docker compose up -d

```

Ekosistem akan otomatis berjalan di latar belakang dan tersetel untuk otomatis menyala setiap kali komputer dihidupkan (*restart: always*).

---

## 📈 Rekayasa Pipa Data & Implementasi Kode (Engineering Deep Dive)

### 1. Sinkronisasi Data Lintas Kumpulan & Pembersihan Data Kotor (MongoDB Node)

Untuk memastikan data analitis yang diserahkan ke Bot Telegram 100% sinkron dan presisi dengan Visualisasi Metabase dari 4 *collection* terpisah (`komentar_official`, `komentar_oj`, `komentar_brad`, `komentar_sirtag`), diterapkan pipeline agregasi menggunakan operator `$unionWith` dan `$facet`. Kode ini juga menyaring data kotor/kosong (`$match` dan `$ifNull`) agar hasil kalkulasi genap dan valid sesuai dengan visualisasi grafik Metabase:

```json
[
  { "$unionWith": { "coll": "komentar_oj" } },
  { "$unionWith": { "coll": "komentar_brad" } },
  { "$unionWith": { "coll": "komentar_sirtag" } },
  {
    "$facet": {
      "total_komentar": [
        { "$match": { "sentimen_mayoritas": { "$in": ["Positif", "Negatif", "Netral"] } } },
        { "$count": "hitung" }
      ],
      "analisis_sentimen": [
        { "$match": { "sentimen_mayoritas": { "$in": ["Positif", "Negatif", "Netral"] } } },
        { "$group": { "_id": "$sentimen_mayoritas", "jumlah": { "$sum": 1 } } }
      ],
      "kategori_komentar": [
        { "$match": { "kategori_komentar": { "$in": ["Keluhan/Nerf", "Hiburan", "Strategi"] } } },
        { "$group": { "_id": "$kategori_komentar", "jumlah": { "$sum": 1 } } },
        { "$sort": { "jumlah": -1 } }
      ],
      "kartu_top": [
        { "$match": { "kartu_terpopuler": { "$nin": ["Tidak Ada", null, "", "-"] } } },
        { "$group": { "_id": "$kartu_terpopuler", "jumlah": { "$sum": 1 } } },
        { "$sort": { "jumlah": -1 } },
        { "$limit": 10 }
      ],
      "rata_frustrasi": [
        { "$match": { "sentimen_mayoritas": { "$in": ["Positif", "Negatif", "Netral"] } } },
        { 
          "$group": { 
            "_id": null, 
            "skala": { "$avg": { "$ifNull": ["$tingkat_frustrasi", 0] } } 
          } 
        }
      ]
    }
  }
]

```

### 2. Data Flattening & Pre-Processing (JavaScript Node)

Mencegah LLM dari melakukan kalkulasi persentase secara mandiri (karena sifat LLM yang probabilistik dan rentan salah hitung), node **Translator** berbasis JavaScript disisipkan untuk meratakan (*flattening*) struktur array JSON bersarang dari MongoDB menjadi laporan teks ringkas:

```javascript
let data = $input.first().json;
let text = "DATA STATISTIK CLASH ROYALE:\n";

let total = data.total_komentar[0]?.hitung || 0;
text += `- Total Seluruh Komentar: ${total}\n\n`;

text += `- Kategori Komentar (Ranking dari terbesar):\n`;
let kategori = data.kategori_komentar.sort((a,b) => b.jumlah - a.jumlah);
kategori.forEach((k, index) => {
    let persentase = ((k.jumlah / total) * 100).toFixed(1);
    text += `  ${index + 1}. ${k._id || 'Lainnya'} (${k.jumlah} komentar, ${persentase}%)\n`;
});
text += `\n`;

text += `- Topik/Kartu Terpopuler (Ranking):\n`;
data.kartu_top.forEach((k, index) => {
    text += `  ${index + 1}. ${k._id} (${k.jumlah} sebutan)\n`;
});
text += `\n`;

let frus = data.rata_frustrasi[0]?.skala || 0;
text += `- Tingkat Frustrasi Global: ${frus.toFixed(2)} / 10\n`;

return [{ json: { statistik_bersih: text } }];

```

### 3. Mengatasi Isu Parser Markdown Telegram API

Untuk mencegah runtuhnya sistem pengiriman pesan karena kegagalan pemisahan entitas (*parse entity error*) yang dipicu oleh karakter khusus asterisk murni (`*`) dari respons LLM, dipasang sistem *safety bypass regex* pada node **Answer**:

```javascript
let rawText = $input.first().json.choices[0].message.content;
try {
    let finalData = JSON.parse(rawText);
    let teksAman = finalData.jawaban.replace(/\*/g, 'x'); // Mengubah asterisk perkalian menjadi karakter 'x' aman bagi Telegram Markdown
    return [{ json: { teks_balasan: teksAman } }];
} catch (e) {
    return [{ json: { teks_balasan: "Maaf, format balasan AI sedang gangguan." } }];
}

```

---

## 🎯 Filter Data Cleansing (Metabase Dashboard)

Untuk menyelaraskan antara data mentah hasil penambangan kotor (*noise*) dengan visualisasi grafik eksekutif, dipasang kriteria saringan di Metabase pada panel **Keluhan Kritis**:

1. Mengisolasi `kategori_komentar` hanya pada nilai `Keluhan/Nerf`.
2. Menerapkan ambang batas kritis tingkat frustrasi pada parameter `tingkat_frustrasi >= 8`.
3. Mengeklusi meta-komentar halusinasi AI masa lalu dengan klausa kondisional mengecualikan teks (*Does not contain*): `"Komentar ini"`, `"Ditulis berdasarkan"`, serta mengeklusi pembahasan luar game seperti `"COC"`, `"Brawl Stars"`, dan `"Sprout"`.

---

## 👥 Tim Pengembang

* **Nama:** Muhammad Jaysyurrahman
* **NRP:** (Silakan isi NRP kamu di sini)
* **Program Studi:** S1 Teknik Komputer
* **Institusi:** Institut Teknologi Sepuluh Nopember (ITS)

```

```