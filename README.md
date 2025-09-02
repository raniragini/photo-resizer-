# photo-resizer- 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Photo Resizer – JPG, Fixed px & KB</title>
  <style>
    :root{
      --bg:#0f172a; /* slate-900 */
      --panel:#111827; /* gray-900 */
      --muted:#94a3b8; /* slate-400 */
      --text:#e5e7eb; /* gray-200 */
      --accent:#38bdf8; /* sky-400 */
      --accent-2:#a78bfa; /* violet-400 */
      --good:#10b981; /* emerald-500 */
      --warn:#f59e0b; /* amber-500 */
      --err:#ef4444; /* red-500 */
      --card:#0b1220; /* dark panel */
      --border:#1f2937; /* gray-800 */
    }
    *{box-sizing:border-box}
    body{
      margin:0; font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, Noto Sans, Helvetica Neue, Arial;
      background: radial-gradient(1200px 600px at 20% -10%, rgba(56,189,248,.15), transparent 60%),
                  radial-gradient(800px 500px at 100% 10%, rgba(167,139,250,.12), transparent 60%),
                  var(--bg);
      color:var(--text);
      min-height:100dvh; display:flex; align-items:center; justify-content:center; padding:24px;
    }
    .app{
      width:min(1100px, 100%); display:grid; gap:16px; grid-template-columns: 1.2fr .8fr; align-items:start;
    }
    @media(max-width:980px){.app{grid-template-columns:1fr}}
    .card{
      background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)), var(--card);
      border:1px solid var(--border);
      border-radius:18px; padding:18px; box-shadow: 0 10px 30px rgba(0,0,0,.25);
    }
    h1{font-size:clamp(22px,3.4vw,32px); margin:0 0 8px; letter-spacing:.3px}
    p.caption{margin:0 0 14px; color:var(--muted)}

    .inputs{display:grid; grid-template-columns: repeat(2, 1fr); gap:12px}
    .inputs .full{grid-column:1/-1}
    label{font-size:13px; color:var(--muted); display:block; margin:0 0 6px}
    input[type="number"], input[type="text"], input[type="file"], select{
      width:100%; padding:12px 12px; background:#0a0f1b; border:1px solid var(--border); color:var(--text);
      border-radius:12px; outline:none; transition:border .2s, box-shadow .2s; font-size:14px;
    }
    input[type="file"]{padding:10px}
    input:focus, select:focus{border-color:var(--accent); box-shadow:0 0 0 3px rgba(56,189,248,.15)}
    .row{display:flex; gap:10px; align-items:center}
    .row .grow{flex:1}
    .hint{font-size:12px; color:var(--muted)}

    button{
      appearance:none; border:none; padding:12px 16px; border-radius:12px; font-weight:600; cursor:pointer;
      background:linear-gradient(135deg, var(--accent), var(--accent-2)); color:#001018; letter-spacing:.2px;
      transition:transform .04s ease, filter .2s ease; width:100%;
    }
    button:hover{filter:brightness(1.05)}
    button:active{transform:translateY(1px)}
    button.secondary{
      background:#0b1220; color:var(--text); border:1px solid var(--border);
    }

    .preview{
      display:grid; grid-template-columns:1fr 1fr; gap:12px;
    }
    @media(max-width:720px){.preview{grid-template-columns:1fr}}
    .pane{position:relative; border-radius:14px; overflow:hidden; background:#05080f; border:1px dashed #233046; min-height:260px}
    .pane header{display:flex; justify-content:space-between; align-items:center; padding:10px 12px; border-bottom:1px solid #101828; color:var(--muted); font-size:12px}
    .pane main{display:flex; align-items:center; justify-content:center; padding:12px}
    .pane img, .pane canvas{max-width:100%; height:auto; display:block}
    .stats{display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-top:10px; font-size:13px; color:var(--muted)}
    .badge{display:inline-flex; align-items:center; gap:6px; padding:6px 10px; border-radius:999px; background:#0e1626; border:1px solid var(--border); color:var(--text)}
    .ok{color:var(--good)} .warn{color:var(--warn)} .err{color:var(--err)}

    .footer{display:flex; gap:10px; align-items:center}
    a.download{display:inline-flex; align-items:center; justify-content:center; gap:8px; text-decoration:none; width:100%;}
    .tiny{font-size:11px; color:var(--muted)}
  </style>
</head>
<body>
  <div class="app">
    <!-- LEFT: Controls -->
    <section class="card">
      <h1>Resize Photo to <span style="color:var(--accent)">JPG</span> with exact <span style="color:var(--accent-2)">Width × Height (px)</span> and target <span style="color:var(--accent)">KB</span></h1>
      <p class="caption">Upload an image → set output width, height (px) and size (KB) → get a compressed JPG that matches your limits. Everything runs in your browser.</p>

      <div class="inputs">
        <div class="full">
          <label>Upload image</label>
          <input id="file" type="file" accept="image/*" />
        </div>

        <div>
          <label>Output width (px)</label>
          <input id="outW" type="number" min="10" step="1" placeholder="e.g., 200" />
        </div>
        <div>
          <label>Output height (px)</label>
          <input id="outH" type="number" min="10" step="1" placeholder="e.g., 230" />
        </div>

        <div>
          <label>Target size (KB)</label>
          <input id="targetKB" type="number" min="1" step="1" placeholder="e.g., 40" />
          <div class="hint">We aim for ±2 KB. If impossible by quality alone, dimensions will shrink slightly.</div>
        </div>
        <div>
          <label>File name</label>
          <input id="fileName" type="text" placeholder="output.jpg" value="resized.jpg" />
        </div>

        <div class="full row">
          <button id="processBtn">Resize & Compress</button>
          <button id="resetBtn" class="secondary">Reset</button>
        </div>
      </div>

      <div class="stats" id="info"></div>

      <div class="footer" style="margin-top:12px">
        <a class="download" id="downloadLink" href="#" download>
          <button id="downloadBtn" class="secondary" disabled>Download JPG</button>
        </a>
      </div>
      <div class="tiny" style="margin-top:6px">Tip: For passport-style photos, many sites need 200×230 px and 20–50 KB. Set width/height and target KB accordingly.</div>
    </section>

    <!-- RIGHT: Previews -->
    <section class="card">
      <div class="preview">
        <div class="pane">
          <header><span>Original</span><span id="origMeta">—</span></header>
          <main><img id="origImg" alt="Original preview" /></main>
        </div>
        <div class="pane">
          <header><span>Output (JPG)</span><span id="outMeta">—</span></header>
          <main><canvas id="canvas" hidden></canvas><img id="outImg" alt="Output preview" /></main>
        </div>
      </div>
    </section>
  </div>

  <script>
    const el = id => document.getElementById(id);
    const fileInput = el('file');
    const outW = el('outW');
    const outH = el('outH');
    const targetKB = el('targetKB');
    const processBtn = el('processBtn');
    const resetBtn = el('resetBtn');
    const downloadBtn = el('downloadBtn');
    const downloadLink = el('downloadLink');
    const fileName = el('fileName');
    const info = el('info');

    const origImg = el('origImg');
    const outImg = el('outImg');
    const canvas = el('canvas');
    const origMeta = el('origMeta');
    const outMeta = el('outMeta');

    let originalBlob = null;

    function formatBytes(bytes){
      const kb = bytes / 1024;
      if (kb < 1024) return `${kb.toFixed(1)} KB`;
      return `${(kb/1024).toFixed(2)} MB`;
    }

    async function readAsDataURL(file){
      return new Promise((res, rej)=>{
        const fr = new FileReader();
        fr.onload = ()=> res(fr.result);
        fr.onerror = rej;
        fr.readAsDataURL(file);
      });
    }

    function drawToCanvas(img, w, h){
      const ctx = canvas.getContext('2d');
      canvas.width = Math.max(1, Math.round(w));
      canvas.height = Math.max(1, Math.round(h));
      // High-quality scaling hints
      ctx.imageSmoothingEnabled = true;
      ctx.imageSmoothingQuality = 'high';
      ctx.clearRect(0,0,canvas.width, canvas.height);
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      return canvas;
    }

    function canvasToBlob(canvas, quality=0.92, type='image/jpeg'){
      return new Promise((resolve)=>{
        canvas.toBlob((blob)=> resolve(blob), type, quality);
      });
    }

    async function compressToTargetKB(canvas, targetBytes){
      const TOL = 2 * 1024; // ±2KB tolerance
      let qMin = 0.1, qMax = 0.95, best = null;
      for(let i=0;i<18;i++){
        const q = (qMin + qMax) / 2;
        const blob = await canvasToBlob(canvas, q);
        if(!blob) break;
        if (blob.size <= targetBytes + TOL){
          best = {blob, q};
          // try higher quality to approach target from below
          qMin = q;
        } else {
          // too big – decrease quality
          qMax = q;
        }
      }
      if (best) return best.blob;
      // if still too large at very low quality, return last low-quality attempt
      return await canvasToBlob(canvas, qMin);
    }

    async function ensureFitsKB(img, targetBytes, w, h){
      // Try with requested dimensions first
      drawToCanvas(img, w, h);
      let blob = await compressToTargetKB(canvas, targetBytes);
      if (blob.size <= targetBytes + 2048) return {blob, w, h};

      // If we can't hit target with quality alone, gradually downscale until it fits or reaches tiny size
      let scale = 0.95; // shrink by 5% steps
      let curW = w, curH = h;
      for (let i=0; i<25 && (curW > 40 && curH > 40); i++){
        curW = Math.max(1, Math.round(curW * scale));
        curH = Math.max(1, Math.round(curH * scale));
        drawToCanvas(img, curW, curH);
        blob = await compressToTargetKB(canvas, targetBytes);
        if (blob.size <= targetBytes + 2048){
          return {blob, w:curW, h:curH};
        }
      }
      // Return best-effort (smallest we got)
      return {blob, w:canvas.width, h:canvas.height};
    }

    function setInfo(html){ info.innerHTML = html; }

    function validate(){
      const errors = [];
      if (!fileInput.files || !fileInput.files[0]) errors.push('Please choose an image.');
      const w = parseInt(outW.value, 10);
      const h = parseInt(outH.value, 10);
      const kb = parseInt(targetKB.value, 10);
      if (!Number.isFinite(w) || w <= 0) errors.push('Width must be a positive number.');
      if (!Number.isFinite(h) || h <= 0) errors.push('Height must be a positive number.');
      if (!Number.isFinite(kb) || kb <= 0) errors.push('Target KB must be a positive number.');
      return {ok: errors.length===0, errors, w, h, kb};
    }

    fileInput.addEventListener('change', async () =>{
      const file = fileInput.files?.[0];
      if (!file) return;
      originalBlob = file;
      const url = await readAsDataURL(file);
      origImg.src = url;
      origImg.onload = ()=>{
        origMeta.textContent = `${origImg.naturalWidth}×${origImg.naturalHeight}px • ${formatBytes(file.size)}`;
      };
      // Clear previous output
      outImg.src = '';
      outMeta.textContent = '—';
      downloadBtn.disabled = true;
      setInfo('');
    });

    resetBtn.addEventListener('click', () => {
      fileInput.value = '';
      outW.value = '';
      outH.value = '';
      targetKB.value = '';
      fileName.value = 'resized.jpg';
      origImg.src = '';
      outImg.src = '';
      origMeta.textContent = '—';
      outMeta.textContent = '—';
      downloadBtn.disabled = true;
      setInfo('');
    });

    processBtn.addEventListener('click', async () =>{
      const {ok, errors, w, h, kb} = validate();
      if (!ok){
        setInfo(errors.map(e=>`<span class="badge err">${e}</span>`).join(' '));
        return;
      }
      if (!originalBlob){
        setInfo('<span class="badge err">Please choose an image.</span>');
        return;
      }

      processBtn.disabled = true; processBtn.textContent = 'Working…';
      setInfo('<span class="badge">Processing…</span>');

      try{
        const img = new Image();
        img.src = await readAsDataURL(originalBlob);
        await img.decode();
        const targetBytes = kb * 1024;

        const result = await ensureFitsKB(img, targetBytes, w, h);
        const blob = result.blob;

        const outURL = URL.createObjectURL(blob);
        outImg.src = outURL;
        outMeta.textContent = `${result.w}×${result.h}px • ${formatBytes(blob.size)}`;

        // Prepare download
        const name = (fileName.value || 'resized.jpg').replace(/\s+/g,'_');
        downloadLink.href = outURL;
        downloadLink.download = name.endsWith('.jpg') || name.endsWith('.jpeg') ? name : `${name}.jpg`;
        downloadBtn.disabled = false;

        const within = Math.abs(blob.size - targetBytes) <= 2048;
        setInfo(`
          <span class="badge ${within ? 'ok' : 'warn'}">
            ${within ? 'Success' : 'Best effort'}: ${formatBytes(blob.size)} (target ≈ ${kb} KB)
          </span>
          <span class="badge">Quality tuned to fit; dimensions ${within ? 'kept' : 'may be slightly reduced'}.</span>
        `);
      } catch(err){
        console.error(err);
        setInfo(`<span class="badge err">Error: ${err?.message || err}</span>`);
      } finally{
        processBtn.disabled = false; processBtn.textContent = 'Resize & Compress';
      }
    });
  </script>
  <footer>
    <p class="text-center text-gray-700 mt-4">By ragini❤️</p>
  </footer>
</body>
 

</html>

