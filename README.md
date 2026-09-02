from pathlib import Path
import zipfile, shutil, json

src = Path("/mnt/data/v8build/index.html")
outdir = Path("/mnt/data/v9build")
outdir.mkdir(exist_ok=True)

html = src.read_text(encoding="utf-8")

# Add V9 navigation buttons before global navigation.
if 'onclick="show(\'flights\')"' not in html:
    html = html.replace(
        '<button onclick="show(\'hotels\')"',
        '<button onclick="show(\'flights\')" class="v9nav">✈️ <span id="navFlights">الطيران</span></button><button onclick="show(\'cars\')" class="v9nav">🚗 <span id="navCars">السيارات</span></button><button onclick="show(\'places\')" class="v9nav">🏛️ <span id="navPlaces">السياحة</span></button><button onclick="show(\'currency\')" class="v9nav">💱 <span id="navCurrency">العملات</span></button><button onclick="show(\'hotels\')"'
    )

sections = r'''
<section id="flights" class="hidden">
  <div class="hero"><h2>✈️ الرحلات الجوية</h2><p>ابحث عن رحلات الطيران، ثم انتقل إلى منصة الحجز لإكمال الحجز.</p></div>
  <div class="hotelSearch">
    <input id="flightFrom" placeholder="من: مدينة أو مطار" />
    <input id="flightTo" placeholder="إلى: مدينة أو مطار" />
    <input id="flightDate" type="date" />
    <select id="flightPassengers"><option>1</option><option selected>2</option><option>3</option><option>4</option><option>5</option></select>
    <button class="primary" onclick="searchFlights()">🔎 بحث الرحلات</button>
  </div>
  <div id="flightResults" class="grid"><article class="card"><h3>✈️ بحث عالمي</h3><p>أدخل مدينة المغادرة والوصول والتاريخ.</p></article></div>
</section>

<section id="cars" class="hidden">
  <div class="hero"><h2>🚗 تأجير السيارات</h2><p>ابحث عن سيارات للإيجار في وجهتك.</p></div>
  <div class="hotelSearch">
    <input id="carCity" placeholder="المدينة أو المطار" />
    <input id="carDate" type="date" />
    <button class="primary" onclick="searchCars()">🔎 بحث السيارات</button>
  </div>
  <div id="carResults" class="grid"><article class="card"><h3>🚗 تأجير عالمي</h3><p>ابدأ باختيار المدينة.</p></article></div>
</section>

<section id="places" class="hidden">
  <div class="hero"><h2>🏛️ الأماكن السياحية</h2><p>اكتشف المعالم والأماكن السياحية في أي مدينة.</p></div>
  <div class="hotelSearch">
    <input id="placeCity" placeholder="اكتب المدينة أو الدولة..." />
    <button class="primary" onclick="searchPlaces()">🔎 اكتشف الأماكن</button>
  </div>
  <div id="placeResults" class="grid"><article class="card"><h3>🌍 اكتشف العالم</h3><p>ابحث عن الأماكن السياحية في وجهتك.</p></article></div>
</section>

<section id="currency" class="hidden">
  <div class="hero"><h2>💱 تحويل العملات</h2><p>تحويل سريع بين العملات. السعر المعروض إرشادي ويتغير باستمرار.</p></div>
  <div class="hotelSearch">
    <input id="amount" type="number" value="1" min="0" />
    <select id="fromCurrency"><option>DZD</option><option>EUR</option><option>USD</option><option>GBP</option><option>SAR</option><option>AED</option></select>
    <select id="toCurrency"><option>EUR</option><option>USD</option><option>DZD</option><option>GBP</option><option>SAR</option><option>AED</option></select>
    <button class="primary" onclick="convertCurrency()">💱 تحويل</button>
  </div>
  <div id="currencyResult" class="grid"><article class="card"><h3>💱 حاسبة العملات</h3><p>أدخل المبلغ واختر العملتين.</p></article></div>
</section>
'''
if 'id="flights"' not in html:
    html = html.replace('<section id="hotels"', sections + '\n<section id="hotels"')

css = r'''
.v9nav{margin-left:4px}
'''
html = html.replace('</style>', css + '</style>')

js = r'''
function searchFlights(){
  const from=(document.getElementById('flightFrom').value||'').trim();
  const to=(document.getElementById('flightTo').value||'').trim();
  const date=document.getElementById('flightDate').value;
  if(!from||!to){document.getElementById('flightFrom').focus();return;}
  const q=encodeURIComponent(from+" "+to+" flights"+(date?" "+date:""));
  document.getElementById('flightResults').innerHTML=`<article class="card"><h3>✈️ ${from} → ${to}</h3><p>نتائج البحث الخارجية:</p><a class="btn" target="_blank" rel="noopener" href="https://www.google.com/travel/flights?q=${q}">Google Flights ↗</a></article>`;
}
function searchCars(){
  const city=(document.getElementById('carCity').value||'').trim();
  if(!city){document.getElementById('carCity').focus();return;}
  const q=encodeURIComponent(city+" car rental");
  document.getElementById('carResults').innerHTML=`<article class="card"><h3>🚗 ${city}</h3><p>ابحث عن سيارات للإيجار:</p><a class="btn" target="_blank" rel="noopener" href="https://www.google.com/search?q=${q}">بحث تأجير السيارات ↗</a></article>`;
}
function searchPlaces(){
  const city=(document.getElementById('placeCity').value||'').trim();
  if(!city){document.getElementById('placeCity').focus();return;}
  const q=encodeURIComponent(city+" tourist attractions");
  document.getElementById('placeResults').innerHTML=`<article class="card"><h3>🏛️ ${city}</h3><p>اكتشف المعالم والأماكن السياحية:</p><a class="btn" target="_blank" rel="noopener" href="https://www.google.com/search?q=${q}">اكتشف الأماكن ↗</a></article>`;
}
function convertCurrency(){
  const amount=Number(document.getElementById('amount').value||0);
  const from=document.getElementById('fromCurrency').value, to=document.getElementById('toCurrency').value;
  const url=`https://www.google.com/search?q=${encodeURIComponent(amount+" "+from+" to "+to)}`;
  document.getElementById('currencyResult').innerHTML=`<article class="card"><h3>💱 ${amount} ${from} → ${to}</h3><p>للحصول على السعر الحالي:</p><a class="btn" target="_blank" rel="noopener" href="${url}">عرض سعر التحويل ↗</a></article>`;
}
'''
html = html.replace('</script>\n</body>', js + '\n</script>\n</body>')

# Ensure show() includes new sections.
html = html.replace(
    "['home','login','signup','global','hotels','account']",
    "['home','login','signup','global','hotels','flights','cars','places','currency','account']"
)

(outdir/"index.html").write_text(html, encoding="utf-8")
for name in ["app_icon_512.png","feature_graphic_1024x500.jpg"]:
    p=Path("/mnt/data/v8build")/name
    if p.exists(): shutil.copy2(p,outdir/name)

(outdir/"V9_README.txt").write_text(
"""Schengen Visa DZ — V9
مساعد سفر عالمي

الجديد في V9:
✈️ البحث عن الرحلات عبر Google Flights.
🚗 البحث عن تأجير السيارات.
🏛️ البحث عن الأماكن السياحية.
💱 قسم تحويل العملات.
🏨 الفنادق العالمية من V8.
🛂 التأشيرات العالمية من الإصدارات السابقة.
🌐 تعدد اللغات.

ملاحظة:
الروابط الخارجية تنقل المستخدم إلى خدمات خارجية. الأسعار والتوفر والحجوزات النهائية تعتمد على تلك الخدمات.
"""
, encoding="utf-8")

zip_path=Path("/mnt/data/Schengen_Visa_DZ_Global_V9_By_Melbou_Kekouche.zip")
with zipfile.ZipFile(zip_path,"w",zipfile.ZIP_DEFLATED) as z:
    for f in outdir.iterdir(): z.write(f,f.name)
print(zip_path)
