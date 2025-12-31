<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>OCR مطور - عربي + إنجليزي</title>
<style>
  body { font-family: Arial; text-align: center; margin-top: 30px; }
  input, button { padding: 10px; font-size: 16px; margin: 5px; }
  textarea { width: 80%; margin-top: 10px; height: 150px; }
  img { max-width: 200px; margin: 10px; border: 1px solid #ccc; }
  #progress { width: 80%; height: 20px; margin: 10px auto; background: #eee; border-radius: 10px; overflow: hidden; }
  #bar { width: 0%; height: 100%; background: #4caf50; }
</style>
</head>
<body>

<h2>تحويل الصور إلى نص</h2>

<input type="file" id="imageInput" accept="image/*"><br>
<img id="preview" src="" alt="معاينة الصورة"><br>

<button id="start">تحليل الصورة</button>
<div id="progress"><div id="bar"></div></div>
<p id="status"></p>

<textarea id="result" placeholder="النص سيظهر هنا..."></textarea><br>

<button id="copyBtn">نسخ النص</button>
<button id="downloadBtn">تحميل TXT</button>
<button id="downloadPdfBtn">تحميل PDF</button>

<script src="https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<script>
const imageInput = document.getElementById("imageInput");
const startBtn = document.getElementById("start");
const resultBox = document.getElementById("result");
const statusText = document.getElementById("status");
const preview = document.getElementById("preview");
const bar = document.getElementById("bar");
const copyBtn = document.getElementById("copyBtn");
const downloadBtn = document.getElementById("downloadBtn");
const downloadPdfBtn = document.getElementById("downloadPdfBtn");

// معاينة الصورة
imageInput.onchange = function() {
  if (imageInput.files && imageInput.files[0]) {
    preview.src = URL.createObjectURL(imageInput.files[0]);
    resultBox.value = "";
    statusText.textContent = "";
    bar.style.width = "0%";
  }
};

// تحليل الصورة
startBtn.onclick = function() {
  if (!imageInput.files.length) return;
  statusText.textContent = "جاري تحليل الصورة... ⏳";
  resultBox.value = "";
  bar.style.width = "0%";

  Tesseract.recognize(imageInput.files[0], "ara+eng", {
    logger: info => {
      if (info.status === "recognizing text") {
        bar.style.width = Math.floor(info.progress * 100) + "%";
        statusText.textContent = "التقدم: " + Math.floor(info.progress * 100) + "%";
      }
    }
  }).then(result => {
    resultBox.value = result.data.text;
    statusText.textContent = "تم استخراج النص ✅";
    bar.style.width = "100%";
  }).catch(err => {
    statusText.textContent = "حدث خطأ ❌";
    console.error(err);
  });
};

// نسخ النص
copyBtn.onclick = function() {
  resultBox.select();
  document.execCommand("copy");
  alert("تم نسخ النص!");
};

// تحميل TXT
downloadBtn.onclick = function() {
  const blob = new Blob([resultBox.value], {type: "text/plain"});
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "extracted_text.txt";
  link.click();
};

// تحميل PDF
downloadPdfBtn.onclick = function() {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  const lines = resultBox.value.split("\n");
  let y = 10;
  for (let line of lines) {
    doc.text(line, 10, y);
    y += 10;
    if (y > 280) {
      doc.addPage();
      y = 10;
    }
  }
  doc.save("extracted_text.pdf");
};
</script>

</body>
</html>
