<div align="center">

# CortexPay QRIS Template

**Template integrasi pembayaran QRIS yang simpel, tanpa framework, siap pakai.**

![License](https://img.shields.io/badge/license-MIT-168653?style=flat-square)
![Stack](https://img.shields.io/badge/stack-HTML%20%7C%20CSS%20%7C%20JS-168653?style=flat-square)
![Framework](https://img.shields.io/badge/framework-none-168653?style=flat-square)
![Responsive](https://img.shields.io/badge/responsive-mobile%20friendly-168653?style=flat-square)

[Quick Start](#quick-start) ·
[Cara Kerja](#cara-kerja) ·
[Konfigurasi](#konfigurasi) ·
[API Reference](#api-reference) ·
[Keamanan](#keamanan) ·
[Troubleshooting](#troubleshooting)

</div>

<!-- Taruh screenshot di sini, contoh:
<p align="center"><img src="docs/preview.png" width="720" alt="Preview"></p>
-->

---

## Tentang

Template ini bikin kamu bisa menerima pembayaran QRIS lewat [CortexPay](https://cortexpay.my.id) hanya dengan tiga file statis. Nggak perlu build, nggak perlu install dependency. Isi API key, buka di browser, selesai.

## Fitur

| | Fitur | Keterangan |
|---|---|---|
| **QRIS** | QRIS dinamis | Nominal sudah terkunci di dalam kode QR |
| **QR** | Tampil otomatis | QR code langsung dirender setelah transaksi dibuat |
| **Rp** | Format Rupiah | Input dan tampilan nominal otomatis rapi |
| **ID** | Order ID otomatis | Bisa diubah manual kalau perlu |
| **Poll** | Cek status otomatis | Polling tiap 3 detik sampai pembayaran terdeteksi |
| **UI** | Responsif | Nyaman dipakai di HP maupun desktop |
| **Err** | Error handling | Pesan error jelas kalau API gagal |

## Quick Start

**1. Download**

```bash
git clone <repository-url>
cd cortexpay-integration-template
```

**2. Isi API key**

Buka `app.js`, lalu ganti konfigurasinya:

```javascript
const CONFIG = {
  API_BASE_URL: 'https://cortexpay.my.id',
  API_KEY: 'YOUR_API_KEY_HERE',  // ganti dengan API key kamu
  MERCHANT_ID: null
};
```

<details>
<summary><b>Cara dapat API key</b></summary>

1. Login ke dashboard CortexPay
2. Buka menu **API Keys**
3. Klik **Buat API Key Baru**
4. Salin lalu tempel ke `app.js`

</details>

**3. Jalankan**

Buka `index.html` langsung di browser, atau pakai local server:

```bash
# Python
python -m http.server 3000

# Node.js
npx serve

# PHP
php -S localhost:3000
```

Lalu buka `http://localhost:3000`.

## Cara Kerja

<p align="center">
  <img src="docs/flow.svg" alt="Alur pembayaran: buat transaksi, tampil QRIS, pembeli bayar, konfirmasi" width="100%">
</p>

| Langkah | Proses | Detail |
|:---:|---|---|
| 1 | Buat transaksi | Pembeli isi nominal, lalu halaman mengirim `POST /pay/v1/create` |
| 2 | Tampil QRIS | API membalas `qris_payload`, QR code langsung dirender |
| 3 | Pembeli bayar | Scan QR dari e-wallet atau m-banking apa pun yang mendukung QRIS |
| 4 | Konfirmasi | Halaman cek `GET /pay/v1/{external_id}` tiap 3 detik sampai status `paid` |

### Langkah pemakaian

1. **Nominal**: isi jumlah pembayaran (minimal Rp 1.000)
2. **Order ID**: terisi otomatis, boleh diubah
3. **Deskripsi**: keterangan pembayaran (opsional)
4. Klik **Buat QRIS**

Setelah QR muncul, kamu bisa:

- Scan QR langsung dari e-wallet atau m-banking
- Klik **Salin QRIS String** untuk copy manual
- Klik **Transaksi Baru** untuk reset form

## Konfigurasi

| Opsi | Default | Fungsi |
|---|---|---|
| `API_BASE_URL` | `https://cortexpay.my.id` | Ganti kalau kamu pakai CortexPay self-hosted |
| `API_KEY` | `YOUR_API_KEY_HERE` | API key dari dashboard |
| `MERCHANT_ID` | `null` | Opsional |

<details>
<summary><b>Ganti warna tema</b></summary>

Edit `styles.css`:

```css
.btn-primary {
  background: #168653;
}

header h1 {
  color: #168653;
}
```

</details>

<details>
<summary><b>Matikan payment polling</b></summary>

Comment baris ini di `app.js`:

```javascript
// startPaymentPolling(externalId);
```

</details>

<details>
<summary><b>Tambah logo</b></summary>

`index.html`:

```html
<header>
  <img src="logo.png" alt="Logo" class="logo">
  <h1>QRIS Payment</h1>
  <p>Scan QRIS untuk pembayaran</p>
</header>
```

`styles.css`:

```css
.logo {
  width: 80px;
  height: auto;
  margin-bottom: 16px;
}
```

</details>

<details>
<summary><b>Tambah suara notifikasi</b></summary>

```javascript
function paymentSuccess() {
  const audio = new Audio('success.mp3');
  audio.play();

  // ... kode yang sudah ada
}
```

</details>

<details>
<summary><b>Tambah notifikasi WhatsApp</b></summary>

```javascript
function paymentSuccess() {
  const phone = '6281234567890';
  const message = 'Pembayaran berhasil!';
  window.open(`https://wa.me/${phone}?text=${encodeURIComponent(message)}`);

  // ... kode yang sudah ada
}
```

</details>

## Struktur File

```
cortexpay-integration-template/
├── index.html     # Halaman utama
├── styles.css     # Tampilan
├── app.js         # Logic dan integrasi API
├── docs/
│   └── flow.svg   # Diagram alur untuk README
└── README.md      # Dokumentasi
```

## Keamanan

> [!WARNING]
> **Jangan hardcode API key di production.** Kalau API key ditaruh di `app.js`, siapa pun yang buka halaman kamu bisa melihatnya. Pakai backend proxy dan simpan key di environment variable.

<details>
<summary><b>Contoh backend proxy (Node.js + Express)</b></summary>

Frontend (`app.js`):

```javascript
async function createPayment(amount, externalId, description) {
  const response = await fetch('/api/create-payment', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ amount, externalId, description })
  });
  return await response.json();
}
```

Backend (`backend.js`):

```javascript
app.post('/api/create-payment', async (req, res) => {
  const { amount, externalId, description } = req.body;

  const response = await fetch('https://cortexpay.my.id/pay/v1/create', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-API-Key': process.env.CORTEXPAY_API_KEY
    },
    body: JSON.stringify({ amount, externalId, description })
  });

  res.json(await response.json());
});
```

</details>

## API Reference

### Create Payment

`POST /pay/v1/create`

| Header | Nilai |
|---|---|
| `X-API-Key` | API key kamu |
| `Content-Type` | `application/json` |

Request:

```json
{
  "external_id": "ORDER-001",
  "amount": 50000,
  "description": "Pembayaran produk",
  "currency": "IDR",
  "payment_method": "qris"
}
```

Response:

```json
{
  "id": "txn_xxxxx",
  "external_id": "ORDER-001",
  "amount": 50000,
  "qris_payload": "000201010211...",
  "status": "pending"
}
```

### Check Payment Status

`GET /pay/v1/{external_id}`

Response:

```json
{
  "id": "txn_xxxxx",
  "external_id": "ORDER-001",
  "amount": 50000,
  "status": "paid"
}
```

| Status | Arti |
|---|---|
| `pending` | Menunggu pembayaran |
| `paid` | Pembayaran berhasil |
| `expired` | QRIS sudah kedaluwarsa |
| `cancelled` | Transaksi dibatalkan |

## Troubleshooting

<details>
<summary><b>Error "Failed to fetch"</b></summary>

**Penyebab:** CORS policy atau API key tidak valid.

**Solusi:**
- Pastikan API key benar
- Pastikan domain kamu diizinkan di dashboard CortexPay
- Pakai backend proxy untuk production

</details>

<details>
<summary><b>Error "Quota exceeded"</b></summary>

**Penyebab:** Limit API terlampaui.

**Solusi:**
- Tunggu beberapa menit sampai limit reset
- Upgrade plan CortexPay kalau perlu

</details>

<details>
<summary><b>QR code tidak muncul</b></summary>

**Penyebab:** Library QRCode.js gagal dimuat.

**Solusi:**
- Cek koneksi internet (library dimuat dari CDN)
- Atau download library-nya dan simpan lokal

</details>

## Link

- Dashboard: https://cortexpay.my.id
- Dokumentasi: https://cortexpay.my.id/docs

## License

[MIT](LICENSE). Bebas dipakai untuk project personal maupun komersial.

---

<div align="center">
<sub>Dibuat untuk developer Indonesia</sub>
</div>
