# PDF→PNG_コード完成版

## プロジェクト

PDF→PNG

## 情報の種類

コード完成版

## 検索キーワード

PDF, PNG、pdf-to-png、2026.8.22

## 保存場所

GitHub

## 内容

<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>PDF→PNG（A4固定・個別＋ZIP）</title>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>

  <style>
    body {
      font-family: sans-serif;
      padding: 20px;
      background: #f4f6f8;
    }

    #drop-area {
      border: 2px dashed #888;
      border-radius: 8px;
      padding: 20px;
      text-align: center;
      background: #ffffff;
      margin-bottom: 20px;
      cursor: pointer;
    }

    #drop-area.dragover {
      background: #e8f0ff;
      border-color: #1a73e8;
    }

    .zip-area {
      margin: 10px 0 20px;
      padding: 10px;
      background: #e6fff2;
      border: 1px solid #0b8043;
      border-radius: 6px;
    }

    .zip-btn {
      padding: 8px 12px;
      background: #0b8043;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }

    .page-container {
      margin-bottom: 20px;
      padding: 10px;
      background: #ffffff;
      border: 1px solid #ddd;
      border-radius: 6px;
    }

    .download-btn {
      margin-top: 6px;
      padding: 6px 10px;
      background: #1a73e8;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }

    img {
      max-width: 100%;
      border: 1px solid #ccc;
      display: block;
    }

    .title{
      display:flex;
      align-items:center;
      gap:10px;
    }

    .logo{
      width:40px;
      height:40px;
    }
  </style>
</head>

<body>

<h1 class="title">
  <img src="logo.png" class="logo">
  PDF → PNG 変換ツール（A4固定）
</h1>

<h2>① PDFの読み込み（ドラッグ or クリック）</h2>

<div id="drop-area">
  ここにPDFをドラッグ＆ドロップ<br>
  またはクリックして選択
</div>

<input type="file" id="pdf-upload" accept="application/pdf" style="display:none;" />

<!-- ===== ZIPエリア（変換後に表示）===== -->
<div id="zipArea" class="zip-area" style="display:none;">
  <h2>③-2 一括ダウンロード（ZIP・複数枚）←推奨</h2>
  <p>すべてのページをまとめて1回で保存できます。</p>
  <button id="zipBtn" class="zip-btn">
    全ページをZIPでダウンロード
  </button>
</div>

<h2>③-1 個別ダウンロード（1枚のみ）</h2>
<p>必要なページだけ保存したい場合はこちら。</p>

<div id="output"></div>

<script>
const pdfjsLib = window['pdfjs-dist/build/pdf'];
pdfjsLib.GlobalWorkerOptions.workerSrc =
 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.worker.min.js';

const dropArea = document.getElementById('drop-area');
const fileInput = document.getElementById('pdf-upload');
const output = document.getElementById('output');
const zipArea = document.getElementById('zipArea');
const zipBtn = document.getElementById('zipBtn');

// ===== A4サイズ（300dpi相当）=====
const A4_WIDTH = 2480;
const A4_HEIGHT = 3508;

/* ===== Drag & Drop ===== */

dropArea.addEventListener('click', () => fileInput.click());

['dragenter','dragover'].forEach(evt => {
  dropArea.addEventListener(evt, e => {
    e.preventDefault();
    dropArea.classList.add('dragover');
  });
});

['dragleave','drop'].forEach(evt => {
  dropArea.addEventListener(evt, e => {
    e.preventDefault();
    dropArea.classList.remove('dragover');
  });
});

dropArea.addEventListener('drop', e => {
  const file = e.dataTransfer.files[0];
  if (file && file.type === "application/pdf") {
    handlePDF(file);
  } else {
    alert("PDFファイルをドロップしてください。");
  }
});

fileInput.addEventListener('change', e => {
  const file = e.target.files[0];
  if (file) handlePDF(file);
});

/* ===== PDF処理 ===== */

async function handlePDF(file) {

  const baseName = file.name.replace(/\.pdf$/i, '');

  output.innerHTML = "<p>変換中…少しお待ちください。</p>";
  zipArea.style.display = "none";

  const arrayBuffer = await file.arrayBuffer();
  const pdf = await pdfjsLib.getDocument({ data: arrayBuffer }).promise;

  output.innerHTML = "";

  const pngList = []; // ZIP用に保存

  for (let pageNum = 1; pageNum <= pdf.numPages; pageNum++) {

    const page = await pdf.getPage(pageNum);
    const viewport = page.getViewport({ scale: 2 });

    // まず高品質レンダリング
    const tmpCanvas = document.createElement('canvas');
    const tctx = tmpCanvas.getContext('2d');
    tmpCanvas.width = viewport.width;
    tmpCanvas.height = viewport.height;

    tctx.fillStyle = "#ffffff";
    tctx.fillRect(0, 0, tmpCanvas.width, tmpCanvas.height);

    await page.render({ canvasContext: tctx, viewport }).promise;

    // ===== A4キャンバスにリサイズ =====
    const a4Canvas = document.createElement('canvas');
    const actx = a4Canvas.getContext('2d');
    a4Canvas.width = A4_WIDTH;
    a4Canvas.height = A4_HEIGHT;

    actx.fillStyle = "#ffffff";
    actx.fillRect(0, 0, A4_WIDTH, A4_HEIGHT);

    const scale = Math.min(
      A4_WIDTH / tmpCanvas.width,
      A4_HEIGHT / tmpCanvas.height
    );

    const newW = tmpCanvas.width * scale;
    const newH = tmpCanvas.height * scale;
    const x = (A4_WIDTH - newW) / 2;
    const y = (A4_HEIGHT - newH) / 2;

    actx.drawImage(tmpCanvas, x, y, newW, newH);

    const pngDataUrl = a4Canvas.toDataURL("image/png");
    const baseName = file.name.replace(/\.pdf$/i, '');

    pngList.push({
      name: `${baseName}_${pageNum}.png`,
      data: pngDataUrl
    });

    // ===== 個別ダウンロード表示 =====
    const container = document.createElement("div");
    container.className = "page-container";

    const title = document.createElement("p");
    title.textContent = `ページ ${pageNum}（A4固定）`;

    const img = document.createElement("img");
    img.src = pngDataUrl;

    const btn = document.createElement("button");
    btn.textContent = "このページをダウンロード";
    btn.className = "download-btn";

    btn.addEventListener("click", () => {
      const a = document.createElement("a");
      a.href = pngDataUrl;
      const baseName = file.name.replace(/\.pdf$/i, '');

      a.download = `${baseName}_${pageNum}.png`;
      a.click();
    });

    container.appendChild(title);
    container.appendChild(img);
    container.appendChild(btn);
    output.appendChild(container);
  }

  // ===== ZIPエリアを表示 =====
  zipArea.style.display = "block";

  zipBtn.onclick = async () => {
    const zip = new JSZip();

    for (const item of pngList) {
      const base64 = item.data.split(',')[1];
      zip.file(item.name, base64, { base64: true });
    }

    const blob = await zip.generateAsync({ type: "blob" });
    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = `${baseName}.zip`;
    a.click();
  };
}
</script>

</body>
</html>
