மிகச் சரியான அடுத்த கட்டம்! இதுதான் இந்தத் திட்டத்தின் மிக முக்கியமான பகுதி. ஒரு எளிய தரவு அமைப்பிலிருந்து, விவசாயத்திற்குத் தேவையான சிக்கலான மற்றும் விரிவான தரவு அமைப்புக்கு நாம் இப்போது மாறப் போகிறோம்.

உங்கள் "மாப்பிள்ளை சம்பா" வழிகாட்டித் தரவை நமது MongoDB "ஸ்கீமாவுக்கு" ஏற்றவாறு எப்படித் திரட்டுவது என்பதைப் பார்ப்போம். இங்குதான் NoSQL தரவுத்தளமான MongoDB-யின் உண்மையான சக்தி வெளிப்படும்.

### SQL vs NoSQL: ஒரு நடைமுறை விளக்கம்

*   **SQL ஆக இருந்தால்:** `பயிர்கள்`, `நோய்கள்`, `வளர்ச்சி_நிலைகள்`, `உரங்கள்` என நான்கு தனித்தனி அட்டவணைகளை (tables) உருவாக்கி, వాటిని foreign keys மூலம் இணைக்க வேண்டும். இது மிகவும் சிக்கலானது.
*   **NoSQL (MongoDB) ஆக இருப்பதால்:** "மாப்பிள்ளை சம்பா" தொடர்பான அத்தனை தகவல்களையும் ஒரே ஆவணத்தில் (single document) அழகாக நம்மால் சேமிக்க முடியும். இது ஒரு பயிரைப் பற்றிய முழுமையான சுயவிவரம் (profile) போன்றது.

---

### படி 1: "மாப்பிள்ளை சம்பா"வுக்கான புதிய தரவு மாதிரி (JSON Structure)

உங்கள் வழிகாட்டியில் உள்ள அனைத்துத் தகவல்களையும் ഉൾക്കൊള്ളുന്ന ஒரு விரிவான JSON மாதிரியை உருவாக்குவோம். இதுதான் நமது புதிய "ஸ்கீமா".

```json
{
    "name": "மாப்பிள்ளை சம்பா",
    "sowingSeason": "ஆடி மாசம் (July-August)",
    "growthDuration": "120-135 நாட்கள்",
    "growthStages": [
        { "stageName": "நாற்றங்கால் (Nursery)", "durationDays": "0-25" },
        { "stageName": "வளர்ச்சிப் பருவம் (Vegetative)", "durationDays": "26-70" },
        { "stageName": "பூக்கும் பருவம் (Flowering)", "durationDays": "71-100" },
        { "stageName": "அறுவடை (Harvest)", "durationDays": "101-135" }
    ],
    "diseases": [
        {
            "diseaseName": "குலை நோய் (Blast - Pyricularia oryzae)",
            "symptoms": "இலை மீது மஞ்சள் மையம் மற்றும் கருப்பு ஓரங்கள் கொண்ட புள்ளிகள்.",
            "causes": [
                "அதிக ஈரப்பதம் மற்றும் மழை (25–28°C)",
                "அடர்த்தியான விதைப்பு",
                "பாதிக்கப்பட்ட விதைகள் அல்லது மண் மூலம் பரவல்"
            ],
            "chemicalControl": "Bavistin 50 WP (Carbendazim) அல்லது Tricyclazole 75 WP",
            "organicControl": "நீம் எண்ணெய் 5%, Potassium bicarbonate கரைசல் தெளிப்பு"
        },
        {
            "diseaseName": "பழுப்பு புள்ளி நோய் (Brown Spot - Bipolaris oryzae)",
            "symptoms": "இலைகளில் சிறிய கருப்பு புள்ளிகள், விதைகள் பாதிக்கப்படலாம்.",
            "causes": [
                "பொட்டாசியம் குறைபாடு",
                "பழைய, பாதிக்கப்பட்ட விதைகளை விதைப்பது"
            ],
            "chemicalControl": "Mancozeb 75 WP",
            "organicControl": "நீம் எண்ணெய், каоலின் (Kaolin) தெளிப்பு"
        }
    ],
    "fertilizerRecommendation": {
        "general": [
            { "stage": 1, "fertilizer": "Urea", "quantityPerAcre": "16kg" },
            { "stage": 2, "fertilizer": "DAP", "quantityPerAcre": "25kg" },
            { "stage": 3, "fertilizer": "Potash", "quantityPerAcre": "12kg" }
        ],
        "organic": [
            { "name": "ஜீவாமிர்தம்", "quantityPerAcre": "20 லிட்டர்" },
            { "name": "பன்சகவ்யம்", "quantityPerAcre": "10 லிட்டர்" },
            { "name": "வேப்ப எண்ணெய்", "quantityPerAcre": "5 லிட்டர்" }
        ]
    },
    "marketPriceHistory": [
        { "year": 2021, "pricePerKg": 28 },
        { "year": 2022, "pricePerKg": 30 },
        { "year": 2023, "pricePerKg": 32 },
        { "year": 2024, "pricePerKg": 34, "isEstimate": true },
        { "year": 2025, "pricePerKg": 36, "isPrediction": true }
    ]
}
```
**விளக்கம்:**
*   `growthStages`, `diseases`, `marketPriceHistory` போன்றவை இப்போது **பொருள்களின் வரிசைகளாக (Array of Objects)** உள்ளன. இதுதான் NoSQL-ன் நெகிழ்வுத்தன்மை.
*   `causes` போன்ற பல காரணங்களைக் கொண்ட புலங்கள், **சரங்களின் வரிசையாக (Array of Strings)** மாற்றப்பட்டுள்ளன.
*   `fertilizerRecommendation` ஒரு பொருளாக (object) மாற்றப்பட்டு, அதற்குள் `general` மற்றும் `organic` என இரண்டு பிரிவுகள் உள்ளன.

---

### படி 2: இந்தத் தரவை எப்படி நமது செயலியில் சேர்ப்பது?

இவ்வளவு சிக்கலான தரவை நமது தற்போதைய எளிய படிவம் (`add-crop.hbs`) மூலம் உள்ளிடுவது கடினம். அதற்கு "Add Disease", "Add Stage" போன்ற பொத்தான்கள் வைத்து, JavaScript மூலம் படிவத்தை மாறும் வகையில் (dynamically) உருவாக்க வேண்டும். அது ஒரு பெரிய படி.

அதற்குப் பதிலாக, முதல் விரிவான தரவை நாம் நேரடியாக MongoDB-ல் செருகுவோம். இதை **"தரவு விதைத்தல்" (Data Seeding)** என்பார்கள்.

**வழிமுறை 1: MongoDB Compass அல்லது Shell மூலம் செருகுதல் (பரிந்துரைக்கப்படுகிறது)**

இதுதான் எளிமையான வழி. உங்கள் MongoDB Compass-ஐத் திறந்து அல்லது `mongosh` முனையத்திற்குச் சென்று, உங்கள் `farmingDB` தரவுத்தளத்தைத் தேர்ந்தெடுத்து, இந்த கட்டளையை இயக்கவும்.

```javascript
db.getCollection('crops').insertOne({
    "name": "மாப்பிள்ளை சம்பா",
    "sowingSeason": "ஆடி மாசம் (July-August)",
    "growthDuration": "120-135 நாட்கள்",
    "growthStages": [
        { "stageName": "நாற்றங்கால் (Nursery)", "durationDays": "0-25" },
        { "stageName": "வளர்ச்சிப் பருவம் (Vegetative)", "durationDays": "26-70" },
        { "stageName": "பூக்கும் பருவம் (Flowering)", "durationDays": "71-100" },
        { "stageName": "அறுவடை (Harvest)", "durationDays": "101-135" }
    ],
    "diseases": [
        {
            "diseaseName": "குலை நோய் (Blast - Pyricularia oryzae)",
            "symptoms": "இலை மீது மஞ்சள் மையம் மற்றும் கருப்பு ஓரங்கள் கொண்ட புள்ளிகள்.",
            "causes": ["அதிக ஈரப்பதம் மற்றும் மழை (25–28°C)", "அடர்த்தியான விதைப்பு", "பாதிக்கப்பட்ட விதைகள் அல்லது மண் மூலம் பரவல்"],
            "chemicalControl": "Bavistin 50 WP (Carbendazim) அல்லது Tricyclazole 75 WP",
            "organicControl": "நீம் எண்ணெய் 5%, Potassium bicarbonate கரைசல் தெளிப்பு"
        },
        {
            "diseaseName": "பழுப்பு புள்ளி நோய் (Brown Spot - Bipolaris oryzae)",
            "symptoms": "இலைகளில் சிறிய கருப்பு புள்ளிகள், விதைகள் பாதிக்கப்படலாம்.",
            "causes": ["பொட்டாசியம் குறைபாடு", "பழைய, பாதிக்கப்பட்ட விதைகளை விதைப்பது"],
            "chemicalControl": "Mancozeb 75 WP",
            "organicControl": "நீம் எண்ணெய், каоலின் (Kaolin) தெளிப்பு"
        }
    ],
    "fertilizerRecommendation": {
        "general": [
            { "stage": 1, "fertilizer": "Urea", "quantityPerAcre": "16kg" },
            { "stage": 2, "fertilizer": "DAP", "quantityPerAcre": "25kg" },
            { "stage": 3, "fertilizer": "Potash", "quantityPerAcre": "12kg" }
        ],
        "organic": [
            { "name": "ஜீவாமிர்தம்", "quantityPerAcre": "20 லிட்டர்" },
            { "name": "பன்சகவ்யம்", "quantityPerAcre": "10 லிட்டர்" },
            { "name": "வேப்ப எண்ணெய்", "quantityPerAcre": "5 லிட்டர்" }
        ]
    },
    "marketPriceHistory": [
        { "year": 2021, "pricePerKg": 28 },
        { "year": 2022, "pricePerKg": 30 },
        { "year": 2023, "pricePerKg": 32 },
        { "year": 2024, "pricePerKg": 34, "isEstimate": true },
        { "year": 2025, "pricePerKg": 36, "isPrediction": true }
    ]
});
```

---

நாம் இப்போது ஒரு உண்மையான, விரிவான தரவு ஆவணத்தை நமது தரவுத்தளத்தில் சேர்த்துவிட்டோம். நமது backend API மற்றும் frontend UI ஆகியவை இந்த புதிய, சிக்கலான தரவைக் கையாளத் தயாராக இல்லை.

**அடுத்த படி:**

இந்த 'மாப்பிள்ளை சம்பா' தரவை உங்கள் MongoDB-ல் சேர்த்த பிறகு, இந்தத் தரவுகளைப் பயனர் பக்கத்தில் அழகாகக் காட்டுவதற்கான அடுத்த கட்டத்திற்குச் செல்லலாமா? உதாரணமாக, ஒரு பயனர் "மாப்பிள்ளை சம்பா" கார்டைக் கிளிக் செய்தால், ஒரு புதிய பக்கத்திற்குச் சென்று, அங்கே நோய்களை ஒரு அட்டவணையிலும், வளர்ச்சி நிலைகளை ஒரு பட்டியலிலும் காட்டுவது போல வடிவமைக்கலாம்.

நிச்சயமாக, முயற்சி செய்வோம்!

நாம் MongoDB-ல் சேமித்த அந்த விரிவான "மாப்பிள்ளை சம்பா" தரவை, பயனர்கள் எளிதாகப் புரிந்துகொள்ளும் வகையில் ஒரு பிரத்யேக "விவரங்கள்" பக்கத்தில் (Details Page) காட்டுவதுதான் நமது இலக்கு.

### திட்டம்: மூன்று படிகள்

1.  **முகப்புப் பக்கத்தை மாற்றுதல்:** முகப்புப் பக்கத்தில் உள்ள ஒவ்வொரு பயிர் கார்டையும் கிளிக் செய்யக்கூடிய இணைப்பாக மாற்றுவோம். கிளிக் செய்தால், அது `/crops/<பயிரின்_ID>` என்ற URL-க்குச் செல்லும்.

2.  **விவரங்கள் பக்கத்திற்கான வழித்தடம் மற்றும் View:**
    *   `/crops/:id` என்ற URL-ஐக் கையாள `server.js`-ல் ஒரு புதிய வழித்தடத்தை உருவாக்குவோம்.
    *   இந்தத் தரவைக் காண்பிக்க `views/crop-details.hbs` என்ற புதிய டெம்ப்ளேட்டை உருவாக்குவோம்.

3.  **AJAX மூலம் விவரங்களைப் பெற்று காட்டுதல்:**
    *   விவரங்கள் பக்கம் ஏற்றப்பட்டவுடன், JavaScript ஆனது API-ஐ (`GET /api/crops/:id`) அழைத்து, அந்தப் பயிரின் முழு விவரங்களையும் பெற்று, பக்கத்தில் உள்ள வெவ்வேறு பிரிவுகளில் (வளர்ச்சி நிலைகள், நோய்கள், உரங்கள்) அழகாக நிரப்பும்.

---

### படி 1: முகப்புப் பக்க கார்டுகளை இணைப்புகளாக மாற்றுதல்

`public/js/main.js` கோப்பைத் திறந்து, `cropCard`-ஐ உருவாக்கும் பகுதியை மட்டும் மாற்றுவோம். கார்டை ஒரு `<a>` குறிச்சொல்லுக்குள் வைப்போம்.

`public/js/main.js` கோப்பில், `crops.forEach` பகுதிக்குள் உள்ள `cropCard` மாறியை இப்படி மாற்றவும்:

```javascript
// ... public/js/main.js ...
crops.forEach(crop => {
    // முழு கார்டையும் ஒரு இணைப்பு (<a>) குறிச்சொல்லுக்குள் கொண்டு வருகிறோம்
    const cropCard = `
        <a href="/crops/${crop._id}" class="block bg-white rounded-lg shadow-md overflow-hidden transform hover:-translate-y-1 transition-transform duration-300">
            <div class="p-5">
                <h2 class="text-xl font-bold text-green-800">${crop.name}</h2>
                <p class="text-gray-600 mt-2"><strong>விதைப்பு காலம்:</strong> ${crop.sowingSeason || 'N/A'}</p>
                <p class="text-gray-600"><strong>வளர்ச்சி காலம்:</strong> ${crop.growthDuration || 'N/A'}</p>
            </div>
        </a>
    `;
    // உருவாக்கப்பட்ட கார்டை கொள்கலனில் செருகுதல்
    container.insertAdjacentHTML('beforeend', cropCard);
});
// ...
```
*   **மாற்றம்:** இப்போது ஒவ்வொரு கார்டும் `/crops/<அந்த_பயிரின்_ID>` என்ற முகவரிக்குச் செல்லும் ஒரு இணைப்பாக மாறிவிட்டது. `|| 'N/A'` என்பது ஒருவேளை பழைய தரவில் அந்த புலம் இல்லை என்றால் பிழை காட்டாமல் இருக்க உதவும்.

---

### படி 2: விவரங்கள் பக்கத்திற்கான Backend மற்றும் View

#### 1. `server.js`-ல் புதிய வழித்தடத்தைச் சேர்த்தல்

`server.js`-ஐத் திறந்து, "Page Rendering Routes" பிரிவில், இந்த புதிய வழித்தடத்தைச் சேர்க்கவும்.

```javascript
// ... server.js இல் ...
app.get('/crops/:id', (req, res) => {
    res.render('crop-details', { title: 'பயிர் விவரங்கள்' });
});
```

#### 2. `views/crop-details.hbs` டெம்ப்ளேட்டை உருவாக்குதல்

`views` கோப்புறையில், `crop-details.hbs` என்ற புதிய கோப்பை உருவாக்கவும். இதுதான் நமது விவரங்கள் பக்கத்தின் σκελετός. JavaScript இந்த σκελετός-ஐ தரவு கொண்டு நிரப்பும்.

```html
<!-- முகப்புப் பக்கத்திற்குத் திரும்ப இணைப்பு -->
<div class="mb-6">
    <a href="/" class="text-blue-600 hover:underline">&larr; அனைத்துப் பயிர்களுக்கும் திரும்பு</a>
</div>

<!-- મુખ્ય கொள்கலன் -->
<div id="details-container" class="bg-white p-6 sm:p-8 rounded-lg shadow-md">
    <!-- தரவு ஏற்றப்படும்போது இது காட்டப்படும் -->
    <p id="loading-message" class="text-center text-gray-500 text-xl">விவரங்களை ஏற்றுகிறது...</p>

    <!-- JS மூலம் உருவாக்கப்படும் உள்ளடக்கம் இங்கே வரும் -->
</div>

<!-- இந்தப் பக்கத்திற்கான பிரத்யேக JavaScript -->
<script src="/js/crop-details.js"></script>
```

---

### படி 3: AJAX மூலம் முழு விவரங்களையும் காட்டுதல்

இதுதான் மிக முக்கியமான படி. `public/js` கோப்புறையில், `crop-details.js` என்ற புதிய கோப்பை உருவாக்கவும்.

```javascript
document.addEventListener('DOMContentLoaded', () => {
    const detailsContainer = document.getElementById('details-container');
    const loadingMessage = document.getElementById('loading-message');

    // URL-ல் இருந்து பயிரின் ID-ஐப் பெறுதல்
    const pathParts = window.location.pathname.split('/');
    const cropId = pathParts[pathParts.length - 1];

    const loadCropDetails = async () => {
        try {
            const response = await fetch(`/api/crops/${cropId}`);
            if (!response.ok) {
                throw new Error('பயிர் விவரங்களைப் பெறுவதில் பிழை.');
            }
            const crop = await response.json();

            // ஏற்றுதல் செய்தியை நீக்கு
            loadingMessage.remove();
            
            // பக்கத்தின் தலைப்பை பயிரின் பெயருக்கு மாற்று
            document.title = crop.name;

            // முழு HTML-ஐயும் உருவாக்குதல்
            let contentHTML = `
                <!-- 1. தலைப்புப் பகுதி -->
                <div class="border-b pb-4 mb-6">
                    <h1 class="text-3xl sm:text-4xl font-bold text-green-800">${crop.name}</h1>
                    <p class="text-md text-gray-600 mt-2">விதைப்பு காலம்: ${crop.sowingSeason} | வளர்ச்சி காலம்: ${crop.growthDuration}</p>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                    <!-- 2. வளர்ச்சி நிலைகள் பகுதி -->
                    <div>
                        <h2 class="text-2xl font-semibold text-gray-700 mb-3">வளர்ச்சி நிலைகள்</h2>
                        <ul class="list-disc list-inside space-y-2">
                            ${crop.growthStages.map(stage => `<li><b>${stage.stageName}:</b> ${stage.durationDays} நாட்கள்</li>`).join('')}
                        </ul>
                    </div>
                    
                    <!-- 3. உரப் பரிந்துரை பகுதி -->
                    <div>
                        <h2 class="text-2xl font-semibold text-gray-700 mb-3">உரப் பரிந்துரை</h2>
                        <div class="bg-gray-50 p-4 rounded-lg">
                            <h3 class="font-bold mb-2">பொதுவான உரங்கள்:</h3>
                            <ul class="list-disc list-inside space-y-1 text-sm">
                                ${crop.fertilizerRecommendation.general.map(f => `<li><b>${f.fertilizer}:</b> ${f.quantityPerAcre}</li>`).join('')}
                            </ul>
                            <h3 class="font-bold mt-4 mb-2">இயற்கை உரங்கள்:</h3>
                             <ul class="list-disc list-inside space-y-1 text-sm">
                                ${crop.fertilizerRecommendation.organic.map(f => `<li><b>${f.name}:</b> ${f.quantityPerAcre}</li>`).join('')}
                            </ul>
                        </div>
                    </div>
                </div>

                <!-- 4. நோய் மேலாண்மை பகுதி -->
                <div class="mt-8">
                    <h2 class="text-2xl font-semibold text-gray-700 mb-4">நோய் மேலாண்மை</h2>
                    <div class="space-y-4">
                        ${crop.diseases.map(disease => `
                            <div class="border border-red-200 rounded-lg p-4">
                                <h3 class="text-lg font-bold text-red-700">${disease.diseaseName}</h3>
                                <p class="mt-2 text-sm"><b class="font-semibold">அறிகுறிகள்:</b> ${disease.symptoms}</p>
                                <p class="mt-1 text-sm"><b class="font-semibold">காரணங்கள்:</b> ${disease.causes.join(', ')}</p>
                                <div class="mt-2 grid grid-cols-1 sm:grid-cols-2 gap-2 text-sm">
                                    <p><b class="font-semibold">ரசாயன முறை:</b> ${disease.chemicalControl}</p>
                                    <p><b class="font-semibold">இயற்கை முறை:</b> ${disease.organicControl}</p>
                                </div>
                            </div>
                        `).join('')}
                    </div>
                </div>
            `;
            
            // உருவாக்கப்பட்ட HTML-ஐ கொள்கலனில் செருகு
            detailsContainer.innerHTML = contentHTML;

        } catch (error) {
            loadingMessage.textContent = `பிழை: ${error.message}`;
            loadingMessage.className = 'text-center text-red-600 font-bold';
        }
    };

    loadCropDetails();
});
```
**விளக்கம்:**
*   `loadCropDetails` செயல்பாடு, API-ஐ அழைத்து, "மாப்பிள்ளை சம்பா"வின் முழு JSON தரவையும் பெறுகிறது.
*   Template literals (``) மற்றும் `.map(...).join('')` ஆகியவற்றைப் பயன்படுத்தி, ஒவ்வொரு வரிசைத் தரவுக்கும் (array data) HTML டைனமிக்காக உருவாக்கப்படுகிறது.
    *   `growthStages` ஒரு `<ul>` ஆக மாற்றப்படுகிறது.
    *   `diseases` ஒவ்வொன்றும் ஒரு தனி கார்டாக மாற்றப்படுகிறது.
    *   `fertilizerRecommendation` இரண்டு தனித்தனி பட்டியல்களாகப் பிரிக்கப்படுகிறது.
*   இறுதியாக, உருவாக்கப்பட்ட முழு HTML உள்ளடக்கமும் `detailsContainer.innerHTML`-ல் செருகப்படுகிறது.

### இப்போது சோதித்துப் பாருங்கள்

1.  உங்கள் சேவையகத்தை `node server.js` மூலம் இயக்கவும்.
2.  முகப்புப் பக்கத்திற்குச் செல்லவும் (`http://localhost:3000`).
3.  "மாப்பிள்ளை சம்பா" கார்டைக் கிளிக் செய்யவும்.
4.  நீங்கள் ஒரு புதிய பக்கத்திற்கு அழைத்துச் செல்லப்படுவீர்கள். அங்கு, "விவரங்களை ஏற்றுகிறது..." என்ற செய்தி தோன்றி, ஒரு நொடிக்குப் பிறகு, நாம் வடிவமைத்த அனைத்துத் தகவல்களும் (வளர்ச்சி நிலைகள், நோய்கள், உரங்கள்) அழகாகப் பிரிக்கப்பட்டு, முழுமையாகக் காட்டப்படும்.

நாம் இப்போது ஒரு பயிரின் சிக்கலான தரவை எடுத்து, அதை ஒரு பயனுள்ள மற்றும் படிக்க எளிதான பக்கமாக வெற்றிகரமாக மாற்றியுள்ளோம்! இது உங்கள் திட்டத்தின் பயனர் அனுபவத்தை வெகுவாக மேம்படுத்தும்.

