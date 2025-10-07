<script>
let produkData = [];
let apiUrlConfig = "";

// Saat halaman pertama kali dibuka
window.onload = () => {
  const savedUrl = localStorage.getItem("apiUrl");
  if (savedUrl) {
    document.getElementById("apiUrl").value = savedUrl;
    apiUrlConfig = savedUrl;
    loadData(); // langsung coba ambil data dari Sheets
  } else {
    renderTableEmpty();
  }
  document.getElementById("tanggalTransaksi").valueAsDate = new Date();
};

// Simpan URL ke localStorage
function saveConfig() {
  const url = document.getElementById("apiUrl").value.trim();
  if (!url) return alert("❌ URL tidak boleh kosong!");
  apiUrlConfig = url;
  localStorage.setItem("apiUrl", url);
  alert("✅ URL disimpan! Sekarang bisa klik 'Muat Data dari Sheets'");
}

// Ambil data dari Google Sheets
async function loadData() {
  if (!apiUrlConfig) return alert("❌ Isi URL Google Apps Script dulu!");

  try {
    document.getElementById("tableBody").innerHTML =
      '<tr><td colspan="10" class="loading">Memuat data...</td></tr>';

    const res = await fetch(apiUrlConfig + "?action=getProduk");
    const text = await res.text();
    const result = JSON.parse(text);

    if (result.status === "success") {
      produkData = result.data;
      renderTable();
      hitungOtomatis();
      alert("✅ Data berhasil dimuat dari Google Sheets!");
    } else {
      throw new Error(result.message);
    }
  } catch (e) {
    console.error(e);
    alert(
      "❌ Gagal memuat data: " +
        e.message +
        "\n\nPastikan:\n1️⃣ URL benar\n2️⃣ Web App sudah di-deploy\n3️⃣ Akses diatur ke 'Siapa saja'"
    );
    renderTableEmpty();
  }
}

// Simpan data ke Google Sheets
async function simpanKeSheets() {
  if (!apiUrlConfig) return alert("❌ URL Google Apps Script belum diisi!");

  try {
    const res = await fetch(apiUrlConfig + "?action=saveProduk", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(produkData),
    });

    const text = await res.text();
    const result = JSON.parse(text);

    if (result.status === "success") {
      alert("✅ Data berhasil disimpan ke Google Sheets!");
    } else {
      throw new Error(result.message);
    }
  } catch (e) {
    alert("❌ Gagal menyimpan ke Sheets: " + e.message);
  }
}

// ===== RENDER TABEL =====
function renderTableEmpty() {
  const tbody = document.getElementById("tableBody");
  tbody.innerHTML = `
    <tr>
      <td colspan="10" style="text-align:center; padding:40px;">
        📭 Belum ada data. Klik "Muat Data dari Sheets" atau "Tambah Produk".
      </td>
    </tr>`;
}

function renderTable() {
  const tbody = document.getElementById("tableBody");
  tbody.innerHTML = "";

  if (produkData.length === 0) {
    renderTableEmpty();
    return;
  }

  produkData.forEach((item, i) => {
    const row = tbody.insertRow();
    row.innerHTML = `
      <td>${item.no}</td>
      <td><strong>${item.kode}</strong></td>
      <td>${item.nama}</td>
      <td style="text-align:center;">${item.stok}</td>
      <td style="text-align:center;"><input type="number" min="0" value="0" id="terjual_${i}" onchange="hitung(${i})"></td>
      <td style="text-align:center;"><input type="number" min="0" value="0" id="kembali_${i}" onchange="hitung(${i})"></td>
      <td style="text-align:center;" id="tersedia_${i}">-</td>
      <td style="text-align:center;" id="selisih_${i}">-</td>
      <td id="status_${i}">-</td>
      <td style="text-align:center;">
        <button class="btn btn-danger" onclick="hapusProduk(${item.no})" style="padding:6px 12px;font-size:12px;">🗑️</button>
      </td>`;
  });

  updateSummary();
}

// ===== PERHITUNGAN =====
function hitung(i) {
  const item = produkData[i];
  const terjual = parseInt(document.getElementById(`terjual_${i}`).value) || 0;
  const kembali = parseInt(document.getElementById(`kembali_${i}`).value) || 0;
  const tersedia = item.stok - terjual;
  const selisih = tersedia - kembali;

  document.getElementById(`tersedia_${i}`).textContent = tersedia;
  document.getElementById(`selisih_${i}`).textContent = selisih;

  let status = "";
  if (selisih > 0)
    status = '<span class="badge badge-danger">⚠️ Hilang: ' + selisih + "</span>";
  else if (selisih < 0)
    status =
      '<span class="badge badge-warning">⚠️ Lebih: ' + Math.abs(selisih) + "</span>";
  else status = '<span class="badge badge-success">✅ Sesuai</span>';

  document.getElementById(`status_${i}`).innerHTML = status;
}

function hitungOtomatis() {
  produkData.forEach((_, i) => hitung(i));
  updateSummary();
}

function updateSummary() {
  let totalStok = 0,
    totalTerjual = 0,
    totalKembali = 0,
    totalTersedia = 0;

  produkData.forEach((item, i) => {
    totalStok += item.stok;
    const t = parseInt(document.getElementById(`terjual_${i}`)?.value) || 0;
    const k = parseInt(document.getElementById(`kembali_${i}`)?.value) || 0;
    totalTerjual += t;
    totalKembali += k;
    totalTersedia += item.stok - t;
  });

  document.getElementById("totalItem").textContent = produkData.length;
  document.getElementById("stokAwal").textContent = totalStok;
  document.getElementById("totalTerjual").textContent = totalTerjual;
  document.getElementById("totalKembali").textContent = totalKembali;
  document.getElementById("stokTersedia").textContent = totalTersedia;
}

// ===== TAMBAH / HAPUS PRODUK =====
function openModalTambah() {
  document.getElementById("modalTambah").classList.add("active");
}
function closeModal() {
  document.getElementById("modalTambah").classList.remove("active");
}
function tambahProduk() {
  const kode = document.getElementById("inputKode").value.trim();
  const nama = document.getElementById("inputNama").value.trim();
  const stok = parseInt(document.getElementById("inputStok").value) || 0;
  const sampel = parseInt(document.getElementById("inputSampel").value) || 0;

  if (!kode || !nama)
    return alert("❌ Kode dan nama produk wajib diisi!");

  const newItem = {
    no: produkData.length + 1,
    kode,
    nama,
    stok,
    sampel,
  };

  produkData.push(newItem);
  renderTable();
  updateSummary();
  closeModal();
  alert("✅ Produk berhasil ditambahkan!");
}

function hapusProduk(no) {
  if (!confirm("Yakin ingin menghapus produk ini?")) return;
  produkData = produkData.filter((p) => p.no !== no);
  produkData.forEach((p, i) => (p.no = i + 1));
  renderTable();
  updateSummary();
}

// ===== EXPORT =====
function exportExcel() {
  const ws = XLSX.utils.json_to_sheet(produkData);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "Inventory");
  XLSX.writeFile(wb, `Laporan_Inventory_${new Date().toISOString().slice(0, 10)}.xlsx`);
}
function generatePDF() {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  doc.setFontSize(16);
  doc.text("Laporan Inventory Booth", 14, 20);
  doc.text(`Tanggal: ${document.getElementById("tanggalTransaksi").value}`, 14, 28);

  const data = produkData.map((p, i) => [
    i + 1,
    p.kode,
    p.nama,
    p.stok,
    document.getElementById(`terjual_${i}`)?.value || 0,
    document.getElementById(`kembali_${i}`)?.value || 0,
    document.getElementById(`tersedia_${i}`)?.textContent || "-",
    document.getElementById(`selisih_${i}`)?.textContent || "-",
    document.getElementById(`status_${i}`)?.innerText || "-",
  ]);

  doc.autoTable({
    head: [
      [
        "No",
        "Kode",
        "Nama Barang",
        "Stok Awal",
        "Terjual",
        "Kembali",
        "Tersedia",
        "Selisih",
        "Status",
      ],
    ],
    body: data,
    startY: 35,
    theme: "grid",
  });
  doc.save(`Laporan_Inventory_${new Date().toISOString().slice(0, 10)}.pdf`);
}
</script>
