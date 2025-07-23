# HomeStore
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>حاسبة الأسعار</title>
  <style>
    body {
      font-family: 'Arial', sans-serif;
      background-color: #f4f4f4;
      color: #333;
      direction: rtl;
      padding: 20px;
    }

    header {
      text-align: center;
      margin-bottom: 20px;
    }

    header img {
      max-height: 80px;
    }

    h1 {
      margin-top: 10px;
      color: #222;
    }

    .container {
      background: #fff;
      border-radius: 8px;
      padding: 20px;
      max-width: 900px;
      margin: auto;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }

    select, input {
      padding: 10px;
      width: 100%;
      margin: 5px 0 15px;
      border-radius: 5px;
      border: 1px solid #ccc;
    }

    label {
      font-weight: bold;
      margin-top: 10px;
      display: block;
    }

    button {
      background-color: #444;
      color: white;
      padding: 10px 15px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      margin-top: 10px;
    }

    button:hover {
      background-color: #333;
    }

    .result-box {
      background-color: #eee;
      padding: 15px;
      border-radius: 6px;
      margin-top: 20px;
      white-space: pre-line;
    }

    .results {
      margin-top: 30px;
    }

    .currency-toggle {
      margin-top: 10px;
      display: flex;
      gap: 10px;
      align-items: center;
    }

    .currency-toggle input {
      width: auto;
    }

  </style>
</head>
<body>

<header>
  <img src="your-logo.png" alt="شعار الشركة" />
  <h1>حاسبة الأسعار</h1>
</header>

<div class="container">
  <label for="category">نوع المنتج</label>
  <select id="category">
    <option value="">اختر الفئة</option>
    <option value="نوافذ">نوافذ</option>
    <option value="أبواب">أبواب</option>
    <option value="أبواب سحب">أبواب السحب</option>
  </select>

  <label for="type">النوع</label>
  <select id="type"></select>

  <label for="movement">التصنيف (ثابت / حركة...)</label>
  <select id="movement"></select>

  <label for="length">الطول (متر)</label>
  <input type="number" id="length" step="0.01">

  <label for="width">العرض (متر)</label>
  <input type="number" id="width" step="0.01">

  <label for="quantity">الكمية</label>
  <input type="number" id="quantity" value="1">

  <label>إضافات:</label>
  <input type="checkbox" id="addCurtain"> ستارة داخلية (26 ريال للمتر)
  <br>
  <input type="checkbox" id="addMesh"> شبكة

  <div class="currency-toggle">
    <label>تحويل العملة:</label>
    <input type="checkbox" id="convertCurrency"> إلى رينبي صيني (1 ريال عماني = 18.41 رينبي)
  </div>

  <button onclick="calculate()">إضافة النتيجة</button>
  <button onclick="clearResults()">🗑️ حذف النتائج</button>
  <button onclick="saveAsWord()">💾 حفظ كملف Word</button>

  <div class="results" id="resultsBox"></div>
</div>

<script>
const priceData = {
  "نوافذ": {
    "دبل جلاس دبل فريم": { base: { "ثابتة": 34, "حركة": 73, "حركتين": 92 }, factor: 0.13 },
    "دبل جلاس سنجل فريم": { base: { "ثابتة": 26, "حركة": 46, "حركتين": 58 }, factor: 0.13 },
    "سنجل جلاس سنجل فريم": { base: { "ثابتة": 20, "حركة": 43, "حركتين": 47 }, factor: 0.13 },
    "نوافذ السلايدنج": { base: { "سلايد": 0 }, factor: 0.13 },
    "النوافذ الكهربائية": { base: { "كهربائية": 102 }, factor: 0.13 },
    "سكاي لايت": { base: { "بدون مكينة": 56, "مع مكينة": 145 }, factor: 0.13 },
    "كارتن وول": { base: { "ثقيل": 56, "خفيف": 45 }, factor: 0.13 },
    "الشبك": { base: { "باب": 39, "فولدنج": 18, "سلايد": 14 }, factor: 0.13 }
  },
  "أبواب": {
    "باب المدخل": { base: { "زينك": 66, "ستينلس ستيل": 120, "كاست المنيوم": 168 }, factor: 0.2, plus10: true },
    "أبواب WPC": { base: { "فارغ": 45, "مع خشب": 50, "مع حشوة ضد الصوت": 60, "مع فريم ألمنيوم": 67, "سلايدنج WPC": 65 }, factor: 0.11 },
    "أبواب الألمنيوم": { base: { "فارغ": 65, "مع خشب": 75, "فل ألمنيوم": 85, "مخفي": 110, "خارجي": 61 }, factor: 0.11 },
    "باب دورات المياه": { base: { "النوع الجديد": 55, "النوع الأقدم": 45, "مخفي زجاجي": 65 }, factor: 0.11 },
    "أبواب الحدائق": { base: { "كاست المنيوم": 91 }, factor: 0.2 },
    "حواجز": { base: { "درج": 43, "حمام": 32 }, factor: 0.05 }
  },
  "أبواب سحب": {
    "داخلي زجاج": { base: { "سحب": 38 }, factor: 0.13 },
    "داخلي متين": { base: { "سحب": 41 }, factor: 0.13 },
    "خارجي جزء مفتوح": { base: { "سحب": 55 }, factor: 0.13 },
    "خارجي جزئين مفتوحات": { base: { "سحب": 58 }, factor: 0.13 },
    "WPC سلايد": { base: { "سحب": 61 }, factor: 0.11 },
    "فولدنج داخلي": { base: { "سحب": 39 }, factor: 0.13 },
    "فولدنج خارجي": { base: { "سحب": 56 }, factor: 0.13 },
    "شتر دور خارجي": { base: { "سحب": 28 }, factor: 0.13 }
  }
};

let results = [];

document.getElementById('category').addEventListener('change', function() {
  const typeSelect = document.getElementById('type');
  typeSelect.innerHTML = '';
  const selected = priceData[this.value];
  for (let key in selected) {
    const option = document.createElement('option');
    option.value = key;
    option.textContent = key;
    typeSelect.appendChild(option);
  }
  typeSelect.dispatchEvent(new Event('change'));
});

document.getElementById('type').addEventListener('change', function() {
  const movementSelect = document.getElementById('movement');
  movementSelect.innerHTML = '';
  const cat = document.getElementById('category').value;
  const type = this.value;
  const selected = priceData[cat][type];
  for (let move in selected.base) {
    const opt = document.createElement('option');
    opt.value = move;
    opt.textContent = move;
    movementSelect.appendChild(opt);
  }

  // preset dimensions for أبواب
  if (cat === "أبواب" && ["أبواب WPC", "أبواب الألمنيوم", "باب دورات المياه"].includes(type)) {
    document.getElementById("length").value = 2.2;
    document.getElementById("width").value = 1;
  }
});

function calculate() {
  const cat = document.getElementById('category').value;
  const type = document.getElementById('type').value;
  const move = document.getElementById('movement').value;
  const len = parseFloat(document.getElementById('length').value);
  const wid = parseFloat(document.getElementById('width').value);
  const qty = parseInt(document.getElementById('quantity').value) || 1;
  const curtain = document.getElementById('addCurtain').checked;
  const mesh = document.getElementById('addMesh').checked;
  const convert = document.getElementById('convertCurrency').checked;

  if (!cat || !type || !move || isNaN(len) || isNaN(wid)) return alert("أدخل جميع البيانات");

  const area = len * wid;
  const item = priceData[cat][type];
  let basePrice = item.base[move] || 0;
  let price = basePrice * area;

  if (item.plus10) price += 10;
  let shipping = area * (item.factor || 0) * 48;
  let total = (price + shipping) * qty;

  if (curtain) total += (len * wid) * 26;
  if (mesh && cat === "نوافذ") total += 14 * area;

  let currency = "ريال عماني";
  if (convert) {
    total *= 18.41;
    currency = "رينبي صيني";
  }

  const resultText = `
✅ النوع: ${type} - ${move}
📏 المقاس: ${len}م × ${wid}م
🧮 الكمية: ${qty}
🚚 الشحن: ${shipping.toFixed(2)} ريال
💰 المجموع: ${total.toFixed(2)} ${currency}
--------------------------
`;

  results.push(resultText);
  document.getElementById('resultsBox').innerText = results.join('\n');
}

function clearResults() {
  results = [];
  document.getElementById('resultsBox').innerText = '';
}

function saveAsWord() {
  const content = results.join('\n');
  const blob = new Blob(["\ufeff" + content], {
    type: "application/msword"
  });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "النتائج.doc";
  a.click();
}
</script>

</body>
</html>
