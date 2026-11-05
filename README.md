# CortexPay QRIS Integration Template

Template sederhana untuk integrasi pembayaran QRIS menggunakan CortexPay API.

## 📋 Fitur

- ✅ Generate QRIS dinamis
- ✅ Display QR code otomatis
- ✅ Format nominal Rupiah
- ✅ Auto-generate Order ID
- ✅ Payment status polling
- ✅ Responsive design (mobile-friendly)
- ✅ Error handling

## 🚀 Quick Start

### 1. Clone / Download

```bash
# Clone repository ini atau download folder
git clone <repository-url>
cd cortexpay-integration-template
```

### 2. Konfigurasi API

Edit file `app.js` dan ganti konfigurasi berikut:

```javascript
const CONFIG = {
  API_BASE_URL: 'https://cortexpay.my.id',
  API_KEY: 'YOUR_API_KEY_HERE',  // ← Ganti dengan API Key Anda
  MERCHANT_ID: null
};
```

**Cara mendapatkan API Key:**
1. Login ke dashboard CortexPay
2. Buka menu **API Keys**
3. Klik **Buat API Key Baru**
4. Salin API Key dan tempel ke `app.js`

### 3. Jalankan

Buka file `index.html` di browser, atau gunakan local server:

```bash
# Menggunakan Python
python -m http.server 3000

# Menggunakan Node.js (jika ada package.json)
npx serve

# Menggunakan PHP
php -S localhost:3000
```

Kemudian buka `http://localhost:3000` di browser.

## 📖 Cara Menggunakan

### Form Pembayaran

1. **Nominal**: Masukkan jumlah pembayaran (minimal Rp 1,000)
2. **Order ID**: Otomatis terisi, bisa diubah manual
3. **Deskripsi**: Keterangan pembayaran (opsional)
4. Klik **Buat QRIS**

### Setelah QRIS Dibuat

- QR code akan ditampilkan
- Nominal pembayaran dalam format Rupiah
- Tombol **Salin QRIS String** untuk copy manual
- Tombol **Transaksi Baru** untuk reset form

### Payment Polling (Otomatis)

Template ini sudah include auto-polling untuk cek status pembayaran setiap 3 detik. Ketika pembayaran terdeteksi, akan muncul notifikasi "Pembayaran Berhasil".

## 🔧 Kustomisasi

### Ganti Warna Tema

Edit file `styles.css`:

```css
/* Warna hijau CortexPay */
.btn-primary {
  background: #168653;  /* ← Ganti warna ini */
}

header h1 {
  color: #168653;  /* ← Ganti warna ini */
}
```

### Ubah API Endpoint

Jika menggunakan self-hosted CortexPay:

```javascript
const CONFIG = {
  API_BASE_URL: 'https://your-domain.com',  // ← Ganti URL
  API_KEY: 'YOUR_API_KEY',
  MERCHANT_ID: null
};
```

### Disable Payment Polling

Hapus atau comment bagian polling di `app.js`:

```javascript
// startPaymentPolling(externalId);  // ← Comment baris ini
```

## 📁 Struktur File

```
cortexpay-integration-template/
├── index.html          # Halaman utama
├── styles.css          # Styling
├── app.js              # Logic & API integration
└── README.md           # Dokumentasi
```

## 🔐 Keamanan

### ⚠️ PENTING: Jangan Hardcode API Key di Production!

Untuk production, gunakan environment variables atau backend proxy:

**Contoh dengan Backend Proxy:**

```javascript
// app.js
async function createPayment(amount, externalId, description) {
  const response = await fetch('/api/create-payment', {  // ← Proxy ke backend Anda
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ amount, externalId, description })
  });
  return await response.json();
}
```

```javascript
// backend.js (Node.js Express)
app.post('/api/create-payment', async (req, res) => {
  const { amount, externalId, description } = req.body;
  
  const response = await fetch('https://cortexpay.my.id/pay/v1/create', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-API-Key': process.env.CORTEXPAY_API_KEY  // ← Dari .env
    },
    body: JSON.stringify({ amount, externalId, description })
  });
  
  res.json(await response.json());
});
```

## 🛠️ Troubleshooting

### Error: "Failed to fetch"

**Penyebab:** CORS policy atau API Key tidak valid.

**Solusi:**
1. Pastikan API Key benar
2. Pastikan domain Anda diizinkan di dashboard CortexPay
3. Gunakan backend proxy untuk production

### Error: "Quota exceeded"

**Penyebab:** Limit API terlampaui.

**Solusi:**
- Tunggu beberapa menit untuk reset limit
- Upgrade plan CortexPay jika perlu

### QR Code Tidak Muncul

**Penyebab:** Library QRCode.js tidak dimuat.

**Solusi:**
- Pastikan koneksi internet aktif (CDN)
- Atau download library dan simpan lokal

## 📚 API Reference

### Create Payment

```
POST /pay/v1/create
Headers:
  X-API-Key: YOUR_API_KEY
  Content-Type: application/json

Body:
{
  "external_id": "ORDER-001",
  "amount": 50000,
  "description": "Pembayaran produk",
  "currency": "IDR",
  "payment_method": "qris"
}

Response:
{
  "id": "txn_xxxxx",
  "external_id": "ORDER-001",
  "amount": 50000,
  "qris_payload": "000201010211...",
  "status": "pending"
}
```

### Check Payment Status

```
GET /pay/v1/{external_id}
Headers:
  X-API-Key: YOUR_API_KEY

Response:
{
  "id": "txn_xxxxx",
  "external_id": "ORDER-001",
  "amount": 50000,
  "status": "paid"  // paid, pending, expired, cancelled
}
```

## 🎨 Customization Examples

### Tambah Logo

Edit `index.html`:

```html
<header>
  <img src="logo.png" alt="Logo" class="logo">
  <h1>QRIS Payment</h1>
  <p>Scan QRIS untuk pembayaran</p>
</header>
```

```css
/* styles.css */
.logo {
  width: 80px;
  height: auto;
  margin-bottom: 16px;
}
```

### Tambah Sound Notification

Edit `app.js`:

```javascript
function paymentSuccess() {
  // Play sound
  const audio = new Audio('success.mp3');
  audio.play();
  
  // ... existing code
}
```

### Tambah WhatsApp Notification

```javascript
function paymentSuccess() {
  // Send WhatsApp notification
  const phone = '6281234567890';
  const message = 'Pembayaran berhasil!';
  window.open(`https://wa.me/${phone}?text=${encodeURIComponent(message)}`);
  
  // ... existing code
}
```

## 📞 Support

Untuk dokumentasi lengkap CortexPay API, kunjungi:
- **Dashboard:** https://cortexpay.my.id
- **Dokumentasi:** https://cortexpay.my.id/docs

## 📄 License

MIT License - Bebas digunakan untuk project personal maupun komersial.

---

**Dibuat dengan ❤️ untuk developer Indonesia**
