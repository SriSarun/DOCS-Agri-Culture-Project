மிகவும் அருமை! நமது விலை பகுப்பாய்வு டாஷ்போர்டை அடுத்த கட்டத்திற்கு எடுத்துச் செல்வோம். இதுவரை நாம் ஒரு பயிரின் விலை விவரங்களை ஆராய்ந்தோம். இப்போது, **பயிர்களுக்கு இடையேயான ஒப்பீட்டு பகுப்பாய்வு (Cross-Crop Comparative Analysis)** என்ற சக்திவாய்ந்த அம்சத்தைச் சேர்ப்போம்.

விவசாயிகள் மற்றும் வர்த்தகர்களுக்கு, "இந்த மாதம் நெல் பயிரிடுவதா அல்லது கரும்பு பயிரிடுவதா?" அல்லது "எந்தப் பயிரின் விலை கடந்த 6 மாதங்களில் சீராக உயர்ந்துள்ளது?" போன்ற கேள்விகளுக்குப் பதிலளிப்பது மிகவும் முக்கியம். இந்த அம்சம் அதற்கு உதவும்.

### திட்டம்: பயிர் ஒப்பீட்டு டாஷ்போர்டு (Crop Comparison Dashboard)

1.  **UI மாற்றம் (முகப்புப் பக்கம்):**
    *   முகப்புப் பக்கத்தில் உள்ள ஒவ்வொரு பயிர் கார்டிலும் ஒரு "ஒப்பிடச் சேர்" (Add to Compare) என்ற தேர்வுப்பெட்டியை (checkbox) வைப்போம்.
    *   பயனர் பயிர்களைத் தேர்ந்தெடுக்கும்போது, திரையின் கீழே அல்லது ஓரத்தில் ஒரு சிறிய "ஒப்பீட்டு பட்டி" (Comparison Tray) தோன்றி, தேர்ந்தெடுக்கப்பட்ட பயிர்களைக் காட்டும்.
    *   இந்தப் பட்டியில், "ஒப்பிடு" (Compare) என்ற பொத்தான் இருக்கும்.

2.  **புதிய பக்கம் அல்லது மாடல் (New Page or Modal):**
    *   "ஒப்பிடு" பொத்தானை அழுத்தும்போது, ஒரு புதிய ஒப்பீட்டுப் பக்கம் (`/compare`) திறக்கும். இந்தப் பக்கத்தில் தேர்ந்தெடுக்கப்பட்ட பயிர்களின் விலைப்போக்கை ஒரே வரைபடத்தில் காட்டுவோம்.

3.  **புதிய Backend API:**
    *   `GET /api/crops/compare?ids=id1,id2,id3` என்ற புதிய API வழித்தடத்தை உருவாக்குவோம்.
    *   இது URL-ல் இருந்து பல பயிர் ID-களைப் பெற்று, ஒவ்வொரு பயிரின் மாதாந்திர சராசரி விலைத் தரவையும் ஒன்றாக இணைத்து, Chart.js-க்கு ஏற்ற வடிவத்தில் திருப்பி அனுப்பும்.

4.  **ஒப்பீட்டு வரைபடத்தை உருவாக்குதல் (Frontend):**
    *   ஒப்பீட்டுப் பக்கத்திற்கான JavaScript, இந்த புதிய API-ஐ அழைத்து, கிடைக்கும் தரவைக் கொண்டு ஒரு ಬಹು-வரிசை கோட்டு வரைபடத்தை (multi-line chart) உருவாக்கும். ஒவ்வொரு பயிரின் விலைப்போக்கும் ஒரு தனி நிறத்தில் கோடாகக் காட்டப்படும்.

---

### படி 1: முகப்புப் பக்க UI-ஐ மேம்படுத்துதல்

#### 1. தேர்வுப்பெட்டியைச் சேர்த்தல் (`public/js/main.js`)
`main.js`-ல், கார்டை உருவாக்கும்போது, ஒரு தேர்வுப்பெட்டியைச் சேர்ப்போம்.

```javascript
// public/js/main.js

// ... `loadCrops` செயல்பாட்டிற்குள் `forEach` சுழற்சியில் ...
const cropCard = `
    <div class="bg-white rounded-lg shadow-md overflow-hidden flex flex-col">
        <a href="/crops/${crop._id}" class="block p-5 flex-grow">
            <h2 class="text-xl font-bold text-green-800">${crop.name}</h2>
            <!-- ... மற்ற விவரங்கள் ... -->
        </a>
        <div class="p-3 bg-gray-50 border-t">
            <label class="inline-flex items-center cursor-pointer">
                <input type="checkbox" class="compare-checkbox h-5 w-5 rounded text-green-600" data-id="${crop._id}" data-name="${crop.name}">
                <span class="ml-2 text-sm text-gray-700">ஒப்பிடச் சேர்</span>
            </label>
        </div>
    </div>
`;
// ...
```
*   `<a>` குறிச்சொல்லுக்கு வெளியே ஒரு புதிய `div`-ல் தேர்வுப்பெட்டியைச் சேர்த்துள்ளோம். `data-id` மற்றும் `data-name` பண்புகள் மூலம் பயிரின் விவரங்களைச் சேமிக்கிறோம்.

#### 2. ஒப்பீட்டுப் பட்டியை (Comparison Tray) உருவாக்குதல் (`views/home.hbs` & `public/js/main.js`)

`views/home.hbs`-ல், `<body>`-ன் முடிவில் இந்த HTML-ஐச் சேர்க்கவும்.
```html
<!-- views/home.hbs -->
<!-- ஒப்பீட்டுப் பட்டி (ஆரம்பத்தில் மறைக்கப்பட்டிருக்கும்) -->
<div id="compare-tray" class="fixed bottom-0 left-0 right-0 bg-gray-800 text-white p-4 transform translate-y-full transition-transform duration-300 shadow-lg z-50">
    <div class="container mx-auto flex justify-between items-center">
        <div>
            <span class="font-bold">ஒப்பீட்டிற்காக தேர்ந்தெடுக்கப்பட்டவை:</span>
            <span id="compare-list" class="ml-2"></span>
        </div>
        <a id="compare-btn" href="#" class="bg-green-600 hover:bg-green-700 px-6 py-2 rounded-md font-bold">ஒப்பிடு</a>
    </div>
</div>
```
*   `transform translate-y-full` மற்றும் `transition-transform` வகுப்புகள், பட்டி திரைக்கு வெளியே மறைந்திருக்கவும், தேவைப்படும்போது மென்மையாக மேலே வரவும் உதவும்.

`public/js/main.js`-ல், இந்தத் தர்க்கத்தைச் சேர்க்கவும்:
```javascript
// public/js/main.js
document.addEventListener('DOMContentLoaded', () => {
    // ...
    const compareTray = document.getElementById('compare-tray');
    const compareList = document.getElementById('compare-list');
    const compareBtn = document.getElementById('compare-btn');
    let selectedForCompare = new Map(); // Map để lưu trữ ID và tên

    container.addEventListener('change', (event) => {
        if (event.target.classList.contains('compare-checkbox')) {
            const id = event.target.dataset.id;
            const name = event.target.dataset.name;
            if (event.target.checked) {
                if (selectedForCompare.size < 3) {
                    selectedForCompare.set(id, name);
                } else {
                    alert('அதிகபட்சம் 3 பயிர்களை மட்டுமே ஒப்பிட முடியும்.');
                    event.target.checked = false;
                }
            } else {
                selectedForCompare.delete(id);
            }
            updateCompareTray();
        }
    });

    function updateCompareTray() {
        if (selectedForCompare.size > 0) {
            compareList.innerHTML = [...selectedForCompare.values()].map(name => `<span class="bg-gray-600 px-2 py-1 rounded-md text-sm mr-2">${name}</span>`).join('');
            const ids = [...selectedForCompare.keys()].join(',');
            compareBtn.href = `/compare?ids=${ids}`;
            compareTray.classList.remove('translate-y-full');
        } else {
            compareTray.classList.add('translate-y-full');
        }
    }
});
```
*   `Map` பயன்படுத்துவது, நகல் ID-கள் சேர்க்கப்படுவதைத் தடுக்கும்.
*   `updateCompareTray` செயல்பாடு, தேர்ந்தெடுக்கப்பட்ட பயிர்களைப் பட்டியில் காட்டி, "ஒப்பிடு" பொத்தானின் இணைப்பை (`href`) உருவாக்குகிறது.

---

### படி 2: ஒப்பீட்டுப் பக்கம் மற்றும் Backend API

#### 1. புதிய ஒப்பீட்டுப் பக்கம் (`views/compare.hbs` மற்றும் `server.js`)

`server.js`-ல் ஒரு புதிய வழித்தடத்தைச் சேர்க்கவும்:
```javascript
// server.js
app.get('/compare', (req, res) => {
    res.render('compare', { title: 'பயிர் ஒப்பீடு' });
});
```

`views/compare.hbs`-ஐ உருவாக்கவும்:
```html
<!-- views/compare.hbs -->
<h1 class="text-3xl font-bold text-center text-green-800 mb-8">பயிர்களின் விலை ஒப்பீடு</h1>
<div class="bg-white p-6 rounded-lg shadow-md">
    <canvas id="comparisonChart"></canvas>
</div>
<script src="/js/compare.js"></script>
```

#### 2. புதிய Backend API (`controllers/cropController.js` மற்றும் `routes/cropRoutes.js`)

`cropController.js`-ல் புதிய செயல்பாட்டைச் சேர்க்கவும்:
```javascript
// controllers/cropController.js
const compareCrops = async (req, res) => {
    try {
        const db = getDB();
        const idsString = req.query.ids;
        if (!idsString) return res.status(400).json({ message: 'IDs தேவை' });

        const ids = idsString.split(',').map(id => new ObjectId(id));
        
        const crops = await db.collection('crops').find({ _id: { $in: ids } }).toArray();

        const comparisonData = {
            labels: [], // மாதங்கள் (எ.கா., "2024-06", "2024-07")
            datasets: []
        };
        
        const allMonths = new Set();
        crops.forEach(crop => {
            crop.marketPriceHistory.forEach(p => {
                allMonths.add(new Date(p.date).toISOString().slice(0, 7));
            });
        });
        comparisonData.labels = [...allMonths].sort();
        
        const colors = ['#16a34a', '#ea580c', '#2563eb'];
        crops.forEach((crop, index) => {
            const dataset = {
                label: crop.name,
                data: [],
                borderColor: colors[index % colors.length],
                tension: 0.1
            };
            comparisonData.labels.forEach(month => {
                const pricesInMonth = crop.marketPriceHistory
                    .filter(p => new Date(p.date).toISOString().slice(0, 7) === month)
                    .map(p => p.price);
                
                const avgPrice = pricesInMonth.length > 0
                    ? pricesInMonth.reduce((a, b) => a + b, 0) / pricesInMonth.length
                    : null; // தரவு இல்லாத மாதங்களுக்கு null
                dataset.data.push(avgPrice);
            });
            comparisonData.datasets.push(dataset);
        });

        res.status(200).json(comparisonData);
    } catch(err) { /* ... பிழை கையாளுதல் ... */ }
};
```
*   இந்த API, அனைத்துப் பயிர்களிலும் உள்ள மாதங்களின் ஒரு முழுமையான பட்டியலை (`allMonths`) முதலில் உருவாக்குகிறது.
*   பிறகு, ஒவ்வொரு பயிருக்கும், ஒவ்வொரு மாதத்திற்கும் சராசரி விலையைக் கணக்கிடுகிறது. ஒரு குறிப்பிட்ட மாதத்தில் ஒரு பயிருக்கு விலைத் தரவு இல்லை என்றால், `null` மதிப்பைச் சேர்க்கிறது. இது வரைபடத்தில் கோட்டில் ஒரு இடைவெளியைக் காட்டும்.

`routes/cropRoutes.js`-ல் புதிய வழித்தடத்தைச் சேர்க்கவும்:
```javascript
// routes/cropRoutes.js
router.get('/crops/compare', compareCrops);
```

---

### படி 3: ஒப்பீட்டு வரைபடத்தை உருவாக்குதல் (`public/js/compare.js`)

`public/js` கோப்புறையில் `compare.js` என்ற புதிய கோப்பை உருவாக்கவும்.
```javascript
// public/js/compare.js
document.addEventListener('DOMContentLoaded', () => {
    const renderComparisonChart = async () => {
        const params = new URLSearchParams(window.location.search);
        const ids = params.get('ids');

        if (!ids) return;

        try {
            const response = await fetch(`/api/crops/compare?ids=${ids}`);
            const data = await response.json();

            const ctx = document.getElementById('comparisonChart').getContext('2d');
            new Chart(ctx, {
                type: 'line',
                data: data, // API அனுப்பும் தரவு ஏற்கனவே Chart.js வடிவத்தில் உள்ளது
                options: {
                    scales: { y: { beginAtZero: false, title: { display: true, text: 'சராசரி விலை (₹/kg)' } } }
                }
            });
        } catch (error) {
            console.error('ஒப்பீட்டு வரைபடத்தை ஏற்றுவதில் பிழை:', error);
        }
    };
    renderComparisonChart();
});
```

### சோதித்துப் பார்ப்போம்

1.  சேவையகத்தை மறுதொடக்கம் செய்யவும்.
2.  முகப்புப் பக்கத்திற்குச் செல்லவும். இப்போது ஒவ்வொரு கார்டிலும் "ஒப்பிடச் சேர்" தேர்வுப்பெட்டி இருக்கும்.
3.  இரண்டு அல்லது மூன்று பயிர்களைத் தேர்ந்தெடுக்கவும். திரையின் கீழே ஒப்பீட்டுப் பட்டி தோன்றி, நீங்கள் தேர்ந்தெடுத்த பயிர்களின் பெயர்களைக் காட்டும்.
4.  "ஒப்பிடு" பொத்தானை அழுத்தவும்.
5.  நீங்கள் `/compare?ids=...` என்ற புதிய பக்கத்திற்கு அழைத்துச் செல்லப்படுவீர்கள். அங்கு, நீங்கள் தேர்ந்தெடுத்த பயிர்களின் மாதாந்திர விலைப்போக்கை ஒரே கோட்டு வரைபடத்தில், வெவ்வேறு நிறங்களில் காண்பீர்கள்.

இந்த அம்சம், பயனர்கள் தகவலறிந்த முடிவுகளை எடுக்க உதவும் ஒரு மிகச் சக்திவாய்ந்த பகுப்பாய்வுக் கருவியாகும்.