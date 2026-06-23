<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistem Informasi Madrasah</title>
  <script src="[https://cdn.tailwindcss.com](https://cdn.tailwindcss.com)"></script>
  <script src="[https://unpkg.com/html5-qrcode](https://unpkg.com/html5-qrcode)"></script>
</head>
<body class="bg-slate-100 min-h-screen font-sans">

  <script>
    // ⚠️ GANTI DENGAN URL WEB APP GOOGLE APPS SCRIPT HASIL DEPLOY ANDA
    const URL_API = "https://script.google.com/macros/s/AKfycbwwYVypvuZufCBXqOpTEhTnWmA4A4NNEIUVYFhh4wsRKmk4VCyBJJ0i3WTLVKz1WpS2/exec";
    // Fungsi jembatan pengganti 'google.script.run' ke Apps Script
    async function panggilAPI(actionName, payloadData = {}) {
      try {
        const respon = await fetch(URL_API, {
          method: "POST",
          body: JSON.stringify({
            action: actionName,
            payload: payloadData,
            token: localStorage.getItem("session_token"), // token login otomatis disisipkan jika ada
            username: payloadData.username || "",
            password: payloadData.password || ""
          })
        });
        return await respon.json();
      } catch (err) {
        console.error("Gagal menghubungi server database: ", err);
        return { sukses: false, pesan: "Koneksi ke database gagal. Periksa internet Anda." };
      }
    }
    
    // Variabel Penampung State Sesi Aplikasi
    let sesiUser = { token: localStorage.getItem("session_token"), nama: localStorage.getItem("session_nama"), role: localStorage.getItem("session_role") };
  </script>

  <div id="panel-login" class="flex items-center justify-center min-h-screen px-4">
    <div class="bg-white p-8 rounded-2xl shadow-xl w-full max-w-md border border-slate-200">
      <h2 class="text-2xl font-bold text-center text-slate-800 mb-2">Portal Madrasah</h2>
      <p class="text-slate-500 text-sm text-center mb-6">Silakan masuk menggunakan akun guru/admin Anda</p>
      
      <div class="space-y-4">
        <div>
          <label class="block text-sm font-semibold text-slate-700 mb-1">Username</label>
          <input type="text" id="log-user" class="w-full px-4 py-2.5 rounded-lg border border-slate-300 focus:outline-none focus:ring-2 focus:ring-blue-500" placeholder="Masukkan username">
        </div>
        <div>
          <label class="block text-sm font-semibold text-slate-700 mb-1">Password</label>
          <input type="password" id="log-pass" class="w-full px-4 py-2.5 rounded-lg border border-slate-300 focus:outline-none focus:ring-2 focus:ring-blue-500" placeholder="••••••••">
        </div>
        <button onclick="eksekusiLogin()" id="btn-login" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3 rounded-lg transition shadow-md">Masuk Aplikasi</button>
      </div>
    </div>
  </div>

  <div id="panel-dashboard" class="hidden">
    <nav class="bg-white shadow-md border-b border-slate-200 px-6 py-4 flex justify-between items-center">
      <div>
        <h1 class="text-lg font-bold text-slate-800">Sistem Informasi Madrasah</h1>
        <p class="text-xs text-slate-500">Selamat datang, <span id="nama-user-nav" class="font-medium text-blue-600">-</span> (<span id="role-user-nav">-</span>)</p>
      </div>
      <button onclick="eksekusiLogout()" class="px-4 py-1.5 border border-red-300 text-red-600 rounded-lg text-sm font-semibold hover:bg-red-50 transition">Keluar</button>
    </nav>

    <div class="max-w-4xl mx-auto p-4 space-y-6">
      
      <div class="bg-white p-6 rounded-2xl shadow border border-slate-200">
        <h3 class="text-lg font-bold text-slate-800 mb-1">Presensi Scanner QR Code</h3>
        <p class="text-sm text-slate-500 mb-4">Arahkan kamera HP ke kartu santri untuk melakukan pencatatan presensi kehadiran otomatis.</p>
        
        <div class="flex flex-col items-center">
          <button onclick="mulaiKameraScanner()" id="btn-start-camera" class="bg-green-600 hover:bg-green-700 text-white font-semibold px-6 py-2.5 rounded-xl shadow transition mb-4 flex items-center gap-2">
            Aktifkan Kamera Live
          </button>
          
          <div id="box-scanner-video" class="hidden w-full max-w-sm aspect-square bg-black rounded-xl overflow-hidden border-4 border-slate-800 shadow-inner relative">
            <div id="live-video-reader" class="w-full h-full"></div>
          </div>
          
          <div id="status-scanner" class="mt-3 text-sm text-center font-medium text-slate-600">Kamera dinonaktifkan</div>
        </div>
      </div>

    </div>
  </div>

  <script>
    // Cek status sesi login awal
    if (sesiUser.token) {
      tampilkanDashboard();
    }

    async function eksekusiLogin() {
      const u = document.getElementById("log-user").value;
      const p = document.getElementById("log-pass").value;
      const btn = document.getElementById("btn-login");
      
      if(!u || !p) return alert("Isi lengkap data!");
      btn.innerText = "Memproses...";
      btn.disabled = true;
      
      const res = await panggilAPI("login", { username: u, password: p });
      btn.innerText = "Masuk Aplikasi";
      btn.disabled = false;
      
      if (res.sukses) {
        localStorage.setItem("session_token", res.token);
        localStorage.setItem("session_nama", res.nama);
        localStorage.setItem("session_role", res.role);
        sesiUser = { token: res.token, nama: res.nama, role: res.role };
        tampilkanDashboard();
      } else {
        alert(res.pesan);
      }
    }

    function tampilkanDashboard() {
      document.getElementById("panel-login").classList.add("hidden");
      document.getElementById("panel-dashboard").classList.remove("hidden");
      document.getElementById("nama-user-nav").innerText = sesiUser.nama;
      document.getElementById("role-user-nav").innerText = sesiUser.role;
    }

    function eksekusiLogout() {
      localStorage.clear();
      location.reload();
    }

    // ==========================================================================
    // ENGINE SCANNER KAMERA LIVE STREAMING VIDEO (HTML5-QRCODE)
    // ==========================================================================
    let html5QrScanner = null;

    function mulaiKameraScanner() {
      const boxVideo = document.getElementById("box-scanner-video");
      const statusText = document.getElementById("status-scanner");
      const btnCamera = document.getElementById("btn-start-camera");
      
      boxVideo.classList.remove("hidden");
      statusText.innerText = "Membuka akses kamera video...";
      btnCamera.classList.add("hidden");

      // Inisialisasi library scanner pada element 'live-video-reader'
      html5QrScanner = new Html5Qrcode("live-video-reader");
      
      // Jalankan Kamera Belakang HP (environment) secara Live Video Stream
      html5QrScanner.start(
        { facingMode: "environment" }, 
        { fps: 12, qrbox: { width: 230, height: 230 } },
        (decodedText) => {
          // KETIKA QR CODE BERHASIL TERDETEKSI SECARA LIVE
          statusText.innerHTML = `<span class="text-blue-600 font-bold">QR Terbaca: ${decodedText}. Menyimpan ke database...</span>`;
          
          // Matikan kamera sebentar biar tidak menscan berulang-ulang dalam satu detik
          html5QrScanner.stop().then(() => {
            kirimDataPresensiKeSheets(decodedText);
          });
        },
        (errorMessage) => {
          // Callback pencarian kode di setiap frame video (abaikan agar console tidak penuh)
        }
      ).catch(err => {
        statusText.innerHTML = `<span class="text-red-600 font-bold">Gagal buka kamera: ${err}. Pastikan izin kamera diberikan atau ganti browser (gunakan Chrome/Safari).</span>`;
        boxVideo.classList.add("hidden");
        btnCamera.classList.remove("hidden");
      });
    }

    async function kirimDataPresensiKeSheets(idSantriMata) {
      const statusText = document.getElementById("status-scanner");
      const btnCamera = document.getElementById("btn-start-camera");
      
      // Kirim data ke Google Sheets secara Real-Time lewat API POST
      const res = await panggilAPI("simpanAbsen", {
        token: sesiUser.token,
        idSantri: idSantriMata,
        status: "Hadir" // Status default scanner, atau bisa Anda tambahkan opsi radio button status di atasnya
      });
      
      if(res.sukses) {
        statusText.innerHTML = `<span class="text-green-600 font-bold">✅ Berhasil Terarsip: ${res.pesan}</span>`;
      } else {
        statusText.innerHTML = `<span class="text-red-600 font-bold">❌ Gagal: ${res.pesan}</span>`;
      }
      
      // Tampilkan kembali tombol untuk scan murid berikutnya
      btnCamera.classList.remove("hidden");
      btnCamera.innerText = "Scan Murid Berikutnya";
    }
  </script>
</body>
</html>
