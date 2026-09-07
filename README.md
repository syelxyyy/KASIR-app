<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aplikasi Kasir Sederhana</title>
    <style>
        * { box-sizing: border-box; font-family: Arial, sans-serif; }
        body { margin: 20px; background-color: #f4f6f9; }
        .container { display: flex; gap: 20px; flex-wrap: wrap; }
        .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 6px rgba(0,0,0,0.1); flex: 1; min-width: 300px; }
        h2 { margin-top: 0; color: #333; border-bottom: 2px solid #eee; padding-bottom: 10px; }
        input, select, button { width: 100%; padding: 10px; margin: 6px 0 12px; border: 1px solid #ccc; border-radius: 4px; }
        button { background-color: #007bff; color: white; border: none; cursor: pointer; font-weight: bold; }
        button:hover { background-color: #0056b3; }
        .btn-danger { background-color: #dc3545; }
        .btn-danger:hover { background-color: #bd2130; }
        .btn-success { background-color: #28a745; }
        .btn-success:hover { background-color: #218838; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f8f9fa; }
        .total-box { font-size: 1.2em; font-weight: bold; margin-top: 10px; text-align: right; }
        
        @media print {
            body * { visibility: hidden; }
            #areaStruk, #areaStruk * { visibility: visible; }
            #areaStruk { position: absolute; left: 0; top: 0; width: 100%; }
            .no-print { display: none; }
        }
    </style>
</head>
<body>

<div class="container no-print">
    <!-- Tambah Produk -->
    <div class="card">
        <h2>Kelola Produk</h2>
        <form id="formProduk">
            <label>Nama Produk</label>
            <input type="text" id="namaProduk" required>
            <label>Harga (Rp)</label>
            <input type="number" id="hargaProduk" min="1" required>
            <button type="submit">Tambah Produk</button>
        </form>
        <h3>Daftar Produk</h3>
        <table>
            <thead>
                <tr><th>Produk</th><th>Harga</th><th>Aksi</th></tr>
            </thead>
            <tbody id="tabelProduk"></tbody>
        </table>
    </div>

    <!-- Transaksi Kasir -->
    <div class="card">
        <h2>Kasir</h2>
        <label>Pilih Produk</label>
        <select id="pilihProduk"></select>
        <label>Jumlah</label>
        <input type="number" id="jumlahProduk" value="1" min="1">
        <button type="button" onclick="tambahKeKeranjang()">+ Ke Keranjang</button>

        <h3>Keranjang Belanja</h3>
        <table>
            <thead>
                <tr><th>Item</th><th>Harga</th><th>Qty</th><th>Subtotal</th></tr>
            </thead>
            <tbody id="tabelKeranjang"></tbody>
        </table>
        <div class="total-box">Total: Rp <span id="totalBelanja">0</span></div>
        <button type="button" class="btn-success" onclick="prosesBayar()">Bayar & Cetak Struk</button>
    </div>
</div>

<!-- Area Cetak Struk -->
<div id="areaStruk" style="display:none; padding: 20px; background: white;">
    <h3 style="text-align:center; margin:0;">STRUK PEMBAYARAN</h3>
    <p id="strukWaktu" style="text-align:center; font-size: 0.8em; margin-bottom: 15px;"></p>
    <hr>
    <div id="strukItem"></div>
    <hr>
    <p style="text-align:right; font-weight:bold;">Total: Rp <span id="strukTotal">0</span></p>
    <p style="text-align:center; margin-top:20px;">Terima Kasih Atas Kunjungan Anda!</p>
</div>

<script>
    let produkList = JSON.parse(localStorage.getItem('kasir_produk')) || [];
    let keranjang = [];

    const formProduk = document.getElementById('formProduk');
    const tabelProduk = document.getElementById('tabelProduk');
    const pilihProduk = document.getElementById('pilihProduk');
    const tabelKeranjang = document.getElementById('tabelKeranjang');

    function simpanData() {
        localStorage.setItem('kasir_produk', JSON.stringify(produkList));
        renderProduk();
    }

    function renderProduk() {
        tabelProduk.innerHTML = '';
        pilihProduk.innerHTML = '';
        produkList.forEach((p, index) => {
            tabelProduk.innerHTML += `<tr>
                <td>${p.nama}</td>
                <td>Rp ${p.harga.toLocaleString()}</td>
                <td><button class="btn-danger" style="padding: 4px 8px; margin:0;" onclick="hapusProduk(${index})">Hapus</button></td>
            </tr>`;
            pilihProduk.innerHTML += `<option value="${index}">${p.nama} - Rp ${p.harga.toLocaleString()}</option>`;
        });
    }

    formProduk.addEventListener('submit', (e) => {
        e.preventDefault();
        const nama = document.getElementById('namaProduk').value;
        const harga = parseFloat(document.getElementById('hargaProduk').value);
        produkList.push({ nama, harga });
        simpanData();
        formProduk.reset();
    });

    function hapusProduk(index) {
        produkList.splice(index, 1);
        simpanData();
    }

    function tambahKeKeranjang() {
        if (produkList.length === 0) return alert('Tambahkan produk terlebih dahulu!');
        const index = pilihProduk.value;
        const qty = parseInt(document.getElementById('jumlahProduk').value);
        const item = produkList[index];

        const ada = keranjang.find(k => k.nama === item.nama);
        if (ada) {
            ada.qty += qty;
            ada.subtotal = ada.qty * ada.harga;
        } else {
            keranjang.push({ nama: item.nama, harga: item.harga, qty: qty, subtotal: item.harga * qty });
        }
        renderKeranjang();
    }

    function renderKeranjang() {
        tabelKeranjang.innerHTML = '';
        let total = 0;
        keranjang.forEach(k => {
            total += k.subtotal;
            tabelKeranjang.innerHTML += `<tr>
                <td>${k.nama}</td>
                <td>Rp ${k.harga.toLocaleString()}</td>
                <td>${k.qty}</td>
                <td>Rp ${k.subtotal.toLocaleString()}</td>
            </tr>`;
        });
        document.getElementById('totalBelanja').innerText = total.toLocaleString();
    }

    function prosesBayar() {
        if (keranjang.length === 0) return alert('Keranjang masih kosong!');
        
        let total = keranjang.reduce((sum, item) => sum + item.subtotal, 0);
        let strukHTML = '';
        
        keranjang.forEach(item => {
            strukHTML += `<div style="display:flex; justify-content:space-between;">
                <span>${item.nama} x${item.qty}</span>
                <span>Rp ${item.subtotal.toLocaleString()}</span>
            </div>`;
        });

        document.getElementById('strukWaktu').innerText = new Date().toLocaleString('id-ID');
        document.getElementById('strukItem').innerHTML = strukHTML;
        document.getElementById('strukTotal').innerText = total.toLocaleString();

        const areaStruk = document.getElementById('areaStruk');
        areaStruk.style.display = 'block';
        window.print();
        areaStruk.style.display = 'none';

        keranjang = [];
        renderKeranjang();
    }

    renderProduk();
</script>
</body>
</html>
