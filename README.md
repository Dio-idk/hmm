# hmm
<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Nembak Crush — Single File</title>
  <style>
    :root{--bg:#f7f7fb;--card:#fff;--accent:#7c3aed;--muted:#666}
    body{font-family:Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; background:var(--bg); margin:0; padding:32px; color:#111}
    .wrap{max-width:900px;margin:0 auto}
    .card{background:var(--card);padding:20px;border-radius:12px;box-shadow:0 6px 18px rgba(20,20,50,.06);margin-bottom:18px}
    h1{margin:0 0 8px;font-size:20px}
    label{display:block;margin-top:10px;color:var(--muted);font-size:13px}
    input[type=text], textarea{width:100%;padding:10px;border:1px solid #e6e6ee;border-radius:8px;margin-top:6px;font-size:14px}
    button{background:var(--accent);color:white;padding:10px 14px;border:none;border-radius:10px;font-weight:600;cursor:pointer;margin-top:12px}
    .muted{color:var(--muted);font-size:13px}
    .linkbox{background:#f3f4f8;padding:10px;border-radius:8px;margin-top:10px;font-size:13px}
    .result-bubble{padding:14px;border-radius:10px;margin-top:10px;background:#fef9ef;border:1px solid #fde3b9}
    .row{display:flex;gap:10px}
    .small{font-size:13px;padding:8px;border-radius:8px}
    .copy{background:#eef2ff;color:#3730a3}
    footer{font-size:12px;color:var(--muted);margin-top:8px}
    .center{display:grid;place-items:center}
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <h1>Nembak Crush — Buat Link</h1>
      <div class="muted">Buat link yang bisa kamu kirim ke crush. Mereka membuka link, jawab, lalu akan mendapatkan link hasil yang bisa dikirim kembali ke kamu.</div>

      <label>Namamu</label>
      <input id="fromName" type="text" placeholder="Contoh: Raka" />

      <label>Nama crush</label>
      <input id="toName" type="text" placeholder="Contoh: Siska" />

      <label>Pertanyaan / Pesan (yang akan dilihat crush)</label>
      <textarea id="message" rows="3">Hai, boleh nggak aku ngajak jalan bareng? 😊</textarea>

      <label>Opsi jawaban (pisahkan dengan koma)</label>
      <input id="options" type="text" placeholder="Iya, Boleh, Nanti dulu, Enggak" value="Iya,Boleh,Nanti dulu,Enggak" />

      <button id="createBtn">Buat link untuk crush</button>

      <div id="createArea"></div>
    </div>

    <div class="card">
      <h1>Jika kamu menerima link dari crush (cara jawab)</h1>
      <div class="muted">Jika kamu adalah crush yang menerima link, buka link tersebut, pilih jawabannya, lalu klik "Selesai". Setelah selesai akan muncul sebuah link hasil — kirim link itu ke si pembuat untuk memberitahukan jawabannya.</div>
      <div class="muted" style="margin-top:8px">Catatan: aplikasi ini tanpa server. Data tidak tersimpan otomatis ke pembuat kecuali crush mengirim link hasilnya kembali.</div>
    </div>

    <div class="card center">
      <div class="muted">Mau coba sekarang?</div>
      <a href="#create" style="margin-top:8px; text-decoration:none"><button>Mulai Buat</button></a>
    </div>

    <footer class="muted">Privasi: tidak ada data yang dikirim ke server. Semua tersimpan di link yang dihasilkan. Jangan gunakan untuk hal ilegal atau memaksa orang lain.</footer>
  </div>

<script>
// Simple single-file implementation without backend.
function encode(obj){return btoa(encodeURIComponent(JSON.stringify(obj)))}
function decode(str){try{return JSON.parse(decodeURIComponent(atob(str)))}catch(e){return null}}

function genId(){return Math.random().toString(36).slice(2,10)}

document.getElementById('createBtn').addEventListener('click', ()=>{
  const from = document.getElementById('fromName').value.trim() || 'Seseorang'
  const to = document.getElementById('toName').value.trim() || 'Crush'
  const message = document.getElementById('message').value.trim() || 'Hai :)'
  const options = document.getElementById('options').value.split(',').map(s=>s.trim()).filter(Boolean)
  const id = genId()
  const payload = {id, from, to, message, options}
  const encoded = encode(payload)
  const answerLink = location.origin + location.pathname + '?p=' + encoded

  const html = `
    <div class="linkbox">
      <div><strong>Link untuk dikirim ke crush:</strong></div>
      <div style="margin-top:6px"> <input class="small" id="linkField" style="width:100%" value="${answerLink}" readonly></div>
      <div style="margin-top:8px;display:flex;gap:8px">
        <button class="small copy" id="copyLink">Salin link</button>
        <button class="small" id="openLink">Buka sendiri</button>
      </div>
      <div class="muted" style="margin-top:8px">Instruksi: kirim link ke crush. Setelah mereka jawab, mereka akan menerima link hasil yang bisa mereka kirim balik ke kamu.</div>
    </div>
  `
  document.getElementById('createArea').innerHTML = html
  document.getElementById('copyLink').addEventListener('click', ()=>{
    navigator.clipboard.writeText(answerLink).then(()=>alert('Link disalin ke clipboard'))
  })
  document.getElementById('openLink').addEventListener('click', ()=>{location.href = answerLink})
})

// If page opened with ?p=... it's the answer page (for crush)
(function handleParams(){
  const url = new URL(location.href)
  const p = url.searchParams.get('p')
  const r = url.searchParams.get('r')
  if(p){
    const data = decode(p)
    if(!data){document.body.innerHTML='<div style="padding:40px">Link rusak atau tidak valid.</div>';return}
    renderAnswerPage(data)
  } else if(r){
    // result view (proposer opens a result link to see answer)
    const resp = decode(r)
    if(!resp){document.body.innerHTML='<div style="padding:40px">Link hasil tidak valid.</div>';return}
    renderResultPage(resp)
  }
})()

function renderAnswerPage(data){
  document.body.innerHTML = `
    <div style="max-width:700px;margin:40px auto;background:#fff;padding:22px;border-radius:12px;box-shadow:0 10px 30px rgba(20,20,50,.06)">
      <h2 style="margin:0 0 6px">Halo ${data.to}!</h2>
      <div class="muted">${data.from} mengirim pesan untukmu:</div>
      <div style="margin-top:12px;padding:14px;border-radius:10px;background:#f7f7fb">"${escapeHtml(data.message)}"</div>
      <div style="margin-top:12px">Pilih jawabanmu:</div>
      <div id="opts" style="margin-top:10px"></div>
      <div style="margin-top:14px" class="muted">Setelah memilih dan klik selesai, akan muncul link yang dapat kamu kirimkan kembali ke ${data.from}.</div>
    </div>
  `
  const optsEl = document.getElementById('opts')
  data.options.forEach((opt, i)=>{
    const btn = document.createElement('button')
    btn.textContent = opt
    btn.className = 'small'
    btn.style.marginRight = '8px'
    btn.addEventListener('click', ()=>selectOption(opt))
    optsEl.appendChild(btn)
  })
  const other = document.createElement('div')
  other.style.marginTop='12px'
  other.innerHTML = `<input id="otherText" placeholder="Jawaban lain (opsional)" style="width:60%;padding:8px;border-radius:8px;border:1px solid #e6e6ee"> <button id="finishBtn" class="small">Selesai</button>`
  optsEl.appendChild(other)
  document.getElementById('finishBtn').addEventListener('click', ()=>finishAnswer())
}

let chosen = null
function selectOption(opt){chosen = opt; alert('Dipilih: ' + opt)}
function finishAnswer(){
  const other = document.getElementById('otherText')
  if(other && other.value.trim()){chosen = other.value.trim()}
  if(!chosen){if(!confirm('Kamu belum memilih jawaban. Kirim kosong?')) return}
  // build result payload
  const url = new URL(location.href)
  const p = url.searchParams.get('p')
  const orig = decode(p)
  const result = {id: orig.id, from: orig.from, to: orig.to, answer: chosen || '(kosong)', ts: Date.now()}
  const encoded = encode(result)
  const resultLink = location.origin + location.pathname + '?r=' + encoded
  document.body.innerHTML = `
    <div style="max-width:700px;margin:40px auto;background:#fff;padding:22px;border-radius:12px;box-shadow:0 10px 30px rgba(20,20,50,.06)">
      <h2>Terima kasih!</h2>
      <div class="muted">Jawabanmu: <strong>${escapeHtml(result.answer)}</strong></div>
      <div style="margin-top:12px" class="linkbox">
        <div><strong>Link hasil (kirim ke ${orig.from}):</strong></div>
        <div style="margin-top:8px"><input style="width:100%;padding:8px;border-radius:8px;border:1px solid #e6e6ee" value="${resultLink}" readonly id="finalLink"></div>
        <div style="margin-top:8px;display:flex;gap:8px"><button class="small copy" id="copyFinal">Salin link hasil</button> <button class="small" id="openFinal">Buka link hasil</button></div>
      </div>
      <div style="margin-top:10px" class="muted">Kamu juga bisa copy link dan kirim lewat chat.</div>
    </div>
  `
  document.getElementById('copyFinal').addEventListener('click', ()=>{navigator.clipboard.writeText(resultLink).then(()=>alert('Link hasil disalin'))})
  document.getElementById('openFinal').addEventListener('click', ()=>{location.href = resultLink})
}

function renderResultPage(resp){
  document.body.innerHTML = `
    <div style="max-width:700px;margin:40px auto;background:#fff;padding:22px;border-radius:12px;box-shadow:0 10px 30px rgba(20,20,50,.06)">
      <h2>Hasil dari ${escapeHtml(resp.to)} untuk ${escapeHtml(resp.from)}</h2>
      <div class="muted">Jawaban: <strong>${escapeHtml(resp.answer)}</strong></div>
      <div style="margin-top:8px" class="muted">Waktu: ${new Date(resp.ts).toLocaleString()}</div>
      <div style="margin-top:12px">
        <button class="small" onclick="location.href='${location.origin+location.pathname}'">Buat lagi</button>
      </div>
    </div>
  `
}

function escapeHtml(s){return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')}
</script>
</body>
</html>
