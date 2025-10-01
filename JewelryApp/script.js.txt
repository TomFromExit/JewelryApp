const imgInput = document.getElementById('imageInput');
const dropZone = document.getElementById('drop-zone');
const preview = document.getElementById('preview');
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

let uploadedImg = null;

// Drag & drop
dropZone.addEventListener('dragover', (e) => {
  e.preventDefault();
  dropZone.classList.add('hover');
});
dropZone.addEventListener('dragleave', () => {
  dropZone.classList.remove('hover');
});
dropZone.addEventListener('drop', (e) => {
  e.preventDefault();
  dropZone.classList.remove('hover');
  const file = e.dataTransfer.files[0];
  handleFile(file);
});

// File input
imgInput.addEventListener('change', (e) => {
  const file = e.target.files[0];
  handleFile(file);
});

function handleFile(file) {
  if (!file) return;
  const reader = new FileReader();
  reader.onload = function(event) {
    uploadedImg = new Image();
    uploadedImg.onload = function() {
      canvas.width = uploadedImg.width;
      canvas.height = uploadedImg.height;
      ctx.drawImage(uploadedImg, 0, 0);
      preview.src = uploadedImg.src;
      calculateWeight();
    };
    uploadedImg.src = event.target.result;
  };
  reader.readAsDataURL(file);
}

// Toggle rýdzosť zlata
function toggleKaratOptions() {
  const material = document.getElementById("material").value;
  document.getElementById("karat-container").style.display = (material === "gold") ? "block" : "none";
}

// Dynamický prepočet pri zmene vstupov
document.getElementById("height").addEventListener('input', calculateWeight);
document.getElementById("width").addEventListener('input', calculateWeight);
document.getElementById("thickness").addEventListener('input', calculateWeight);
document.getElementById("material").addEventListener('change', calculateWeight);
document.getElementById("karat").addEventListener('change', calculateWeight);

function calculateWeight() {
  const height = parseFloat(document.getElementById("height").value);
  const width = parseFloat(document.getElementById("width").value);
  const thickness = parseFloat(document.getElementById("thickness").value);
  const material = document.getElementById("material").value;

  if (!uploadedImg) return;
  if (!height || !width || !thickness) return;

  const imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  let opaquePixels = 0;
  for (let i = 0; i < imgData.data.length; i += 4) {
    if (imgData.data[i+3] > 0) opaquePixels++;
  }
  const totalPixels = canvas.width * canvas.height;
  const areaRatio = opaquePixels / totalPixels;

  const densities = { gold: 19.32, silver: 10.49 };
  const karatPercentages = { 9: 0.375, 14: 0.585, 18: 0.75, 24: 1.0 };

  const volume = height * width * thickness * areaRatio;
  const volumeCm3 = volume / 1000;

  let weight = 0;
  let priceInEur = 0;

  if (material === "gold") {
    const karat = parseInt(document.getElementById("karat").value);
    weight = volumeCm3 * densities.gold * karatPercentages[karat];
    const basePrice14k = 64.1; // cena za 1g 14k zlata v €
    priceInEur = (weight * basePrice14k * (karat / 14)).toFixed(2);
  } else {
    weight = volumeCm3 * densities.silver;
    const basePriceSilver = 1.53; // cena za 1g striebra v €
    priceInEur = (weight * basePriceSilver).toFixed(2);
  }

  document.getElementById("result").innerText =
    "Váha: " + weight.toFixed(2) + " g (" + (material === "gold" ? document.getElementById("karat").value + "k zlato" : "striebro") + ")";
  document.getElementById("gold-price").innerText = (material === "gold") ? "Cena: " + priceInEur + " €" : "";
  document.getElementById("silver-price").innerText = (material === "silver") ? "Cena: " + priceInEur + " €" : "";
}

toggleKaratOptions();
