<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>الاستوديو الاحترافي</title>
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
    border-radius: 12px; display: flex; align-items: center; justify-content: center; padding: 10px; }
  canvas { max-width: 100%; max-height: 78vh; border-radius: 4px; touch-action: none; }
  .controls { width: 330px; background: #1e1e2e; border-radius: 12px; padding: 18px; max-height: 80vh; overflow-y: auto; }
  .controls h3 { margin: 16px 0 8px; font-size: 14px; color: #7c6cff; border-bottom: 1px solid #333; padding-bottom: 5px; }
  .controls h3:first-child { margin-top: 0; }
  label { display: flex; justify-content: space-between; font-size: 13px; margin: 10px 0 2px; color: #ccc; }
  label span { color: #7c6cff; font-weight: bold; }
  input[type="range"] { width: 100%; accent-color: #7c6cff; }
  input[type="color"] { width: 100%; height: 38px; border: none; border-radius: 8px; background: none; cursor: pointer; padding: 0; }
  button { width: 100%; padding: 11px; border-radius: 8px; border: none; font-size: 14px; cursor: pointer; margin-top: 7px; font-weight: bold; }
  .primary { background: #7c6cff; color: #fff; }
  .secondary { background: #333; color: #fff; }
  .btn-row { display: flex; gap: 6px; }
  .btn-row button { flex: 1; font-size: 12px; padding: 8px 4px; margin-top: 0; }
  .btn-row button.active { outline: 2px solid #7c6cff; }
  button:disabled { background: #444; color: #888; cursor: wait; }
  #status { margin: 8px 0; font-size: 12px; color: #aaa; text-align: center; min-height: 16px; }
  .presets { display: grid; grid-template-columns: repeat(3, 1fr); gap: 6px; }
  .presets button { margin-top: 0; padding: 9px 2px; font-size: 11px; background: #333; }
  .presets button.active { outline: 2px solid #7c6cff; }
  select { width: 100%; padding: 9px; border-radius: 8px; background: #14141f; color: #fff; border: 1px solid #444; margin-top: 6px; }
</style>
</head>
<body>
<header><h1>🎬 الاستوديو الاحترافي — صور</h1></header>
<div class="container">
  <div class="upload-area" id="uploadArea">
    <p style="font-size:18px">📤 اضغط أو اسحب الصورة هنا</p>
    <small style="color:#888">كل المعالجة على جهازك — خصوصية كاملة</small>
    <input type="file" id="fileInput" accept="image/*" hidden>
  </div>

  <div class="editor" id="editor">
    <div class="canvas-box"><canvas id="canvas"></canvas></div>
    <div class="controls">

      <h3>إزالة الخلفية</h3>
      <button class="primary" id="removeBgBtn">✨ AI إزالة الخلفية</button>
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
      </div>
      <div id="colorSection"><input type="color" id="bgColor" value="#ffffff"></div>
      <div id="gradientSection" style="display:none">
        <input type="color" id="bgColor1" value="#667eea">
        <input type="color" id="bgColor2" value="#764ba2" style="margin-top:4px">
      </div>
      <div id="imageSection" style="display:none">
        <input type="file" id="bgImageInput" accept="image/*">
      </div>

      <h3>🎞️ مؤثرات جاهزة</h3>
      <div class="presets" id="fxPresets"></div>

      <h3>الإضاءة والألوان</h3>
      <label>السطوع <span id="vBrightness">100%</span></label>
      <input type="range" id="brightness" min="30" max="200" value="100">
      <label>التباين <span id="vContrast">100%</span></label>
      <input type="range" id="contrast" min="30" max="200" value="100">
      <label>التشبع <span id="vSaturation">100%</span></label>
      <input type="range" id="saturation" min="0" max="250" value="100">
      <label>الحرارة <span id="vWarmth">0</span></label>
      <input type="range" id="warmth" min="-100" max="100" value="0">
      <label>🌗 ظل ناعم <span id="vShadow">0%</span></label>
      <input type="range" id="shadow" min="0" max="100" value="0">
      <label>🎞️ تظليل الحواف <span id="vVignette">0%</span></label>
      <input type="range" id="vignette" min="0" max="100" value="0">

      <h3>المقاس</h3>
      <select id="sizePreset">
        <option value="original">الأصلي</option>
        <option value="4x6">ورقي 3:2</option>
        <option value="square">مربع 1:1</option>
        <option value="story">ستوري 9:16</option>
      </select>

      <h3>تصدير</h3>
      <button class="primary" id="downloadBtn">💾 تحميل PNG</button>
      <button class="secondary" id="downloadJpgBtn">💾 تحميل JPG</button>
      <button class="secondary" id="resetBtn">🔄 صورة جديدة</button>
    </div>
  </div>
</div>

<script>
const $ = id => document.getElementById(id);
const canvas = $('canvas'), ctx = canvas.getContext('2d');
let subjectImg = null, bgImg = null, brushMode = 'off', currentFx = 'none';
const brushMask = document.createElement('canvas');
const bctx = brushMask.getContext('2d');

// ===== المؤثرات الجاهزة (فلتر + تسريب ضوئي) =====
const FX = {
  none:      { name: 'بدون',   filter: '',                                                                 leak: null },
  warm:      { name: 'دافئ ☀️', filter: 'brightness(106%) contrast(112%) saturate(112%) sepia(12%)',  leak: { c: '255,180,80', a: .18 } },
  cold:      { name: 'بارد ❄️', filter: 'brightness(103%) contrast(108%) saturate(88%) hue-rotate(12deg)', leak: { c: '80,160,255', a: .14 } },
  cinema:    { name: 'سينمائي', filter: 'contrast(122%) saturate(85%) brightness(103%)',              leak: { c: '255,120,40', a: .22 } },
  bw:        { name: 'أبيض/أسود', filter: 'grayscale(100%) contrast(118%) brightness(104%)',           leak: null },
  vintage:   { name: 'فينتاج 📼', filter: 'sepia(45%) contrast(92%) brightness(103%) saturate(85%)',   leak: { c: '255,200,120', a: .25 } },
  vhs:       { name: 'VHS 📼',  filter: 'saturate(130%) contrast(105%) hue-rotate(-8deg) blur(.4px)',  leak: { c: '255,80,200', a: .12 } },
  neon:      { name: 'نيون 💜', filter: 'contrast(135%) saturate(160%) brightness(98%)',              leak: { c: '160,60,255', a: .2 } },
  soft:      { name: 'ناعم 🤍', filter: 'brightness(108%) contrast(92%) saturate(105%) blur(.3px)',    leak: { c: '255,255,255', a: .15 } },
  dramatic:  { name: 'درامي 🎭', filter: 'contrast(145%) brightness(92%) saturate(110%)',              leak: null },
  golden:    { name: 'ذهبي ✨', filter: 'sepia(25%) saturate(130%) contrast(108%) brightness(106%)',   leak: { c: '255,190,60', a: .3 } },
};

const fxBox = $('fxPresets');
Object.entries(FX).forEach(([key, fx]) => {
  const b = document.createElement('button');
  b.textContent = fx.name;
  if (key === 'none') b.classList.add('active');
  b.onclick = () => {
    currentFx = key;
    fxBox.querySelectorAll('button').forEach(x => x.classList.remove('active'));
    b.classList.add('active');
    draw();
  };
  fxBox.appendChild(b);
});

// ===== رفع الصورة =====
$('uploadArea').onclick = () => $('fileInput').click();
$('uploadArea').ondragover = e => e.preventDefault();
$('uploadArea').ondrop = e => { e.preventDefault(); loadImage(e.dataTransfer.files[0]); };
$('fileInput').onchange = e => loadImage(e.target.files[0]);

function loadImage(file) {
  if (!file) return;
  const img = new Image();
  img.onload = () => {
    subjectImg = img;
    brushMask.width = img.width; brushMask.height = img.height;
    bctx.fillStyle = '#fff'; bctx.fillRect(0, 0, brushMask.width, brushMask.height);
    $('uploadArea').style.display = 'none';
    $('editor').classList.add('show');
    draw();
  };
  img.src = URL.createObjectURL(file);
}

// ===== AI إزالة الخلفية (BiRefNet على جهازك) =====
$('removeBgBtn').onclick = async (e) => {
  e.target.disabled = true;
  $('status').textContent = '⏳ المحرك بيعالج (أول مرة بيحمّل الموديل)...';
  try {
    const fd = new FormData();
    fd.append('file', $('fileInput').files[0] || await (await fetch(subjectImg.src)).blob());
    const res = await fetch('/remove-bg', { method: 'POST', body: fd });
    if (!res.ok) throw 0;
    const blob = await res.blob();
    const img = new Image();
    img.onload = () => {
      subjectImg = img;
      bctx.fillStyle = '#fff'; bctx.fillRect(0, 0, brushMask.width, brushMask.height);
      $('status').textContent = '✅ جاهز — دقّق بالفرشاة لو محتاج';
      draw();
    };
    img.src = URL.createObjectURL(blob);
  } catch { $('status').textContent = '❌ حصل خطأ — تأكد إن السيرفر شغال'; }
  e.target.disabled = false;
};

// ===== الفرشاة اليدوية =====
[['brushOff','off'],['brushErase','erase'],['brushRestore','restore']].forEach(([id, m]) => {
  $(id).onclick = () => {
    brushMode = m;
    ['brushOff','brushErase','brushRestore'].forEach(x => $(x).classList.remove('active'));
    $(id).classList.add('active');
    canvas.style.cursor = m === 'off' ? 'default' : 'crosshair';
  };
});
$('brushSize').oninput = () => $('vBrush').textContent = $('brushSize').value;

let painting = false;
function brushPos(e) {
  const r = canvas.getBoundingClientRect();
  return { x: (e.clientX - r.left) * (canvas.width / r.width),
           y: (e.clientY - r.top) * (canvas.height / r.height) };
}
function paint(e) {
  if (!painting || brushMode === 'off' || !subjectImg) return;
  const p = brushPos(e), size = +$('brushSize').value;
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
canvas.onpointerup = canvas.onpointerleave = () => painting = false;

// ===== الخلفية =====
let bgMode = 'color';
const modes = { color: 'optColor', gradient: 'optGradient', image: 'optImage' };
Object.entries(modes).forEach(([m, id]) => {
  $(id).onclick = () => {
    bgMode = m;
    Object.values(modes).forEach(x => $(x).classList.remove('active'));
    $(id).classList.add('active');
    $('colorSection').style.display = m === 'color' ? 'block' : 'none';
    $('gradientSection').style.display = m === 'gradient' ? 'block' : 'none';
    $('imageSection').style.display = m === 'image' ? 'block' : 'none';
    draw();
  };
});
$('bgImageInput').onchange = e => {
  const f = e.target.files[0]; if (!f) return;
  bgImg = new Image();
  bgImg.onload = draw;
  bgImg.src = URL.createObjectURL(f);
};

// ===== الرسم =====
function baseFilter() {
  const b = $('brightness').value, c = $('contrast').value,
        s = $('saturation').value, w = $('warmth').value;
  let f = `brightness(${b}%) contrast(${c}%) saturate(${s}%)`;
  if (w > 0)      f += ` sepia(${w * 0.6}%) saturate(${100 + +w}%) hue-rotate(-${w * 0.3}deg)`;
  else if (w < 0) f += ` saturate(${100 + +w}%) hue-rotate(${-w * 0.4}deg)`;
  return f;
}

function drawSubject(targetCtx, w, h, filter) {
  const tmp = document.createElement('canvas');
  tmp.width = w; tmp.height = h;
  const t = tmp.getContext('2d');
  t.drawImage(subjectImg, 0, 0, w, h);
  t.globalCompositeOperation = 'destination-in';
  t.drawImage(brushMask, 0, 0, w, h);
  targetCtx.filter = filter;
  targetCtx.drawImage(tmp, 0, 0);
  targetCtx.filter = 'none';
}

function draw() {
  if (!subjectImg) return;
  let W = subjectImg.width, H = subjectImg.height;
  const preset = $('sizePreset').value;
  if (preset === '4x6')    { if (W / H > 1.5) W = H * 1.5; else H = W / 1.5; }
  if (preset === 'square') H = W;
  if (preset === 'story')  H = W * 16 / 9;
  canvas.width = W; canvas.height = H;

  const fx = FX[currentFx];
  const filter = baseFilter() + (fx.filter ? ' ' + fx.filter : '');

  // 1) الخلفية
  if (bgMode === 'image' && bgImg) {
    const s = Math.max(W / bgImg.width, H / bgImg.height);
    ctx.drawImage(bgImg, (W - bgImg.width*s)/2, (H - bgImg.height*s)/2, bgImg.width*s, bgImg.height*s);
  } else if (bgMode === 'gradient') {
    const g = ctx.createLinearGradient(0, 0, W, H);
    g.addColorStop(0, $('bgColor1').value); g.addColorStop(1, $('bgColor2').value);
    ctx.fillStyle = g; ctx.fillRect(0, 0, W, H);
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

  // 3) الشخص بالمؤثر
  drawSubject(ctx, W, H, filter);

  // 4) تسريب ضوئي (Light Leak) — روح المؤثرات السينمائية
  if (fx.leak) {
    ctx.save();
    ctx.globalCompositeOperation = 'screen';
    const g = ctx.createRadialGradient(W * 0.85, H * 0.1, 0, W * 0.85, H * 0.1, Math.max(W, H) * 0.9);
    g.addColorStop(0, `rgba(${fx.leak.c},${fx.leak.a})`);
    g.addColorStop(1, `rgba(${fx.leak.c},0)`);
    ctx.fillStyle = g;
    ctx.fillRect(0, 0, W, H);
    ctx.restore();
  }

  // 5) تظليل الحواف
  const v = +$('vignette').value;
  if (v > 0) {
    const g = ctx.createRadialGradient(W/2, H/2, Math.min(W,H)*0.4, W/2, H/2, Math.max(W,H)*0.75);
    g.addColorStop(0, 'rgba(0,0,0,0)');
    g.addColorStop(1, `rgba(0,0,0,${v/130})`);
    ctx.fillStyle = g; ctx.fillRect(0, 0, W, H);
  }
}

// ===== السلايدرات =====
[['brightness','%'],['contrast','%'],['saturation','%'],['warmth',''],['shadow','%'],['vignette','%']].forEach(([id, suf]) => {
  $(id).oninput = () => {
    $('v' + id[0].toUpperCase() + id.slice(1)).textContent = $(id).value + suf;
    draw();
  };
});
['bgColor','bgColor1','bgColor2'].forEach(id => $(id).oninput = draw);
$('sizePreset').onchange = draw;

// ===== التصدير =====
function exportImage(type) {
  const a = document.createElement('a');
  a.download = 'studio-' + Date.now() + (type === 'png' ? '.png' : '.jpg');
  a.href = type === 'png' ? canvas.toDataURL('image/png') : canvas.toDataURL('image/jpeg', 0.95);
  a.click();
}
$('downloadBtn').onclick = () => exportImage('png');
$('downloadJpgBtn').onclick = () => exportImage('jpg');
$('resetBtn').onclick = () => location.reload();
</script>
</body>
</html>
