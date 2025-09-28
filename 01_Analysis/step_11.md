**அடுத்ததாக, இவ்வளவு சிக்கலான தரவுகளை நிர்வாகி (`Admin`) எளிதாக உள்ளிடுவதற்கும், திருத்துவதற்கும் ஏற்றவாறு நமது 'Add Crop' மற்றும் 'Edit Crop' படிவங்களை எப்படி மேம்படுத்துவது என்று ஆராயலாமா?** (உதாரணமாக, "புதிய நோயைச் சேர்" பொத்தான் வைப்பது போன்றவை). இது ஒரு சவாலான ஆனால் சுவாரஸ்யமான படியாகும்.டியாகும்.

நிச்சயமாக. நீங்கள் குறிப்பிட்டது போல, மற்ற பிரிவுகளுக்கும் இதைச் செயல்படுத்துவோம்.

நாம் நோய்களுக்கு உருவாக்கிய அதே டைனமிக் தர்க்கத்தை (dynamic logic) இப்போது **வளர்ச்சி நிலைகள் (Growth Stages)** மற்றும் **உரப் பரிந்துரை (Fertilizer Recommendation)** ஆகிய பிரிவுகளுக்கும் விரிவுபடுத்துவோம்.

---

### படி 1: படிவத்தின் HTML-ஐ மேலும் மேம்படுத்துதல் (`views/add-crop.hbs`)

`add-crop.hbs` கோப்பில், வளர்ச்சி நிலைகள் மற்றும் உரங்களுக்கான `fieldset`-களைச் சேர்ப்போம். உரப் பரிந்துரையை "பொதுவான உரங்கள்" மற்றும் "இயற்கை உரங்கள்" என இரண்டாகப் பிரிப்பது UI-ஐ எளிதாக்கும்.

`views/add-crop.hbs` கோப்பின் முழு உள்ளடக்கத்தையும் இதைக் கொண்டு மாற்றவும்:

```html
<h1 class="text-2xl font-bold text-center text-green-800 mb-6">புதிய பயிர் விவரங்களைச் சேர்க்கவும்</h1>

<div class="max-w-4xl mx-auto bg-white p-8 rounded-lg shadow-md">
    <form id="add-crop-form">
        <!-- 1. அடிப்படை விவரங்கள் -->
        <fieldset class="border p-4 rounded-lg mb-6">
            <legend class="text-xl font-semibold px-2">அடிப்படை விவரங்கள்</legend>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div>
                    <label for="name" class="block text-gray-700 font-bold mb-1">பயிரின் பெயர்:</label>
                    <input type="text" id="name" name="name" class="w-full px-3 py-2 border rounded-lg" required>
                </div>
                <div>
                    <label for="sowingSeason" class="block text-gray-700 font-bold mb-1">விதைப்பு காலம்:</label>
                    <input type="text" id="sowingSeason" name="sowingSeason" class="w-full px-3 py-2 border rounded-lg">
                </div>
                <div>
                    <label for="growthDuration" class="block text-gray-700 font-bold mb-1">வளர்ச்சி காலம்:</label>
                    <input type="text" id="growthDuration" name="growthDuration" class="w-full px-3 py-2 border rounded-lg">
                </div>
            </div>
        </fieldset>

        <!-- 2. வளர்ச்சி நிலைகள் (டைனமிக்) -->
        <fieldset class="border p-4 rounded-lg mb-6">
            <legend class="text-xl font-semibold px-2">வளர்ச்சி நிலைகள்</legend>
            <div id="growth-stages-container" class="space-y-4"></div>
            <button type="button" id="add-stage-btn" class="mt-4 bg-green-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-green-600">
                + புதிய நிலையைச் சேர்
            </button>
        </fieldset>
        
        <!-- 3. நோய் மேலாண்மை (டைனமிக்) -->
        <fieldset class="border p-4 rounded-lg mb-6">
            <legend class="text-xl font-semibold px-2">நோய் மேலாண்மை</legend>
            <div id="diseases-container" class="space-y-4"></div>
            <button type="button" id="add-disease-btn" class="mt-4 bg-green-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-green-600">
                + புதிய நோயைச் சேர்
            </button>
        </fieldset>

        <!-- 4. உரப் பரிந்துரை (டைனமிக்) -->
        <fieldset class="border p-4 rounded-lg">
            <legend class="text-xl font-semibold px-2">உரப் பரிந்துரை</legend>
            <!-- பொதுவான உரங்கள் -->
            <div class="mb-4">
                <h3 class="font-semibold text-gray-700">பொதுவான உரங்கள்</h3>
                <div id="general-fertilizers-container" class="space-y-3 mt-2"></div>
                <button type="button" id="add-general-fertilizer-btn" class="mt-2 bg-gray-500 text-white font-bold py-1 px-3 text-sm rounded-lg hover:bg-gray-600">
                    + பொதுவான உரம்
                </button>
            </div>
             <!-- இயற்கை உரங்கள் -->
            <div>
                <h3 class="font-semibold text-gray-700">இயற்கை உரங்கள்</h3>
                <div id="organic-fertilizers-container" class="space-y-3 mt-2"></div>
                 <button type="button" id="add-organic-fertilizer-btn" class="mt-2 bg-gray-500 text-white font-bold py-1 px-3 text-sm rounded-lg hover:bg-gray-600">
                    + இயற்கை உரம்
                </button>
            </div>
        </fieldset>

        <div class="text-center mt-8">
            <button type="submit" class="bg-blue-700 text-white font-bold py-3 px-8 rounded-lg text-lg hover:bg-blue-800">
                பயிரைச் சேமி
            </button>
        </div>
    </form>
    <div id="form-message" class="mt-4 text-center"></div>
</div>

<script src="/js/admin.js"></script>
```

---

### படி 2: JavaScript-ஐ அனைத்துப் பிரிவுகளுக்கும் விரிவாக்குதல் (`public/js/admin.js`)

இப்போது, `admin.js` கோப்பில் வளர்ச்சி நிலைகள் மற்றும் உரங்களுக்கான UI மற்றும் தரவு சேகரிப்பு தர்க்கத்தைச் சேர்ப்போம்.

**`public/js/admin.js`-ன் முழுமையான, புதுப்பிக்கப்பட்ட குறியீடு:**

```javascript
document.addEventListener('DOMContentLoaded', () => {
    // DOM கூறுகளைப் பெறுதல்
    const form = document.getElementById('add-crop-form');
    const messageDiv = document.getElementById('form-message');

    // வளர்ச்சி நிலைகள்
    const addStageBtn = document.getElementById('add-stage-btn');
    const stagesContainer = document.getElementById('growth-stages-container');
    let stageCounter = 0;

    // நோய்கள்
    const addDiseaseBtn = document.getElementById('add-disease-btn');
    const diseasesContainer = document.getElementById('diseases-container');
    let diseaseCounter = 0;
    
    // உரங்கள்
    const addGeneralFertilizerBtn = document.getElementById('add-general-fertilizer-btn');
    const generalFertilizersContainer = document.getElementById('general-fertilizers-container');
    const addOrganicFertilizerBtn = document.getElementById('add-organic-fertilizer-btn');
    const organicFertilizersContainer = document.getElementById('organic-fertilizers-container');

    // --- UI கையாளுதல் ---

    // 1. புதிய வளர்ச்சி நிலையைச் சேர்
    addStageBtn.addEventListener('click', () => {
        stageCounter++;
        const stageHTML = `
            <div class="dynamic-entry grid grid-cols-1 md:grid-cols-7 gap-2 items-center" data-type="stage">
                <span class="md:col-span-1 font-semibold">நிலை #${stageCounter}</span>
                <input type="text" name="stageName" placeholder="நிலையின் பெயர்" class="md:col-span-3 p-2 border rounded">
                <input type="text" name="durationDays" placeholder="கால அளவு (எ.கா. 0-25 நாட்கள்)" class="md:col-span-2 p-2 border rounded">
                <button type="button" class="remove-btn md:col-span-1 text-red-500 font-bold justify-self-center">X</button>
            </div>
        `;
        stagesContainer.insertAdjacentHTML('beforeend', stageHTML);
    });

    // 2. புதிய நோயைச் சேர் (ஏற்கனவே உள்ளது)
    addDiseaseBtn.addEventListener('click', () => {
        diseaseCounter++;
        // ... (முந்தைய படியில் உள்ள அதே HTML குறியீடு) ...
    });

    // 3. புதிய பொதுவான உரத்தைச் சேர்
    addGeneralFertilizerBtn.addEventListener('click', () => {
        const html = `
            <div class="dynamic-entry grid grid-cols-1 md:grid-cols-7 gap-2 items-center" data-type="general-fertilizer">
                 <span class="md:col-span-1"></span>
                <input type="text" name="generalFertilizerName" placeholder="உரத்தின் பெயர் (எ.கா. யூரியா)" class="md:col-span-3 p-2 border rounded">
                <input type="text" name="generalFertilizerQty" placeholder="அளவு/ஏக்கர் (எ.கா. 16kg)" class="md:col-span-2 p-2 border rounded">
                <button type="button" class="remove-btn md:col-span-1 text-red-500 font-bold justify-self-center">X</button>
            </div>
        `;
        generalFertilizersContainer.insertAdjacentHTML('beforeend', html);
    });
    
    // 4. புதிய இயற்கை உரத்தைச் சேர்
    addOrganicFertilizerBtn.addEventListener('click', () => {
         const html = `
            <div class="dynamic-entry grid grid-cols-1 md:grid-cols-7 gap-2 items-center" data-type="organic-fertilizer">
                 <span class="md:col-span-1"></span>
                <input type="text" name="organicFertilizerName" placeholder="பெயர் (எ.கா. ஜீவாமிர்தம்)" class="md:col-span-3 p-2 border rounded">
                <input type="text" name="organicFertilizerQty" placeholder="அளவு/ஏக்கர் (எ.கா. 20 லிட்டர்)" class="md:col-span-2 p-2 border rounded">
                <button type="button" class="remove-btn md:col-span-1 text-red-500 font-bold justify-self-center">X</button>
            </div>
        `;
        organicFertilizersContainer.insertAdjacentHTML('beforeend', html);
    });

    // பொதுவான நீக்கும் செயல்பாடு
    form.addEventListener('click', (event) => {
        if (event.target.classList.contains('remove-btn')) {
            event.target.closest('.dynamic-entry').remove();
        }
    });


    // --- படிவ சமர்ப்பிப்பைக் கையாளுதல் ---
    form.addEventListener('submit', async (event) => {
        event.preventDefault();
        
        // 1. API அமைப்புக்கு ஏற்றவாறு தரவுப் பொருளை உருவாக்கு
        const data = {
            name: form.querySelector('[name="name"]').value,
            sowingSeason: form.querySelector('[name="sowingSeason"]').value,
            growthDuration: form.querySelector('[name="growthDuration"]').value,
            growthStages: [],
            diseases: [],
            fertilizerRecommendation: {
                general: [],
                organic: []
            },
            marketPriceHistory: [] // இப்போதைக்கு காலி
        };

        // 2. வளர்ச்சி நிலைகளைச் சேகரி
        stagesContainer.querySelectorAll('[data-type="stage"]').forEach(entry => {
            const stageName = entry.querySelector('[name="stageName"]').value;
            if (stageName) {
                data.growthStages.push({
                    stageName: stageName,
                    durationDays: entry.querySelector('[name="durationDays"]').value
                });
            }
        });

        // 3. நோய்களைச் சேகரி
        // ... (முந்தைய படியில் உள்ள அதே தர்க்கம்) ...

        // 4. பொதுவான உரங்களைச் சேகரி
        generalFertilizersContainer.querySelectorAll('[data-type="general-fertilizer"]').forEach(entry => {
            const fertilizer = entry.querySelector('[name="generalFertilizerName"]').value;
            if(fertilizer) {
                data.fertilizerRecommendation.general.push({
                    fertilizer: fertilizer,
                    quantityPerAcre: entry.querySelector('[name="generalFertilizerQty"]').value
                });
            }
        });

        // 5. இயற்கை உரங்களைச் சேகரி
        organicFertilizersContainer.querySelectorAll('[data-type="organic-fertilizer"]').forEach(entry => {
            const name = entry.querySelector('[name="organicFertilizerName"]').value;
            if(name){
                data.fertilizerRecommendation.organic.push({
                    name: name,
                    quantityPerAcre: entry.querySelector('[name="organicFertilizerQty"]').value
                });
            }
        });

        // தரவை அனுப்புதல் (முந்தைய படியில் உள்ள அதே தர்க்கம்)
        // ... fetch API call ...
    });
});
```
**(குறிப்பு: நோய்கள் மற்றும் fetch API அழைப்புக்கான குறியீட்டுப் பகுதிகள் சுருக்கப்பட்டுள்ளன, ஏனெனில் அவை முந்தைய படியிலிருந்து மாறவில்லை.)**

**முக்கிய JavaScript மாற்றங்கள்:**
1.  **புதிய கூறுகள்:** வளர்ச்சி நிலைகள் மற்றும் உரங்களுக்கான பொத்தான்கள் மற்றும் கொள்கலன்கள் தேர்ந்தெடுக்கப்பட்டுள்ளன.
2.  **புதிய Event Listeners:** ஒவ்வொரு "Add" பொத்தானுக்கும் ஒரு listener சேர்க்கப்பட்டு, அது தொடர்புடைய HTML-ஐ உருவாக்குகிறது.
3.  **பொதுவான நீக்கும் செயல்பாடு:** `remove-disease-btn` என்பதற்குப் பதிலாக, ஒரு பொதுவான `remove-btn` மற்றும் `dynamic-entry` வகுப்பைப் பயன்படுத்துகிறோம். இப்போது ஒரே ஒரு event listener அனைத்து நீக்கும் பொத்தான்களையும் கையாளும்.
4.  **Submit Handler மேம்படுத்தப்பட்டது:**
    *   பிரதான `data` பொருள் இப்போது `growthStages` மற்றும் `fertilizerRecommendation` (அதற்குள் `general`, `organic`) ஆகிய புலங்களைக் கொண்டுள்ளது.
    *   நோய்களைச் சேகரித்தது போலவே, புதிய `forEach` சுழற்சிகள் மூலம் வளர்ச்சி நிலைகள் மற்றும் உரங்களின் தரவு சேகரிக்கப்பட்டு, சரியான JSON அமைப்புக்குள் வைக்கப்படுகிறது.

---

### சோதித்துப் பார்ப்போம்

1.  உங்கள் சேவையகத்தை மறுதொடக்கம் செய்யவும் (`node server.js`).
2.  `http://localhost:3000/admin/add` பக்கத்திற்குச் செல்லவும்.
3.  இப்போது நீங்கள் அனைத்துப் பிரிவுகளுக்கும் "+ சேர்" பொத்தான்களைக் காண்பீர்கள்.
4.  ஒவ்வொரு பிரிவிலும் சில மாதிரித் தரவுகளைச் சேர்க்கவும் (எ.கா., 2 வளர்ச்சி நிலைகள், 1 நோய், 1 பொதுவான உரம், 1 இயற்கை உரம்).
5.  படிவத்தைச் சமர்ப்பிக்கவும்.
6.  வெற்றி பெற்றால், முகப்புப் பக்கத்திற்குச் சென்று, நீங்கள் புதிதாக உருவாக்கிய பயிரின் விவரங்கள் பக்கத்தைப் பார்க்கவும். நீங்கள் உள்ளிட்ட அனைத்து டைனமிக் தரவுகளும் அந்தந்த பிரிவுகளில் சரியாகக் காட்டப்படுவதை உறுதிப்படுத்திக் கொள்ளுங்கள்.

இப்போது உங்கள் நிர்வாகப் பக்கம் மிகவும் சக்திவாய்ந்ததாக மாறியுள்ளது. இதன் மூலம் எந்தவொரு பயிரின் எவ்வளவு சிக்கலான தரவையும் எளிதாக உள்ளிட முடியும்.

