# InventoryBoothScript
Welcome Inventory Booth Event 
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistem Inventory Booth</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.31/jspdf.plugin.autotable.min.js"></script>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  
  body {
    font-family: 'Segoe UI', Arial, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    padding: 20px;
  }
  
  .container {
    max-width: 1400px;
    margin: 0 auto;
    background: white;
    border-radius: 20px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
    overflow: hidden;
  }
  
  .header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 30px;
    text-align: center;
  }
  
  .header h1 {
    font-size: 32px;
    margin-bottom: 10px;
    text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
  }
  
  .header p {
    opacity: 0.9;
    font-size: 16px;
  }
  
  .config-section {
    background: #fff3cd;
    border: 2px solid #ffc107;
    padding: 20px;
    margin: 20px;
    border-radius: 10px;
  }
  
  .config-section h3 {
    color: #856404;
    margin-bottom: 15px;
  }
  
  .config-input {
    display: flex;
    gap: 10px;
    align-items: center;
    margin-bottom: 10px;
  }
  
  .config-input label {
    font-weight: 600;
    min-width: 150px;
  }
  
  .config-input input {
    flex: 1;
    padding: 10px;
    border: 2px solid #ddd;
    border-radius: 5px;
    font-size: 14px;
  }
  
  .toolbar {
    padding: 20px;
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
    border-bottom: 2px solid #eee;
  }
  
  .btn {
    padding: 12px 24px;
    border: none;
    border-radius: 8px;
    font-size: 14px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
    display: inline-flex;
    align-items: center;
    gap: 8px;
  }
  
  .btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(0,0,0,0.2);
  }
  
  .btn-primary { background: #667eea; color: white; }
  .btn-success { background: #10b981; color: white; }
  .btn-danger { background: #ef4444; color: white; }
  .btn-info { background: #06b6d4; color: white; }
  .btn-warning { background: #f59e0b; color: white; }
  
  .date-selector {
    padding: 20px;
    background: #f8f9fa;
    border-bottom: 2px solid #eee;
  }
  
  .date-selector label {
    font-weight: 600;
    margin-right: 10px;
  }
  
  .date-selector input[type="date"] {
    padding: 10px;
    border: 2px solid #ddd;
    border-radius: 5px;
    font-size: 14px;
  }
  
  .summary-cards {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 20px;
    padding: 20px;
  }
  
  .card {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 20px;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  }
  
  .card-label {
    font-size: 12px;
    opacity: 0.9;
    margin-bottom: 5px;
  }
  
  .card-value {
    font-size: 32px;
    font-weight: 700;
  }
  
  .table-container {
    padding: 20px;
    overflow-x: auto;
  }
  
  table {
    width: 100%;
    border-collapse: collapse;
    background: white;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  }
  
  th {
    background: #667eea;
    color: white;
    padding: 15px 10px;
    text-align: left;
    font-weight: 600;
    position: sticky;
    top: 0;
    z-index: 10;
  }
  
  td {
    padding: 12px 10px;
    border-bottom: 1px solid #eee;
  }
  
  tr:hover {
    background: #f8f9fa;
  }
  
  input[type="number"] {
    width: 80px;
    padding: 8px;
    border: 2px solid #ddd;
    border-radius: 5px;
    text-align: center;
  }
  
  .badge {
    display: inline-block;
    padding: 4px 12px;
    border-radius: 12px;
    font-size: 12px;
    font-weight: 600;
  }
  
  .badge-success { background: #d1fae5; color: #065f46; }
  .badge-danger { background: #fee2e2; color: #991b1b; }
  .badge-warning { background: #fef3c7; color: #92400e; }
  
  .modal {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0,0,0,0.5);
    z-index: 1000;
    justify-content: center;
    align-items: center;
  }
  
  .modal.active {
    display: flex;
  }
  
  .modal-content {
    background: white;
    padding: 30px;
    border-radius: 15px;
    max-width: 500px;
    width: 90%;
    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
  }
  
  .modal-header {
    font-size: 24px;
    font-weight: 700;
    margin-bottom: 20px;
    color: #667eea;
  }
  
  .form-group {
    margin-bottom: 20px;
  }
  
  .form-group label {
    display: block;
    font-weight: 600;
    margin-bottom: 8px;
    color: #333;
  }
  
  .form-group input {
    width: 100%;
    padding: 12px;
    border: 2px solid #ddd;
    border-radius: 8px;
    font-size: 14px;
  }
  
  .loading {
    text-align: center;
    padding: 40px;
    color: #667eea;
  }
  
  .loading::after {
    content: '⏳';
    font-size: 48px;
    animation: spin 2s linear infinite;
  }
  
  @keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
  }
  
  @media print {
    .toolbar, .config-section, .btn { display: none; }
  }
</style>
</head>
<body>

<div class="container">
  <div class="header">
    <h1>📦 SISTEM INVENTORY BOOTH</h1>
    <p>Tim Surprice - Manajemen Stok & Transaksi</p>
  </div>
  
  <!-- CONFIG SECTION -->
  <div class="config-section">
    <h3>⚙️ Konfigurasi Google Sheets</h3>
    <div class="config-input">
      <label>URL Google Apps Script:</label>
      <input type="text" id="apiUrl" https://script.google.com/macros/s/AKfycbzsI_dxXZ-mgM-5_Vihrv-zyHdeWVKuRKrXhnimdatn4o88pbde0nPEftKXyOlhHJPt-g/exec">

    </div>
    <button class="btn btn-primary" onclick="saveConfig()">💾 Simpan Konfigurasi</button>
    <button class="btn btn-success" onclick="loadData()">🔄 Muat Data dari Sheets</button>
  </div>
  
  <!-- TOOLBAR -->
  <div class="toolbar">
    <button class="btn btn-success" onclick="openModalTambah()">➕ Tambah Produk</button>
    <button class="btn btn-info" onclick="hitungOtomatis()">🔄 Hitung Otomatis</button>
    <button class="btn btn-warning" onclick="simpanKeSheets()">💾 Simpan ke Sheets</button>
    <button class="btn btn-primary" onclick="exportExcel()">📊 Export Excel</button>
    <button class="btn btn-danger" onclick="generatePDF()">📄 Download PDF Laporan</button>
  </div>
  
  <!-- DATE SELECTOR -->
  <div class="date-selector">
    <label>📅 Tanggal Transaksi:</label>
    <input type="date" id="tanggalTransaksi" onchange="updateTanggal()">
  </div>
  
  <!-- SUMMARY CARDS -->
  <div class="summary-cards">
    <div class="card">
      <div class="card-label">Total Item</div>
      <div class="card-value" id="totalItem">0</div>
    </div>
    <div class="card">
      <div class="card-label">Stok Awal</div>
      <div class="card-value" id="stokAwal">0</div>
    </div>
    <div class="card">
      <div class="card-label">Total Terjual</div>
      <div class="card-value" id="totalTerjual">0</div>
    </div>
    <div class="card">
      <div class="card-label">Stok Kembali</div>
      <div class="card-value" id="totalKembali">0</div>
    </div>
    <div class="card">
      <div class="card-label">Stok Tersedia</div>
      <div class="card-value" id="stokTersedia">0</div>
    </div>
  </div>
  
  <!-- TABLE -->
  <div class="table-container">
    <table id="inventoryTable">
      <thead>
        <tr>
          <th style="width: 50px;">No</th>
          <th>Kode Barang</th>
          <th>Nama Barang</th>
          <th style="width: 100px;">Stok Awal</th>
          <th style="width: 100px;">Terjual</th>
          <th style="width: 100px;">Kembali</th>
          <th style="width: 100px;">Tersedia</th>
          <th style="width: 100px;">Selisih</th>
          <th>Status</th>
          <th style="width: 100px;">Aksi</th>
        </tr>
      </thead>
      <tbody id="tableBody">
        <tr>
          <td colspan="10" class="loading">Memuat data...</td>
        </tr>
      </tbody>
    </table>
  </div>
</div>

<!-- MODAL TAMBAH PRODUK -->
<div class="modal" id="modalTambah">
  <div class="modal-content">
    <div class="modal-header">➕ Tambah Produk Baru</div>
    <div class="form-group">
      <label>Kode Barang</label>
      <input type="text" id="inputKode" placeholder="Contoh: CUL-001">
    </div>
    <div class="form-group">
      <label>Nama Barang</label>
      <input type="text" id="inputNama" placeholder="Nama produk">
    </div>
    <div class="form-group">
      <label>Stok Awal</label>
      <input type="number" id="inputStok" placeholder="0" min="0">
    </div>
    <div class="form-group">
      <label>Jumlah Sampel</label>
      <input type="number" id="inputSampel" placeholder="0" min="0">
    </div>
    <div style="display: flex; gap: 10px; margin-top: 20px;">
      <button class="btn btn-success" onclick="tambahProduk()" style="flex: 1;">✅ Simpan</button>
      <button class="btn btn-danger" onclick="closeModal()" style="flex: 1;">❌ Batal</button>
    </div>
  </div>
</div>

<script>
let produkData = [];
let apiUrlConfig = '';

// Set tanggal hari ini sebagai default
document.getElementById('tanggalTransaksi').valueAsDate = new Date();

// Load config dari memory
function loadConfig() {
  const saved = window.apiUrlSaved || '';
  document.getElementById('apiUrl').value = saved;
  apiUrlConfig = saved;
}

// Save config ke memory
function saveConfig() {
  const url = document.getElementById('apiUrl').value.trim();
  if (!url) {
    alert('❌ URL tidak boleh kosong!');
    return;
  }
  apiUrlConfig = url;
  window.apiUrlSaved = url;
  alert('✅ Konfigurasi berhasil disimpan!\n\nSekarang klik "Muat Data dari Sheets"');
}

// Load data dari Google Sheets
async function loadData() {
  if (!apiUrlConfig) {
    alert('❌ Harap isi URL Google Apps Script terlebih dahulu!');
    return;
  }
  
  try {
    document.getElementById('tableBody').innerHTML = '<tr><td colspan="10" class="loading">Memuat data...</td></tr>';
    
    const response = await fetch(apiUrlConfig + '?action=getProduk');
    const result = await response.json();
    
    if (result.status === 'success') {
      produkData = result.data;
      renderTable();
      hitungOtomatis();
      alert('✅ Data berhasil dimuat dari Google Sheets!');
    } else {
      throw new Error(result.message);
    }
  } catch (error) {
    alert('❌ Gagal memuat data: ' + error.message + '\n\nPastikan:\n1. URL sudah benar\n2. Web App sudah di-deploy\n3. Permission diatur ke "Anyone"');
    renderTableLocal();
  }
}

// Render table dari data lokal (backup)
function renderTableLocal() {
  produkData = [
    {no: 1, kode: "Cul 3805", nama: "Cul 3805", stok: 10, sampel: 1},
    {no: 2, kode: "TLG TESSA-1", nama: "TLG TESSA-1", stok: 10, sampel: 1},
    {no: 3, kode: "GILI 9604-GGI", nama: "GILI 9604 - GGI (GBT)", stok: 1, sampel: 1}
  ];
  renderTable();
}

// Render table
function renderTable() {
  const tbody = document.getElementById('tableBody');
  tbody.innerHTML = '';
  
  if (produkData.length === 0) {
    tbody.innerHTML = '<tr><td colspan="10" style="text-align: center; padding: 40px;">Tidak ada data. Klik "Tambah Produk" untuk memulai.</td></tr>';
    return;
  }
  
  produkData.forEach((item, index) => {
    const row = tbody.insertRow();
    row.innerHTML = `
      <td>${item.no}</td>
      <td><strong>${item.kode}</strong></td>
      <td>${item.nama}</td>
      <td style="text-align: center;">${item.stok}</td>
      <td style="text-align: center;">
        <input type="number" min="0" value="0" id="terjual_${index}" onchange="hitung(${index})">
      </td>
      <td style="text-align: center;">
        <input type="number" min="0" value="0" id="kembali_${index}" onchange="hitung(${index})">
      </td>
      <td style="text-align: center;" id="tersedia_${index}">-</td>
      <td style="text-align: center;" id="selisih_${index}">-</td>
      <td id="status_${index}">-</td>
      <td style="text-align: center;">
        <button class="btn btn-danger" onclick="hapusProduk(${item.no})" style="padding: 6px 12px; font-size: 12px;">🗑️</button>
      </td>
    `;
  });
  
  updateSummary();
}

// Hitung per item
function hitung(index) {
  const item = produkData[index];
  const terjual = parseInt(document.getElementById(`terjual_${index}`).value) || 0;
  const kembali = parseInt(document.getElementById(`kembali_${index}`).value) || 0;
  
  const tersedia = item.stok - terjual;
  const selisih = tersedia - kembali;
  
  document.getElementById(`tersedia_${index}`).textContent = tersedia;
  document.getElementById(`selisih_${index}`).textContent = selisih;
  
  // Status badge
  let status = '';
  if (selisih > 0) {
    status = '<span class="badge badge-danger">⚠️ Hilang: ' + selisih + '</span>';
  } else if (selisih < 0) {
    status = '<span class="badge badge-warning">⚠️ Lebih: ' + Math.abs(selisih) + '</span>';
  } else {
    status = '<span class="badge badge-success">✅ Sesuai</span>';
  }
  
  document.getElementById(`status_${index}`).innerHTML = status;
}

// Hitung semua otomatis
function hitungOtomatis() {
  produkData.forEach((item, index) => {
    hitung(index);
  });
  updateSummary();
  alert('✅ Perhitungan selesai!');
}

// Update summary cards
function updateSummary() {
  let totalStok = 0;
  let totalTerjual = 0;
  let totalKembali = 0;
  let totalTersedia = 0;
  
  produkData.forEach((item, index) => {
    totalStok += item.stok;
    const terjual = parseInt(document.getElementById(`terjual_${index}`)?.value) || 0;
    const kembali = parseInt(document.getElementById(`kembali_${index}`)?.value) || 0;
    totalTerjual += terjual;
    totalKembali
    totalTersedia += item.stok - terjual;
  });

  document.getElementById('totalItem').textContent = produkData.length;
  document.getElementById('stokAwal').textContent = totalStok;
  document.getElementById('totalTerjual').textContent = totalTerjual;
  document.getElementById('totalKembali').textContent = totalKembali;
  document.getElementById('stokTersedia').textContent = totalTersedia;
}

// Tambah produk baru
function tambahProduk() {
  const kode = document.getElementById('inputKode').value.trim();
  const nama = document.getElementById('inputNama').value.trim();
  const stok = parseInt(document.getElementById('inputStok').value) || 0;
  const sampel = parseInt(document.getElementById('inputSampel').value) || 0;

  if (!kode || !nama) {
    alert('❌ Kode dan nama produk wajib diisi!');
    return;
  }

  const newItem = {
    no: produkData.length + 1,
    kode,
    nama,
    stok,
    sampel
  };

  produkData.push(newItem);
  renderTable();
  closeModal();
  updateSummary();
  alert('✅ Produk berhasil ditambahkan!');
}

// Hapus produk
function hapusProduk(no) {
  if (!confirm('Apakah yakin ingin menghapus produk ini?')) return;
  produkData = produkData.filter(p => p.no !== no);
  // Reset nomor urut
  produkData.forEach((p, i) => p.no = i + 1);
  renderTable();
  updateSummary();
}

// Simpan ke Google Sheets
async function simpanKeSheets() {
  if (!apiUrlConfig) {
    alert('❌ URL Google Apps Script belum diisi!');
    return;
  }

  try {
    const response = await fetch(apiUrlConfig + '?action=saveProduk', {
      method: 'POST',
      body: JSON.stringify(produkData),
      headers: { 'Content-Type': 'application/json' }
    });

    const result = await response.json();
    if (result.status === 'success') {
      alert('✅ Data berhasil disimpan ke Google Sheets!');
    } else {
      throw new Error(result.message);
    }
  } catch (error) {
    alert('❌ Gagal menyimpan ke Sheets: ' + error.message);
  }
}

// Export ke Excel
function exportExcel() {
  const ws = XLSX.utils.json_to_sheet(produkData);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "Inventory");
  XLSX.writeFile(wb, `Laporan_Inventory_${new Date().toISOString().slice(0,10)}.xlsx`);
}

// Generate PDF
function generatePDF() {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();

  doc.setFontSize(16);
  doc.text("Laporan Inventory Booth", 14, 20);
  doc.setFontSize(10);
  doc.text(`Tanggal: ${document.getElementById('tanggalTransaksi').value}`, 14, 28);

  const data = produkData.map((p, i) => [
    i + 1,
    p.kode,
    p.nama,
    p.stok,
    document.getElementById(`terjual_${i}`)?.value || 0,
    document.getElementById(`kembali_${i}`)?.value || 0,
    document.getElementById(`tersedia_${i}`)?.textContent || '-',
    document.getElementById(`selisih_${i}`)?.textContent || '-',
    document.getElementById(`status_${i}`)?.innerText || '-'
  ]);

  doc.autoTable({
    head: [['No', 'Kode', 'Nama Barang', 'Stok Awal', 'Terjual', 'Kembali', 'Tersedia', 'Selisih', 'Status']],
    body: data,
    startY: 35,
    theme: 'grid'
  });

  doc.save(`Laporan_Inventory_${new Date().toISOString().slice(0,10)}.pdf`);
}

// Modal handling
function openModalTambah() {
  document.getElementById('modalTambah').classList.add('active');
}
function closeModal() {
  document.getElementById('modalTambah').classList.remove('active');
}

// Update tanggal
function updateTanggal() {
  console.log("Tanggal transaksi:", document.getElementById('tanggalTransaksi').value);
}

// Jalankan awal
loadConfig();
renderTableLocal();
</script>
</body>
</html>
