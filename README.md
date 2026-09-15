<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>استوديو الصور الاحترافي برو</title>
<style>
  * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, sans-serif; }
  body { background: #14141f; color: #fff; margin: 0; min-height: 100vh; }
  header { padding: 16px; text-align: center; background: #1e1e2e; border-bottom: 1px solid #333; }
  header h1 { margin: 0; font-size: 22px; }
  .container { max-width: 1300px; margin: 15px auto; padding: 0 12px; }
  .upload-area { border: 2px dashed #7c6cff; border-radius: 16px; padding: 70px 20px; text-align: center; cursor: pointer; }
  .upload-area:hover { background: #1e1e35; }
  .editor { display: none; gap: 15px; }
  .editor.show { display: flex; flex-wrap: wrap; }
  .canvas-box { flex: 1; min-width: 300px; min-height: 400px; background: repeating-conic-gradient(#2a2a2a 0 25%, #383838 0 50%) 0 0/24px 24px;
    border-radius: 12px; display: flex; align-items: center; justify-content: center; padding: 10px; overflow: hidden; }
  canvas { max-width: 100%; max-height: 78vh; border-radius: 4px; touch-action: none; }
  .controls { width: 330px; background: #1e1e2e; border-radius: 12px; padding: 18px; max-height: 80vh; overflow-y: auto; }
  .controls h3 { margin: 16px 0 8px; font-size: 14px; color: #7c6cff; border-bottom: 1px solid #333; padding-bottom: 5px; }
  .controls h3:first-child { margin-top: 0; }
  label { display: flex; justify-content: space-between; font-size: 13px; margin: 10px 0 2px; color: #ccc; }
  label span { color: #7c6cff; font-weight: bold; }
  input[type="range"] { width: 100%; accent-color: #7c6cff; }
  input[type="color"] { width: 100%; height: 38px; border: none; border-radius: 8px; background: none; cursor: pointer; padding: 0; }
  input[type="text"] { width: 100%; padding: 9px; border-radius: 8px; border: 1px solid #444; background: #14141f; color: #fff; margin-top: 6px; }
  button { width: 100%; padding: 11px; border-radius: 8px; border: none; font-size: 14px; cursor: pointer; margin-top: 7px; font-weight: bold; }
  .primary { background: #7c6cff; color: #fff; }
  .secondary { background: #333; color: #fff; }
  .btn-row { display: flex; gap: 6px; }
  .btn-row button { flex: 1; font-size: 12px; padding: 8px 4px; margin-top: 0; }
  .btn-row button.active { outline: 2px solid #7c6cff; }
  button:disabled { background: #444; color: #888; cursor: wait; }
  #status { margin: 8px 0; font-size: 12px; color: #aaa; text-align: center; min-height: 16px; }
  .presets { display: grid; grid-template-columns: repeat(3, 1fr); gap: 6px; }
  .presets button { margin-top: 0; padding: 8px 2px; font-size: 11px; background: #333; }
  .presets button.active { outline: 2px solid #7c6cff; }
  select { width: 100%; padding: 9px; border-radius: 8px; background: #14141f; color: #fff; border: 1px solid #444; margin-top: 6px; }
</style>
</head>
<body>
<header><h1>📸 استوديو الصور الاحترافي برو</h1></header>
<div class="container">
  <div class="upload-area" id="uploadArea">
    <p style="font-size:18px">📤 اضغط أو اسحب صورة العميل هنا</p>
    <small style="color:#888">بتتعالج على جهازك — خصوصية كاملة</small>
    <input type="file" id="fileInput" accept="image/*" hidden>
  </div>

  <div class="editor" id="editor">
    <div class="canvas-box"><canvas id="canvas"></canvas></div>
    <div class="controls">

      <h3>إزالة الخلفية</h3>
      <div class="btn-row">
        <button class="secondary active" id="engineFree">مجاني</button>
        <button class="secondary" id="enginePro">عالية الدقة 🔒</button>
      </div>
      <div id="apiKeySection" style="display:none">
        <input type="text" id="apiKey" placeholder="حط remove.bg API Key هنا">
        <small style="color:#888">مجاني 50 صورة شهريًا من remove.bg</small>
      </div>
      <button class="primary" id="removeBgBtn">✨ إزالة الخلفية</button>
      <div id="status"></div>

      <h3>🖌️ تدقيق يدوي</h3>
      <div class="btn-row">
        <button class="secondary active" id="brushOff">إيقاف</button>
        <button class="secondary" id="brushErase">🧽 مسح</button>
        <button class="secondary" id="brushRestore">↩️ رجوع</button>
      </div>
      <label>حجم الفرشاة <span id="vBrush">30</span></label>
      <input type="range" id="brushSize" min="5" max="150" value="30">

      <h3>الخلفية</h3>
      <div class="btn-row">
        <button class="secondary active" id="optColor">لون</button>
        <button class="secondary" id="optGradient">تدرج</button>
        <button class="secondary" id="optImage">صورة</button>
        <button class="secondary" id="optBlur">غباشة 🌫️</button>
      </div>
      <div id="colorSection"><input type="color" id="bgColor" value="#ffffff"></div>
      <div id="gradientSection" style="display:none">
        <input type="color" id="bgColor1" value="#667eea">
        <input type="color" id="bgColor2" value="#764ba2" style="margin-top:4px">
      </div>
      <div id="imageSection" style="display:none">
        <input type="file" id="bgImageInput" accept="image/*">
      </div>
      <div id="blurSection" style="display:none">
        <label>مستوى الغباشة <span id="vBlur">15</span></label>
        <input type="range" id="bgBlur" min="0" max="40" value="15">
      </div>

      <h3>الإضاءة والألوان</h3>
      <div class="presets" id="presets">
        <button data-f="100,100,100,0">طبيعي</button>
        <button data-f="108,115,115,10">دافئ ☀️</button>
        <button data-f="102,110,95,-15">بارد ❄️</button>
        <button data-f="105,125,85,5">سينمائي 🎬</button>
        <button data-f="100,100,0,0">أبيض وأسود</button>
        <button data-f="112,105,130,10">نابض</button>
      </div>
      <label>السطوع <span id="vBrightness">100%</span></label>
      <input type="range" id="brightness" min="30" max="200" value="100">
      <label>التباين <span id="vContrast">100%</span></label>
      <input type="range" id="contrast" min="30" max="200" value="100">
      <label>التشبع <span id="vSaturation">100%</span></label>
      <input type="range" id="saturation" min="0" max="250" value="100">
      <label>الحرارة <span id="vWarmth">0</span></label>
      <input type="range" id="warmth" min="-100" max="100" value="0">

      <h3>لمسات نهائية</h3>
      <label>🌗 ظل ناعم <span id="vShadow">0%</span></label>
      <input type="range" id="shadow" min="0" max="100" value="0">
      <label>🎞️ تظليل الحواف <span id="vVignette">0%</span></label>
      <input type="range" id="vignette" min="0" max="100" value="0">

      <h3>المقاس</h3>
      <select id="sizePreset">
        <option value="original">الأصلي</option>
        <option value="4x6">ورقي 4×6 (3:2)</option>
        <option value="square">مربع سوشيال (1:1)</option>
        <option value="story">ستوري (9:16)</option>
      </select>

      <h3>تصدير</h3>
      <button class="primary" id="downloadBtn">💾 تحميل PNG</button>
      <button class="secondary" id="downloadJpgBtn">💾 تحميل JPG</button>
      <button class="secondary" id="resetBtn">🔄 صورة جديدة</button>
    </div>
  </div>
</div>

<script type="module">
import { removeBackground } from 'https://cdn.jsdelivr.net/npm/@imgly/background-removal@1.5.5/+esm';

const $ = id => document.getElementById(id);
const canvas = $('canvas'), ctx = canvas.getContext('2d');
let subjectImg = null;   // الشخص بعد إزالة الخلفية (مع ألفا)
let bgImg = null;
let brushMode = 'off';   // off | erase | restore
let engine = 'free';
const brushMask = document.createElement('canvas'); // أبيض=احتفظ، أسود=امسح
const bctx = brushMask.getContext('2d');

// ====== رفع الصورة ======
$('uploadArea').onclick = () => $('fileInput').click();
$('uploadArea').ondragover = e => e.preventDefault();
$('uploadArea').ondrop = e => { e.preventDefault(); loadImage(e.dataTransfer.files[0]); };
$('fileInput').onchange = e => loadImage(e.target.files[0]);

function loadImage(file) {
  if (!file) return;
  const img = new Image();
  img.onload = () => {
    subjectImg = img;
    // جهز قناع الفرشاة (أبيض = كل الصورة تظهر)
    brushMask.width = img.width; brushMask.height = img.height;
    bctx.fillStyle = '#fff';
    bctx.fillRect(0, 0, brushMask.width, brushMask.height);
    $('uploadArea').style.display = 'none';
    $('editor').classList.add('show');
    draw();
  };
  img.src = URL.createObjectURL(file);
}

// ====== اختيار المحرك ======
$('engineFree').onclick = () => setEngine('free');
$('enginePro').onclick = () => setEngine('pro');
function setEngine(e) {
  engine = e;
  $('engineFree').classList.toggle('active', e === 'free');
  $('enginePro').classList.toggle('active', e === 'pro');
  $('apiKeySection').style.display = e === 'pro' ? 'block' : 'none';
}

// ====== إزالة الخلفية ======
$('removeBgBtn').onclick = async (ev) => {
  ev.target.disabled = true;
  $('status').textContent = '⏳ جاري المعالجة...';
  try {
    let blob;
    if (engine === 'pro') {
      const key = $('apiKey').value.trim();
      if (!key) { $('status').textContent = '⚠️ حط الـ API Key الأول'; ev.target.disabled = false; return; }
      const fd = new FormData();
      fd.append('image_file', subjectImg.src);
      fd.append('size', 'auto');
      const res = await fetch('https://api.remove.bg/v1.0/removebg', {
        method: 'POST',
        headers: { 'X-Api-Key': key },
        body: fd
      });
      if (!res.ok) throw new Error('API');
      blob = await res.blob();
    } else {
      blob = await removeBackground(subjectImg.src);
    }
    const img = new Image();
    img.onload = () => {
      subjectImg = img;
      bctx.fillStyle = '#fff';
      bctx.fillRect(0, 0, brushMask.width, brushMask.height);
      $('status').textContent = engine === 'pro' ? '✅ دقة عالية جاهزة' : '✅ جاهزة — استخدم الفرشاة للتدقيق';
      draw();
    };
    img.src = URL.createObjectURL(blob);
  } catch { $('status').textContent = '❌ حصل خطأ — جرب الصورة أوضح'; }
  ev.target.disabled = false;
};

// ====== الفرشاة اليدوية ======
[['brushOff','off'],['brushErase','erase'],['brushRestore','restore']].forEach(([id, m]) => {
  $(id).onclick = () => {
    brushMode = m;
    ['brushOff','brushErase','brushRestore'].forEach(x => $(x).classList.remove('active'));
    $(id).classList.add('active');
    canvas.style.cursor = m === 'off' ? 'default' : 'crosshair';
  };
});
$('brushSize').oninput = () => { $('vBrush').textContent = $('brushSize').value; };

let painting = false;
function brushPos(e) {
  const r = canvas.getBoundingClientRect();
  return {
    x: (e.clientX - r.left) * (canvas.width / r.width),
    y: (e.clientY - r.top) * (canvas.height / r.height)
  };
}
function paint(e) {
  if (!painting || brushMode === 'off' || !subjectImg) return;
  const p = brushPos(e);
  const size = +$('brushSize').value;
  bctx.globalCompositeOperation = brushMode === 'erase' ? 'source-over' : 'destination-over';
  bctx.filter = 'blur(' + (size / 8) + 'px)';
  bctx.fillStyle = brushMode === 'erase' ? '#000' : '#fff';
  bctx.beginPath();
  bctx.arc(p.x, p.y, size / 2, 0, Math.PI * 2);
  bctx.fill();
  bctx.filter = 'none';
  draw();
}
canvas.onpointerdown = e => { painting = true; paint(e); };
canvas.onpointermove = paint;
canvas.onpointerup = () => painting = false;
canvas.onpointerleave = () => painting = false;

// ====== الخلفية ======
let bgMode = 'color';
const modes = { color: 'optColor', gradient: 'optGradient', image: 'optImage', blur: 'optBlur' };
Object.entries(modes).forEach(([m, id]) => {
  $(id).onclick = () => {
    bgMode = m;
    Object.values(modes).forEach(x => $(x).classList.remove('active'));
    $(id).classList.add('active');
    $('colorSection').style.display = m === 'color' ? 'block' : 'none';
    $('gradientSection').style.display = m === 'gradient' ? 'block' : 'none';
    $('imageSection').style.display = m === 'image' ? 'block' : 'none';
    $('blurSection').style.display = m === 'blur' ? 'block' : 'none';
    draw();
  };
});
$('bgImageInput').onchange = e => {
  const f = e.target.files[0]; if (!f) return;
  bgImg = new Image();
  bgImg.onload = draw;
  bgImg.src = URL.createObjectURL(f);
};

// ====== الرسم ======
function currentFilter() {
  const b = $('brightness').value, c = $('contrast').value,
        s = $('saturation').value, w = $('warmth').value;
  let f = `brightness(${b}%) contrast(${c}%) saturate(${s}%)`;
  if (w > 0)      f += ` sepia(${w * 0.6}%) saturate(${100 + +w}%) hue-rotate(-${w * 0.3}deg)`;
  else if (w < 0) f += ` saturate(${100 + +w}%) hue-rotate(${-w * 0.4}deg)`;
  return f;
}

function drawSubject(targetCtx, w, h, filter) {
  // الشخص + قناع الفرشاة
  const tmp = document.createElement('canvas');
  tmp.width = w; tmp.height = h;
  const t = tmp.getContext('2d');
  t.drawImage(subjectImg, 0, 0, w, h);
  t.globalCompositeOperation = 'destination-in';
  t.drawImage(brushMask, 0, 0, w, h);
  targetCtx.filter = filter;
  targetCtx.drawImage(tmp, 0, 0);
  targetCtx.filter = 'none';
  return tmp;
}

function draw() {
  if (!subjectImg) return;
  // المقاسات
  let W = subjectImg.width, H = subjectImg.height;
  const preset = $('sizePreset').value;
  if (preset === '4x6')    { if (W / H > 1.5) W = H * 1.5; else H = W / 1.5; }
  if (preset === 'square') { H = W; }
  if (preset === 'story')  { H = W * 16 / 9; }
  canvas.width = W; canvas.height = H;

  const filter = currentFilter();

  // 1) الخلفية
  if (bgMode === 'image' && bgImg) {
    const s = Math.max(W / bgImg.width, H / bgImg.height);
    ctx.filter = 'blur(' + $('bgBlur').value + 'px)';
    ctx.drawImage(bgImg, (W - bgImg.width * s) / 2, (H - bgImg.height * s) / 2, bgImg.width * s, bgImg.height * s);
    ctx.filter = 'none';
  } else if (bgMode === 'gradient') {
    const g = ctx.createLinearGradient(0, 0, W, H);
    g.addColorStop(0, $('bgColor1').value); g.addColorStop(1, $('bgColor2').value);
    ctx.fillStyle = g; ctx.fillRect(0, 0, W, H);
  } else if (bgMode === 'blur' && bgImg) {
    // الخلفية الأصلية مبغبشة + الشخص واضح
    const s = Math.max(W / bgImg.width, H / bgImg.height);
    ctx.filter = 'blur(' + $('bgBlur').value + 'px) brightness(1.05)';
    ctx.drawImage(bgImg, (W - bgImg.width * s) / 2, (H - bgImg.height * s) / 2, bgImg.width * s, bgImg.height * s);
    ctx.filter = 'none';
  } else {
    ctx.fillStyle = $('bgColor').value;
    ctx.fillRect(0, 0, W, H);
  }

  // 2) ظل ناعم
  const sh = +$('shadow').value;
  if (sh > 0) {
    ctx.save();
    ctx.globalAlpha = sh / 220;
    ctx.filter = 'blur(12px) brightness(0%)';
    ctx.drawImage(subjectImg, W * 0.02, H * 0.02, W, H);
    ctx.restore();
    ctx.globalAlpha = 1;
  }

  // 3) الشخص
  drawSubject(ctx, W, H, filter);

  // 4) تظليل الحواف
  const v = +$('vignette').value;
  if (v > 0) {
    const g = ctx.createRadialGradient(W/2, H/2, Math.min(W,H)*0.4, W/2, H/2, Math.max(W,H)*0.75);
    g.addColorStop(0, 'rgba(0,0,0,0)');
    g.addColorStop(1, `rgba(0,0,0,${v/130})`);
    ctx.fillStyle = g;
    ctx.fillRect(0, 0, W, H);
  }
}

// ====== السلايدرات ======
[['brightness','%'],['contrast','%'],['saturation','%'],['warmth',''],['shadow','%'],['vignette','%'],['bgBlur','']].forEach(([id, suf]) => {
  if ($(id)) $(id).oninput = () => {
    $('v' + id[0].toUpperCase() + id.slice(1)).textContent = $(id).value + suf;
    draw();
  };
});
['bgColor','bgColor1','bgColor2'].forEach(id => $(id).oninput = draw);
$('sizePreset').onchange = draw;

// فلاتر جاهزة
document.querySelectorAll('#presets button').forEach(btn => {
  btn.onclick = () => {
    document.querySelectorAll('#presets button').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    const [b, c, s, w] = btn.dataset.f.split(',');
    $('brightness').value = b; $('contrast').value = c;
    $('saturation').value = s; $('warmth').value = w;
    $('vBrightness').textContent = b + '%';
    $('vContrast').textContent = c + '%';
    $('vSaturation').textContent = s + '%';
    $('vWarmth').textContent = w;
    draw();
  };
});

// ====== التحميل ======
function exportImage(type) {
  const a = document.createElement('a');
  a.download = 'studio-' + Date.now() + (type === 'png' ? '.png' : '.jpg');
  a.href = type === 'png'
    ? canvas.toDataURL('image/png')
    : canvas.toDataURL('image/jpeg', 0.95);
  a.click();
}
$('downloadBtn').onclick = () => exportImage('png');
$('downloadJpgBtn').onclick = () => exportImage('jpg');
$('resetBtn').onclick = () => location.reload();
</script>
</body>
</html>
