<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>استوديو الصور الاحترافي</title>
<style>
  * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, sans-serif; }
  body { background: #14141f; color: #fff; margin: 0; min-height: 100vh; }
  header { padding: 18px; text-align: center; background: #1e1e2e; border-bottom: 1px solid #333; }
  header h1 { margin: 0; font-size: 22px; }
  .container { max-width: 1200px; margin: 20px auto; padding: 0 15px; }
  .upload-area {
    border: 2px dashed #7c6cff; border-radius: 16px; padding: 80px 20px;
    text-align: center; cursor: pointer; transition: 0.3s;
  }
  .upload-area:hover { background: #1e1e35; }
  .upload-area p { font-size: 18px; margin: 5px; }
  .upload-area small { color: #888; }
  .editor { display: none; gap: 20px; }
  .editor.show { display: flex; flex-wrap: wrap; }
  .canvas-box { flex: 1; min-width: 300px; min-height: 400px; background: repeating-conic-gradient(#2a2a2a 0 25%, #383838 0 50%) 0 0/24px 24px;
    border-radius: 12px; display: flex; align-items: center; justify-content: center; padding: 15px; }
  canvas { max-width: 100%; max-height: 75vh; border-radius: 4px; }
  .controls { width: 320px; background: #1e1e2e; border-radius: 12px; padding: 20px; max-height: 80vh; overflow-y: auto; }
  .controls h3 { margin: 18px 0 8px; font-size: 14px; color: #7c6cff; border-bottom: 1px solid #333; padding-bottom: 6px; }
  .controls h3:first-child { margin-top: 0; }
  label { display: flex; justify-content: space-between; font-size: 13px; margin: 12px 0 3px; color: #ccc; }
  label span { color: #7c6cff; font-weight: bold; }
  input[type="range"] { width: 100%; accent-color: #7c6cff; }
  input[type="color"] { width: 100%; height: 40px; border: none; border-radius: 8px; background: none; cursor: pointer; padding: 0; }
  button {
    width: 100%; padding: 12px; border-radius: 8px; border: none;
    font-size: 15px; cursor: pointer; margin-top: 8px; font-weight: bold;
  }
  .primary { background: #7c6cff; color: #fff; }
  .secondary { background: #333; color: #fff; }
  button:disabled { background: #444; color: #888; cursor: wait; }
  #status { margin: 10px 0; font-size: 13px; color: #aaa; text-align: center; min-height: 18px; }
  .bg-options { display: flex; gap: 8px; }
  .bg-options button { flex: 1; padding: 8px; font-size: 13px; margin-top: 0; }
  .bg-options button.active { outline: 2px solid #7c6cff; }
</style>
</head>
<body>
<header><h1>📸 استوديو الصور الاحترافي</h1></header>
<div class="container">
  <div class="upload-area" id="uploadArea">
    <p>📤 اضغط أو اسحب صورة العميل هنا</p>
    <small>الصور بتتعالج على جهازك مباشرة — مفيش حاجة بتتبعت لأي مكان</small>
    <input type="file" id="fileInput" accept="image/*" hidden>
  </div>

  <div class="editor" id="editor">
    <div class="canvas-box"><canvas id="canvas"></canvas></div>
    <div class="controls">

      <h3>الخلفية</h3>
      <button class="primary" id="removeBgBtn">✨ إزالة الخلفية</button>
      <div id="status"></div>
      <div class="bg-options" style="margin-top:8px">
        <button class="secondary active" id="optColor">لون</button>
        <button class="secondary" id="optGradient">تدرج</button>
        <button class="secondary" id="optImage">صورة</button>
      </div>
      <div id="colorSection">
        <input type="color" id="bgColor" value="#ffffff">
      </div>
      <div id="gradientSection" style="display:none">
        <input type="color" id="bgColor1" value="#667eea" style="margin-bottom:6px">
        <input type="color" id="bgColor2" value="#764ba2">
      </div>
      <div id="imageSection" style="display:none">
        <input type="file" id="bgImageInput" accept="image/*">
      </div>

      <h3>الإضاءة والألوان</h3>
      <label>السطوع <span id="vBrightness">100%</span></label>
      <input type="range" id="brightness" min="30" max="200" value="100">
      <label>التباين <span id="vContrast">100%</span></label>
      <input type="range" id="contrast" min="30" max="200" value="100">
      <label>تشبع الألوان <span id="vSaturation">100%</span></label>
      <input type="range" id="saturation" min="0" max="250" value="100">
      <label>حرارة اللون <span id="vWarmth">0</span></label>
      <input type="range" id="warmth" min="-100" max="100" value="0">
      <label>الحدة <span id="vSharpness">0</span></label>
      <input type="range" id="sharpness" min="0" max="100" value="0">

      <h3>تصدير</h3>
      <button class="primary" id="downloadBtn">💾 تحميل PNG</button>
      <button class="secondary" id="resetBtn">🔄 صورة جديدة</button>
    </div>
  </div>
</div>

<script type="module">
import { removeBackground } from 'https://cdn.jsdelivr.net/npm/@imgly/background-removal@1.5.5/+esm';

const $ = id => document.getElementById(id);
const canvas = $('canvas'), ctx = canvas.getContext('2d');
let subjectImg = null, bgImg = null;

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
    $('uploadArea').style.display = 'none';
    $('editor').classList.add('show');
    draw();
  };
  img.src = URL.createObjectURL(file);
}

// ====== إزالة الخلفية ======
$('removeBgBtn').onclick = async (e) => {
  e.target.disabled = true;
  $('status').textContent = '⏳ جاري فصل الشخص عن الخلفية...';
  try {
    const blob = await removeBackground(subjectImg.src);
    const img = new Image();
    img.onload = () => {
      subjectImg = img;
      $('status').textContent = '✅ خلاص، اختار الخلفية الجديدة';
      draw();
    };
    img.src = URL.createObjectURL(blob);
  } catch { $('status').textContent = '❌ حصل خطأ — جرب صورة أوضح'; }
  e.target.disabled = false;
};

// ====== خيارات الخلفية ======
let bgMode = 'color';
$('optColor').onclick = () => setBgMode('color');
$('optGradient').onclick = () => setBgMode('gradient');
$('optImage').onclick = () => setBgMode('image');
function setBgMode(m) {
  bgMode = m;
  ['optColor','optGradient','optImage'].forEach(id => $(id).classList.remove('active'));
  $('opt' + m[0].toUpperCase() + m.slice(1)).classList.add('active');
  $('colorSection').style.display = m === 'color' ? 'block' : 'none';
  $('gradientSection').style.display = m === 'gradient' ? 'block' : 'none';
  $('imageSection').style.display = m === 'image' ? 'block' : 'none';
  draw();
}
$('bgImageInput').onchange = e => {
  const f = e.target.files[0]; if (!f) return;
  bgImg = new Image();
  bgImg.onload = draw;
  bgImg.src = URL.createObjectURL(f);
};

// ====== الرسم ======
function draw() {
  if (!subjectImg) return;
  // نحافظ على أبعاد الصورة الأصلية (جودة كاملة)
  canvas.width = subjectImg.width;
  canvas.height = subjectImg.height;

  // 1) الخلفية
  if (bgMode === 'image' && bgImg) {
    // تغطية كاملة (cover) مع الحفاظ على النسب
    const s = Math.max(canvas.width / bgImg.width, canvas.height / bgImg.height);
    const w = bgImg.width * s, h = bgImg.height * s;
    ctx.drawImage(bgImg, (canvas.width - w)/2, (canvas.height - h)/2, w, h);
  } else if (bgMode === 'gradient') {
    const g = ctx.createLinearGradient(0, 0, canvas.width, canvas.height);
    g.addColorStop(0, $('bgColor1').value);
    g.addColorStop(1, $('bgColor2').value);
    ctx.fillStyle = g;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  } else {
    ctx.fillStyle = $('bgColor').value;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  }

  // 2) الشخص مع الفلاتر
  const b = $('brightness').value, c = $('contrast').value,
        s = $('saturation').value, w = $('warmth').value;

  let filter = `brightness(${b}%) contrast(${c}%) saturate(${s}%)`;
  // الحرارة: أزرق للبرد، برتقالي للدفء
  if (w > 0)      filter += ` sepia(${w * 0.6}%) saturate(${100 + +w}%) hue-rotate(-${w * 0.3}deg)`;
  else if (w < 0) filter += ` saturate(${100 + +w}%) hue-rotate(${-w * 0.4}deg)`;

  ctx.filter = filter;
  ctx.drawImage(subjectImg, 0, 0);
  ctx.filter = 'none';

  // 3) الحدة (Sharpen) — بتعمل بطبقة تانية
  const sh = +$('sharpness').value;
  if (sh > 0) {
    ctx.globalAlpha = sh / 250;
    ctx.filter = 'contrast(120%) saturate(110%)';
    ctx.drawImage(canvas, 0, 0);
    ctx.filter = 'none';
    ctx.globalAlpha = 1;
  }
}

// ====== السلايدرات ======
const sliders = ['brightness','contrast','saturation','warmth','sharpness'];
sliders.forEach(id => {
  $(id).oninput = () => {
    const label = 'v' + id[0].toUpperCase() + id.slice(1);
    const suffix = (id === 'warmth' || id === 'sharpness') ? '' : '%';
    $(label).textContent = $(id).value + suffix;
    draw();
  };
});
['bgColor','bgColor1','bgColor2'].forEach(id => $(id).oninput = draw);

// ====== التحميل ======
$('downloadBtn').onclick = () => {
  const a = document.createElement('a');
  a.download = 'studio-' + Date.now() + '.png';
  a.href = canvas.toDataURL('image/png');
  a.click();
};
$('resetBtn').onclick = () => location.reload();
</script>
</body>
</html>
