<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistem Inventory Booth</title>

<!-- Library -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.31/jspdf.plugin.autotable.min.js"></script>

<style>
  body {
    font-family: 'Segoe UI', Arial, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    margin: 0; padding: 20px;
  }
  .container {
    max-width: 1400px; margin: auto; background: white;
    border-radius: 15px; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.2);
  }
  .header { background: #667eea; color: white; padding: 25px; text-align: center; }
  .header h1 { margin: 0; font-size: 28px; }
  .config-section {
    background: #fff8dc; border: 2px solid #ffc107;
    padding: 15px; margin: 20px; border-radius: 10px;
  }
  label { font-weight: bold; }
  input[type=text] { width: 100%; padding: 8px; margin: 8px 0; border-radius: 5px; border: 1px solid #ccc; }
  button { padding: 10px 18px; border: none; border-radius: 6px; cursor: pointer; margin: 4px; }
  .btn-primary { background: #667eea; color: white; }
  .btn-success { background: #10b981; color: white; }
  .btn-warning { background: #f59e0b; color: white; }
  .btn-danger { background: #ef4444; color: white; }
  .btn-info { background: #06b6d4; color: white; }
  table { width: 100%; border-collapse: collapse; margin-top: 10px; }
  th, td { padding: 10px; border-bottom: 1px solid #ddd; text-align: left; }
  th { background: #667eea; color: white; position: sticky; top: 0; }
  .summary-cards { display: flex; gap: 15px; flex-wrap: wrap; margin: 20px; }
  .card { flex: 1; background: #667eea; color: white; padding: 15px; border-radius: 8px; text-align: center; }
  .badge { padding: 4px 10px; border-radius: 10px; font-size: 12px; font-weight: bold; }
  .badge-success { background: #d1fae5; color: #065f46; }
  .badge-danger { background: #fee2e2; color: #991b1b; }
  .badge-warning { background: #fef3c7; color: #92400e; }
</style>
</head>
<body>

<div class="container">
  <div class="header">
    <h1>📦 SISTEM INVENTORY BOOTH</h1>
    <p>Terhubung langsung ke Google Spreadsheet</p>
  </div>

  <div class="config-section">
    <h3>⚙️ Konfigurasi Google Sheets</h3>
    <label>URL Google Apps Script:</label>
    <input type="text" id="apiUrl"
      value="https://script.google.com/macros/s/AKfycbzkCElaoI9fxJalptSjgddlbs73dPtq6Mc8TMta86dts0rpf8noAYW1Cy82fn3nklcxag/exec"
      placeholder="Masukkan URL Google Apps Script di sini">
    <button class="btn-primary" onclick="saveConfig()">💾 Simpan Konfigurasi</button>
    <button class="btn-success" onclick="loadData()">🔄 Muat Data dari Sheets</button>
  </div>

  <div class="summary-cards">
    <div class="card"><div>Total Item</div><div id="totalItem">0</div></div>
    <div class="card"><div>Stok Awal</div><div id="stokAwal">0</div></div>
    <div class="card"><div>Terjual</div><div id="totalTerjual">0</div></div>
    <div class="card"><div>Kembali</div><div id="totalKembali">0</div></div>
    <div class="card"><div>Tersedia</div><div id="stokTersedia">0</div></div>
  </div>

  <div style="padding: 0 20px 20px 20px;">
    <button class="btn-success" onclick="openModalTambah()">➕ Tambah Produk</button>
    <button class="btn-info" onclick="hitungOtomatis()">🔄 Hitung Otomatis</button>
    <button class="btn-warning" onclick="simpanKeSheets()">💾 Simpan ke Sheets</button>
    <button class="btn-primary" onclick="exportExcel()">📊 Export Excel</button>
    <button class="btn-danger" onclick="generatePDF()">📄 Download PDF</button>
  </div>

  <div class="table-container" style="padding:20px;">
    <table id="inventoryTable">
      <thead>
        <tr>
          <th>No</th>
          <th>Kode Barang</th>
          <th>Nama Barang</th>
          <th>Stok Awal</th>
          <th>Terjual</th>
          <th>Kembali</th>
          <th>Tersedia</th>
          <th>Selisih</th>
          <th>Status</th>
          <th>Aksi</th>
        </tr>
      </thead>
      <tbody id="tableBody">
        <tr><td colspan="10" style="text-align:center; padding:40px;">📭 Belum ada data. Klik "Muat Data dari Sheets".</td></tr>
      </tbody>
    </table>
  </div>
</div>

<!-- Modal Tambah -->
<div id="modalTambah" style="display:none; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.5); justify-content:center; align-items:center;">
  <div style="background:white; padding:20px; border-radius:10px; width:300px;">
    <h3>Tambah Produk</h3>
    <label>Kode Barang:</label><input type="text" id="inputKode"><br>
    <label>Nama Barang:</label><input type="text" id="inputNama"><br>
    <label>Stok Awal:</label><input type="number" id="inputStok"><br>
    <button class="btn-success" onclick="tambahProduk()">✅ Simpan</button>
    <button class="btn-danger" onclick="closeModal()">❌ Batal</button>
  </div>
</div>

<script>
let produkData = [];
let apiUrlConfig = "";

// Load otomatis saat dibuka
window.onload = () => {
  const savedUrl = localStorage.getItem("apiUrl");
  if (savedUrl) {
    document.getElementById("apiUrl").value = savedUrl;
    apiUrlConfig = savedUrl;
    loadData();
  }
};

// Simpan URL
function saveConfig() {
  const url = document.getElementById("apiUrl").value.trim();
  if (!url) return alert("❌ URL tidak boleh kosong!");
  localStorage.setItem("apiUrl", url);
  apiUrlConfig = url;
  alert("✅ URL disimpan!");
}

// Load data
async function loadData() {
  if (!apiUrlConfig) return alert("❌ Isi URL dulu!");
  try {
    const res = await fetch(apiUrlConfig + "?action=getProduk");
    const result = await res.json();
    if (result.status === "success") {
      produkData = result.data;
      renderTable();
      hitungOtomatis();
    } else throw new Error(result.message);
  } catch (e) {
    alert("❌ Gagal muat data: " + e.message);
  }
}

// Simpan ke Sheets
async function simpanKeSheets() {
  if (!apiUrlConfig) return alert("❌ URL belum diisi!");
  try {
    const res = await fetch(apiUrlConfig + "?action=saveProduk", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(produkData),
    });
    const result = await res.json();
    if (result.status === "success") alert("✅ Tersimpan ke Google Sheets!");
    else throw new Error(result.message);
  } catch (e) {
    alert("❌ Gagal simpan: " + e.message);
  }
}

// Render tabel
function renderTable() {
  const tbody = document.getElementById("tableBody");
  tbody.innerHTML = "";
  if (produkData.length === 0) {
    tbody.innerHTML = '<tr><td colspan="10" style="text-align:center;">📭 Belum ada data</td></tr>';
    return;
  }
  produkData.forEach((item, i) => {
    const row = tbody.insertRow();
    row.innerHTML = `
      <td>${i+1}</td><td>${item.kode}</td><td>${item.nama}</td>
      <td>${item.stok}</td>
      <td><input type="number" min="0" value="0" id="terjual_${i}" onchange="hitung(${i})"></td>
      <td><input type="number" min="0" value="0" id="kembali_${i}" onchange="hitung(${i})"></td>
      <td id="tersedia_${i}">-</td><td id="selisih_${i}">-</td><td id="status_${i}">-</td>
      <td><button class="btn-danger" onclick="hapusProduk(${i})">🗑️</button></td>`;
  });
}

// Hitung otomatis
function hitungOtomatis(){ produkData.forEach((_,i)=>hitung(i)); updateSummary(); }
function hitung(i){
  const item = produkData[i];
  const terjual = +document.getElementById(`terjual_${i}`).value || 0;
  const kembali = +document.getElementById(`kembali_${i}`).value || 0;
  const tersedia = item.stok - terjual;
  const selisih = tersedia - kembali;
  document.getElementById(`tersedia_${i}`).textContent = tersedia;
  document.getElementById(`selisih_${i}`).textContent = selisih;
  let status = "✅ Sesuai";
  if (selisih > 0) status = `<span class='badge badge-danger'>⚠️ Hilang ${selisih}</span>`;
  else if (selisih < 0) status = `<span class='badge badge-warning'>⚠️ Lebih ${Math.abs(selisih)}</span>`;
  document.getElementById(`status_${i}`).innerHTML = status;
  updateSummary();
}
function updateSummary(){
  let totalStok=0, totalTerjual=0, totalKembali=0, totalTersedia=0;
  produkData.forEach((item,i)=>{
    totalStok += item.stok;
    totalTerjual += +document.getElementById(`terjual_${i}`)?.value || 0;
    totalKembali += +document.getElementById(`kembali_${i}`)?.value || 0;
    totalTersedia += item.stok - (+document.getElementById(`terjual_${i}`)?.value||0);
  });
  document.getElementById("totalItem").textContent=produkData.length;
  document.getElementById("stokAwal").textContent=totalStok;
  document.getElementById("totalTerjual").textContent=totalTerjual;
  document.getElementById("totalKembali").textContent=totalKembali;
  document.getElementById("stokTersedia").textContent=totalTersedia;
}

// Modal tambah
function openModalTambah(){ document.getElementById("modalTambah").style.display="flex"; }
function closeModal(){ document.getElementById("modalTambah").style.display="none"; }
function tambahProduk(){
  const kode=document.getElementById("inputKode").value.trim();
  const nama=document.getElementById("inputNama").value.trim();
  const stok=+document.getElementById("inputStok").value||0;
  if(!kode||!nama)return alert("❌ Lengkapi data!");
  produkData.push({kode,nama,stok});
  closeModal(); renderTable(); updateSummary();
}

// Hapus produk
function hapusProduk(i){ if(confirm("Hapus produk ini?")){ produkData.splice(i,1); renderTable(); updateSummary(); }}

// Export Excel
function exportExcel(){ const ws=XLSX.utils.json_to_sheet(produkData); const wb=XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb,ws,"Inventory"); XLSX.writeFile(wb,"Laporan_Inventory.xlsx"); }

// PDF
function generatePDF(){ const{jsPDF}=window.jspdf; const doc=new jsPDF(); doc.text("Laporan Inventory Booth",14,20);
doc.autoTable({head:[["No","Kode","Nama","Stok"]],body:produkData.map((p,i)=>[i+1,p.kode,p.nama,p.stok]),startY:30}); doc.save("Laporan_Inventory.pdf"); }
</script>
</body>
</html>
