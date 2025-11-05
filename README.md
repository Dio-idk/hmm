<!doctype html>
<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Nembak Crush — Kirim & Lihat Jawaban</title>
  <style>
    :root{--bg:#fbfbff;--card:#ffffff;--accent:#ef4444;--accent-2:#7c3aed;--muted:#6b7280}
    *{box-sizing:border-box}
    body{font-family:Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, Arial; background:linear-gradient(180deg,#fbfbff 0%, #f5f6fb 100%); margin:0;padding:28px;color:#0f172a}
    .wrap{max-width:920px;margin:0 auto}
    .card{background:var(--card);padding:18px;border-radius:14px;box-shadow:0 8px 30px rgba(15,23,42,.06);margin-bottom:18px}
    h1{margin:0 0 6px;font-size:20px}
    label{display:block;margin-top:10px;color:var(--muted);font-size:13px}
    input[type=text], textarea{width:100%;padding:10px;border:1px solid #e6e8f0;border-radius:10px;margin-top:6px;font-size:14px}
    button{background:var(--accent-2);color:white;padding:10px 14px;border:none;border-radius:10px;font-weight:600;cursor:pointer;margin-top:12px}
    .muted{color:var(--muted);font-size:13px}
    .linkbox{background:#f8fafc;padding:12px;border-radius:10px;margin-top:10px;font-size:13px;border:1px solid #eef2ff}
    .result-bubble{padding:14px;border-radius:10px;margin-top:10px;background:#fffbf0;border:1px solid #fde3b9}
    .row{display:flex;gap:10px}
    .small{font-size:13px;padding:8px;border-radius:8px}
    .copy{background:#eef2ff;color:#3730a3;border:none}
    footer{font-size:12px;color:var(--muted);margin-top:8px}
    .center{display:grid;place-items:center}
    .kbd{background:#eef2f7;padding:6px 8px;border-radius:6px;font-family:monospace}
    a.button-like{display:inline-block;padding:10px 14px;background:var(--accent);color:white;border-radius:10px;text-decoration:none}
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <h1>Nembak Crush — Buat Link Aman (Tanpa Server)</h1>
      <div class="muted">Buat link yang bisa dikirim ke crush. Mereka membuka link, pilih jawaban, lalu menerima link hasil yang dapat dikirim kembali padamu.</div>

      <label>Namamu</label>
      <input id="fromName" type="text" placeholder="Contoh: Raka" />

      <label>Nama crush</label>
      <input id="toName" type="text" placeholder="Contoh: Siska" />

      <label>Pesan untuk crush</label>
      <textarea id="message" rows="3">Hai, boleh nggak aku ngajak jalan bareng? 😊</textarea>

      <label>Opsi jawaban (pisahkan dengan koma)</label>
      <input id="options" type="text" placeholder="Iya, Boleh, Nanti dulu, Enggak" value="Iya,Boleh,Nanti dulu,Enggak" />

      <div style="display:flex;gap:8px;align-items:center;margin-top:10px">
        <button id="createBtn">Buat link untuk crush</button>
        <button id="createShortBtn" style="background:var(--accent);">Buat & buka langsung</button>
        <div class="muted" style="margin-left:auto">File: single-file HTML</div>
      </div>

      <div id="createArea"></div>
    </div>

    <div class="card">
      <h1>Cara kerja (singkat)</h1>
      <ol>
        <li>Kamu buat link lalu kirim ke crush.</li>
        <li>Crush buka link dan memilih jawaban.</li>
        <li>Crush menerima <em>result link</em> — mereka kirim link itu kembali ke kamu.</li>
        <li>Kamu buka link hasil untuk melihat jawabannya.</li>
      </ol>
      <div class="muted">Semua data disimpan di link (data di-encode dalam URL). Tidak ada server yang menyimpan jawaban.</div>
    </div>

    <div class="card center">
      <div class="muted">Siap mencoba? Isi formulir di atas dan buat link.</div>
    </div>

    <footer class="muted">Privasi: ini client-only. Jangan pakai untuk memaksa atau menipu orang lain. Jika ingin jawaban otomatis terlihat tanpa kirim link hasil, butuh backend (server).</footer>
  </div>

<script>
// Utility
function encode(obj){return btoa(encodeURIComponent(JSON.stringify(obj)))}
function decode(str){try{return JSON.parse(decodeURIComponent(atob(str)))}catch(e){return null}}
function genId(){return Math.random().toString(36).slice(2,10)}
function escapeHtml(s){return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')}

// Elements
const createBtn = document.getElementById('createBtn')
const createShortBtn = document.getElementById('createShortBtn')

createBtn.addEventListener('click', createLink)
createShortBtn.addEventListener('click', ()=>{createLink(true)})

function createLink(open=false){
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
      <div style="margin-top:8px"> <input class="kbd" id="linkField" style="width:100%" value="${answerLink}" readonly></div>
      <div style="margin-top:10px;display:flex;gap:8px">
        <button class="small copy" id="copyLink">Salin link</button>
        <button class="small" id="openLink">Buka sendiri</button>
        <a id="waShare" class="small" style="text-decoration:none;display:inline-block;padding:8px;border-radius:8px;background:#25D366;color:white">Bagikan via WA</a>
      </div>
      <div class="muted" style="margin-top:10px">Kirim link ini ke crush lewat chat. Setelah mereka jawab, mereka akan menerima link hasil yang bisa dikirim kembali ke kamu.</div>
    </div>
  `
  document.getElementById('createArea').innerHTML = html
  document.getElementById('copyLink').addEventListener('click', ()=>{navigator.clipboard.writeText(answerLink).then(()=>alert('Link disalin ke clipboard'))})
  document.getElementById('openLink').addEventListener('click', ()=>{location.href = answerLink})
  const wa = document.getElementById('waShare')
  wa.href = 'https://wa.me/?text='+encodeURIComponent('Hai! Buka link ini: '+answerLink)
  if(open) location.href = answerLink
}

// Handle incoming params
(function handleParams(){
  const url = new URL(location.href)
  const p = url.searchParams.get('p')
  const r = url.searchParams.get('r')
  if(p){
    const data = decode(p)
    if(!data){document.body.innerHTML='<div style="padding:40px">Link rusak atau tidak valid.</div>';return}
    renderAnswerPage(data)
  } else if(r){
    const resp = decode(r)
    if(!resp){document.body.innerHTML='<div style="padding:40px">Link hasil tidak valid.</div>';return}
    renderResultPage(resp)
  }
})()

// Answer page shown to crush
function renderAnswerPage(data){
  document.body.innerHTML = `
    <div style="max-width:680px;margin:40px auto;background:#fff;padding:22px;border-radius:12px;box-shadow:0 10px 30px rgba(20,20,50,.06)">
      <h2 style="margin:0 0 6px">Halo ${escapeHtml(data.to)}!</h2>
      <div class="muted">${escapeHtml(data.from)} mengirimi pesan untukmu:</div>
      <div style="margin-top:12px;padding:14px;border-radius:10px;background:#f7f7fb">"${escapeHtml(data.message)}"</div>
      <div style="margin-top:12px">Pilih jawabanmu:</div>
      <div id="opts" style="margin-top:10px;display:flex;flex-wrap:wrap;gap:8px"></div>
      <div style="margin-top:12px"><input id="otherText" placeholder="Jawaban lain (opsional)" style="width:100%;padding:10px;border-radius:10px;border:1px solid #e6e6e9"></div>
      <div style="margin-top:12px;display:flex;gap:8px;align-items:center">
        <button id="finishBtn" class="small" style="background:var(--accent)">Selesai</button>
        <a id="cancelBtn" class="small" href="javascript:location.href='${location.origin+location.pathname}'" style="background:#eee;color:#111">Batal</a>
      </div>
      <div style="margin-top:12px" class="muted">Setelah selesai, akan muncul link hasil yang dapat kamu kirimkan kembali ke ${escapeHtml(data.from)}.</div>
    </div>
  `
  const optsEl = document.getElementById('opts')
  data.options.forEach(opt=>{
    const btn = document.createElement('button')
    btn.textContent = opt
    btn.className = 'small'
    btn.addEventListener('click', ()=>{document.getElementById('otherText').value = opt})
    optsEl.appendChild(btn)
  })
  document.getElementById('finishBtn').addEventListener('click', ()=>finishAnswer(data))
}

function finishAnswer(orig){
  const other = document.getElementById('otherText')
  const chosen = other && other.value.trim() ? other.value.trim() : '(kosong)'
  const result = {id: orig.id, from: orig.from, to: orig.to, answer: chosen, ts: Date.now()}
  const encoded = encode(result)
  const resultLink = location.origin + location.pathname + '?r=' + encoded

  document.body.innerHTML = `
    <div style="max-width:680px;margin:40px auto;background:#fff;padding:22px;border-radius:12px;box-shadow:0 10px 30px rgba(20,20,50,.06)">
      <h2>Terima kasih!</h2>
      <div class="muted">Jawabanmu: <strong>${escapeHtml(result.answer)}</strong></div>
      <div style="margin-top:12px" class="linkbox">
        <div><strong>Link hasil (kirim ke ${escapeHtml(orig.from)}):</strong></div>
        <div style="margin-top:8px"><input style="width:100%;padding:10px;border-radius:8px;border:1px solid #e6e6ee" value="${resultLink}" readonly id="finalLink"></div>
        <div style="margin-top:8px;display:flex;gap:8px">
          <button class="small copy" id="copyFinal">Salin link hasil</button>
          <a class="small" id="waShareFinal" style="text-decoration:none;padding:8px;border-radius:8px;background:#25D366;color:white">Bagikan via WA</a>
          <button class="small" id="openFinal">Buka link hasil</button>
        </div>
      </div>
      <div style="margin-top:10px" class="muted">Kamu juga bisa copy link dan kirim lewat chat.</div>
    </div>
  `
  document.getElementById('copyFinal').addEventListener('click', ()=>{navigator.clipboard.writeText(resultLink).then(()=>alert('Link hasil disalin'))})
  document.getElementById('openFinal').addEventListener('click', ()=>{location.href = resultLink})
  document.getElementById('waShareFinal').href = 'https://wa.me/?text='+encodeURIComponent('Hai! Ini jawabanku: '+resultLink)
}

// Result page for proposer
function renderResultPage(resp){
  document.body.innerHTML = `
    <div style="max-width:720px;margin:40px auto;background:#fff;padding:22px;border-radius:12px;box-shadow:0 10px 30px rgba(20,20,50,.06)">
      <h2>Hasil dari ${escapeHtml(resp.to)} untuk ${escapeHtml(resp.from)}</h2>
      <div class="muted">Jawaban: <strong>${escapeHtml(resp.answer)}</strong></div>
      <div style="margin-top:8px" class="muted">Waktu: ${new Date(resp.ts).toLocaleString()}</div>
      <div style="margin-top:12px;display:flex;gap:8px">
        <button class="small" onclick="location.href='${location.origin+location.pathname}'">Buat lagi</button>
        <button class="small" onclick="navigator.clipboard.writeText('Jawabannya: ${escapeHtml(resp.answer)}').then(()=>alert('Jawaban disalin'))">Salin jawaban</button>
      </div>
    </div>
  `
}
</script>
</body>
</html>
