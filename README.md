<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>محرر الصور - إزالة الخلفية</title>
<style>
  * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, sans-serif; }
  body { background: #1e1e2e; color: #fff; margin: 0; min-height: 100vh; }
  header { padding: 20px; text-align: center; background: #2a2a40; }
  header h1 { margin: 0; font-size: 24px; }
  .container { max-width: 1100px; margin: 20px auto; padding: 0 15px; }
  .upload-area {
    border: 2px dashed #6c63ff; border-radius: 16px; padding: 60px 20px;
    text-align: center; cursor: pointer; transition: 0.3s;
  }
  .upload-area:hover { background: #2a2a50; }
  .editor { display: none; gap: 20px; margin-top: 20px; }
  .editor.show { display: flex; flex-wrap: wrap; }
  .canvas-box { flex: 1; min-width: 300px; background: repeating-conic-gradient(#333 0 25%, #444 0 50%) 0 0/20px 20px;
    border-radius: 12px; display: flex; align-items: center; justify-content: center; padding: 15px; }
  canvas { max-width: 100%; border-radius: 8px; }
  .controls { width: 300px; background: #2a2a40; border-radius: 12px; padding: 20px; }
  .controls label { display: block; margin: 15px 0 5px; font-size: 14px; }
  button, input[type="color"], select {
    width: 100%; padding: 10px; border-radius: 8px; border: none;
    font-size: 15px; cursor: pointer; margin-top: 8px;
  }
  button { background: #6c63ff; color: #fff; font-weight: bold; }
  button:disabled { background: #555; cursor: wait; }
  .bg-btn { background: #444; }
  input[type="range"] { width: 100%; }
  #status { margin-top: 10px; font-size: 13px; color: #aaa; text-align: center; }
</style>
</head>
<body>
<header><h1>🖼️ محرر الصور مع إزالة الخلفية</h1></header>
<div class="container">
  <div class="upload-area" id="uploadArea">
    <p>📤 اضغط هنا أو اسحب الصورة</p>
    <input type="file" id="fileInput" accept="image/*" hidden>
  </div>

  <div class="editor" id="editor">
    <div class="canvas-box"><canvas id="canvas"></canvas></div>
    <div class="controls">
      <button id="removeBgBtn">✨ إزالة الخلفية</button>
      <div id="status"></div>

      <label>لون الخلفية الجديدة</label>
      <input type="color" id="bgColor" value="#ffffff">

      <label>تدرج لوني (اختياري)</label>
      <input type="color" id="bgColor2" value="#ffffff">

      <label>خلفية من صورة</label>
      <input type="file" id="bgImageInput" accept="image/*">

      <label>تدوير</label>
      <input type="range" id="rotate" min="0" max="360" value="0">

      <button class="bg-btn" id="downloadBtn">💾 تحميل النتيجة PNG</button>
      <button class="bg-btn" id="resetBtn">🔄 صورة جديدة</button>
    </div>
  </div>
</div>

<script type="module">
import { removeBackground } from 'https://cdn.jsdelivr.net/npm/@imgly/background-removal@1.5.5/+esm';

const uploadArea = document.getElementById('uploadArea');
const fileInput  = document.getElementById('fileInput');
const editor     = document.getElementById('editor');
const canvas     = document.getElementById('canvas');
const ctx        = canvas.getContext('2d');
const statusEl   = document.getElementById('status');

let subjectImg = null; // الصورة بعد إزالة الخلفية
let bgImg = null;

uploadArea.onclick = () => fileInput.click();
uploadArea.ondragover = e => e.preventDefault();
uploadArea.ondrop = e => { e.preventDefault(); loadImage(e.dataTransfer.files[0]); };
fileInput.onchange = e => loadImage(e.target.files[0]);

function loadImage(file) {
  if (!file) return;
  const url = URL.createObjectURL(file);
  const img = new Image();
  img.onload = () => {
    subjectImg = img;
    uploadArea.style.display = 'none';
    editor.classList.add('show');
    draw();
  };
  img.src = url;
}

document.getElementById('removeBgBtn').onclick = async (e) => {
  e.target.disabled = true;
  statusEl.textContent = '⏳ جاري إزالة الخلفية... (أول مرة بياخد شوية وقت)';
  try {
    const blob = await removeBackground(subjectImg.src);
    const img = new Image();
    img.onload = () => { subjectImg = img; statusEl.textContent = '✅ تمت إزالة الخلفية'; draw(); };
    img.src = URL.createObjectURL(blob);
  } catch (err) {
    statusEl.textContent = '❌ حصل خطأ، حاول تاني';
  }
  e.target.disabled = false;
};

function draw() {
  if (!subjectImg) return;
  const rotation = +document.getElementById('rotate').value;
  const maxW = 800;
  const scale = Math.min(1, maxW / subjectImg.width);
  canvas.width = subjectImg.width * scale;
  canvas.height = subjectImg.height * scale;

  // 1) ارسم الخلفية المطلوبة
  const c1 = document.getElementById('bgColor').value;
  const c2 = document.getElementById('bgColor2').value;
  if (bgImg) {
    ctx.drawImage(bgImg, 0, 0, canvas.width, canvas.height);
  } else if (c1 !== c2) {
    const grad = ctx.createLinearGradient(0, 0, canvas.width, canvas.height);
    grad.addColorStop(0, c1); grad.addColorStop(1, c2);
    ctx.fillStyle = grad;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  } else {
    ctx.fillStyle = c1;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  }

  // 2) ارسم الصورة فوقها
  ctx.save();
  if (rotation) {
    ctx.translate(canvas.width/2, canvas.height/2);
    ctx.rotate(rotation * Math.PI / 180);
    ctx.drawImage(subjectImg, -canvas.width/2, -canvas.height/2, canvas.width, canvas.height);
  } else {
    ctx.drawImage(subjectImg, 0, 0, canvas.width, canvas.height);
  }
  ctx.restore();
}

['bgColor','bgColor2','rotate'].forEach(id =>
  document.getElementById(id).oninput = draw);

document.getElementById('bgImageInput').onchange = e => {
  const f = e.target.files[0]; if (!f) return;
  bgImg = new Image();
  bgImg.onload = draw;
  bgImg.src = URL.createObjectURL(f);
};

document.getElementById('downloadBtn').onclick = () => {
  const a = document.createElement('a');
  a.download = 'edited-image.png';
  a.href = canvas.toDataURL('image/png');
  a.click();
};

document.getElementById('resetBtn').onclick = () => location.reload();
</script>
</body>
</html>
