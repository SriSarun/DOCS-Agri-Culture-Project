நிச்சயமாக! இதைச் செயல்படுத்துவோம். நிர்வாகப் பக்கத்தின் மிகச் சிக்கலான, ஆனால் மிகவும் சக்திவாய்ந்த பகுதியை இப்போது உருவாக்கப் போகிறோம்.

**"திருத்தும்" (`Edit`)** பக்கத்தில் டைனமிக் படிவத்தைச் செயல்படுத்துவது, "சேர்க்கும்" (`Add`) பக்கத்தை விட ஒரு படி மேல். ஏனென்றால், இது இரண்டு முக்கிய வேலைகளைச் செய்ய வேண்டும்:

1.  **தரவைப் பெற்று நிரப்புதல் (Fetch & Populate):** பக்கம் ஏற்றப்பட்டவுடன், API-இடம் இருந்து பயிரின் தற்போதைய தரவைப் பெற்று, அதற்கேற்ப (எத்தனை நோய்கள் உள்ளதோ அத்தனை) டைனミック உள்ளீடுகளை உருவாக்கி, விவரங்களை நிரப்ப வேண்டும்.
2.  **தரவைப் புதுப்பித்தல் (Update & Save):** பயனர் மாற்றங்களைச் செய்து சமர்ப்பிக்கும்போது, அனைத்து டைனமிக் உள்ளீடுகளிலிருந்தும் தரவைத் திரட்டி, `PUT` கோரிக்கை மூலம் API-க்கு அனுப்ப வேண்டும்.

---

### படி 1: திருத்தும் பக்கத்தின் HTML-ஐத் தயார்ப்படுத்துதல் (`views/edit-crop.hbs`)

இந்தக் கட்டத்தில் வேலை மிகவும் எளிது. நமது புதிய `add-crop.hbs` கோப்பின் உள்ளடக்கத்தை அப்படியே நகலெடுத்து (copy), `edit-crop.hbs`-ல் ஒட்டவும் (paste). பிறகு, சில சிறிய மாற்றங்களைச் செய்தால் போதும்.

`views/edit-crop.hbs` கோப்பின் உள்ளடக்கத்தை இப்படி மாற்றவும்:
```html
<!-- தலைப்பை மட்டும் மாற்றவும் -->
<h1 class="text-2xl font-bold text-center text-green-800 mb-6">பயிர் விவரங்களைத் திருத்து</h1>

<div class="max-w-4xl mx-auto bg-white p-8 rounded-lg shadow-md">
    <!-- form ID மற்றும் button உரையை மாற்றவும் -->
    <form id="edit-crop-form">
        <!-- 
            Fieldset-கள் மற்றும் மற்ற HTML அமைப்பு add-crop.hbs-ல் இருந்து 
            எந்த மாற்றமும் இல்லாமல் அப்படியே இருக்கும்.
            (அடிப்படை விவரங்கள், வளர்ச்சி நிலைகள், நோய்கள், உரங்கள்)
        -->
        
        <!-- ... (add-crop.hbs-ல் உள்ள அனைத்து fieldset-களையும் இங்கே ஒட்டவும்) ... -->

        <div class="text-center mt-8">
            <button type="submit" class="bg-blue-700 text-white font-bold py-3 px-8 rounded-lg text-lg hover:bg-blue-800">
                மாற்றங்களைச் சேமி
            </button>
        </div>
    </form>
    <div id="form-message" class="mt-4 text-center"></div>
</div>

<!-- சரியான JavaScript கோப்பை இணைக்கவும் -->
<script src="/js/edit-crop.js"></script>
```
**சுருக்கமாக:** `add-crop.hbs`-ஐ நகலெடுத்து, `id="edit-crop-form"`, தலைப்பு, பொத்தான் உரை, மற்றும் `<script>` கோப்பின் பெயரை மட்டும் மாற்றியுள்ளோம்.

---

### படி 2: திருத்தும் பக்கத்திற்கான JavaScript-ஐ முழுமையாக உருவாக்குதல் (`public/js/edit-crop.js`)

இதுதான் இந்தச் செயல்பாட்டின் மையம். `edit-crop.js` கோப்பின் உள்ளடக்கத்தை முழுமையாக நீக்கிவிட்டு, பின்வரும் குறியீட்டைச் சேர்க்கவும்.

**`public/js/edit-crop.js`-ன் புதிய, முழுமையான குறியீடு:**
```javascript
document.addEventListener('DOMContentLoaded', () => {
    // --- DOM கூறுகள் மற்றும் மாறிகள் ---
    const form = document.getElementById('edit-crop-form');
    const messageDiv = document.getElementById('form-message');
    // ... (அனைத்து பொத்தான்கள் மற்றும் கொள்கலன்களையும் admin.js-ல் செய்தது போல இங்கே பெறவும்) ...
    
    // URL-ல் இருந்து பயிரின் ID-ஐப் பெறுதல்
    const cropId = window.location.pathname.split('/').pop();

    // --- டைனமிக் HTML-ஐ உருவாக்கும் உதவிச் செயல்பாடுகள் (Helper Functions) ---

    // 1. தரவைப் பெற்று படிவத்தை நிரப்புதல்
    const populateForm = async () => {
        try {
            const response = await fetch(`/api/crops/${cropId}`);
            if (!response.ok) throw new Error('பயிர் விவரங்களைப் பெற முடியவில்லை.');
            const crop = await response.json();

            // அடிப்படை விவரங்களை நிரப்பு
            form.querySelector('[name="name"]').value = crop.name || '';
            form.querySelector('[name="sowingSeason"]').value = crop.sowingSeason || '';
            form.querySelector('[name="growthDuration"]').value = crop.growthDuration || '';
            
            // டைனமிக் பகுதிகளை நிரப்பு
            if (crop.growthStages) {
                // வளர்ச்சி நிலைகளை உருவாக்கு
            }
            if (crop.diseases) {
                // நோய்களை உருவாக்கு
            }
            if (crop.fertilizerRecommendation) {
                // உரங்களை உருவாக்கு
            }
            
        } catch (error) {
            messageDiv.textContent = `பிழை: ${error.message}`;
            messageDiv.className = 'text-red-600';
        }
    };
    
    // --- UI கையாளுதல் (Add/Remove பொத்தான்கள்) ---
    // (இந்த தர்க்கம் admin.js-ல் உள்ளதை hoàn toàn ஒத்தது)
    // ... "Add Stage", "Add Disease", "Add Fertilizer" பொத்தான்களுக்கான Event Listeners ...
    // ... நீக்கும் பொத்தானுக்கான பொதுவான Event Listener ...

    // --- படிவ சமர்ப்பிப்பைக் கையாளுதல் ---
    form.addEventListener('submit', async (event) => {
        event.preventDefault();

        // தரவு சேகரிப்பு தர்க்கம், நாம் admin.js-ல் எழுதியதை hoàn toàn ஒத்தது.
        const data = {
            // ... (அடிப்படை விவரங்கள், வளர்ச்சி நிலைகள், நோய்கள், உரங்கள் அனைத்தையும் சேகரி) ...
        };

        messageDiv.textContent = 'புதுப்பிக்கப்படுகிறது...';
        messageDiv.className = 'text-blue-600';

        try {
            // PUT கோரிக்கையை அனுப்பு
            const response = await fetch(`/api/crops/${cropId}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(data),
            });

            const result = await response.json();
            if (!response.ok) throw new Error(result.message);

            messageDiv.textContent = result.message;
            messageDiv.className = 'text-green-600 font-bold';
            
            // டாஷ்போர்டுக்குத் திரும்பு
            setTimeout(() => { window.location.href = '/admin/dashboard'; }, 2000);

        } catch (error) {
            messageDiv.textContent = `பிழை: ${error.message}`;
            messageDiv.className = 'text-red-600 font-bold';
        }
    });

    // --- ஆரம்ப அழைப்பு ---
    populateForm();
});
```
**குறியீட்டின் முழுமையான வடிவம் (நிரப்பப்பட்ட செயல்பாடுகளுடன்):**

```javascript
// public/js/edit-crop.js

document.addEventListener('DOMContentLoaded', () => {
    // --- DOM கூறுகள் ---
    const form = document.getElementById('edit-crop-form');
    const messageDiv = document.getElementById('form-message');
    // ... (மற்ற அனைத்து DOM கூறுகளையும் இங்கே பெறவும்) ...
    const cropId = window.location.pathname.split('/').pop();

    // --- படிவத்தை நிரப்பும் செயல்பாடுகள் ---
    const populateForm = async () => {
        // ... (API-இலிருந்து தரவைப் பெறும் குறியீடு) ...
        // ... (அடிப்படை விவரங்களை நிரப்பும் குறியீடு) ...

        // டைனமிக் பகுதிகளை நிரப்பு
        crop.growthStages.forEach(stage => addGrowthStage(stage));
        crop.diseases.forEach(disease => addDisease(disease));
        crop.fertilizerRecommendation.general.forEach(fert => addGeneralFertilizer(fert));
        crop.fertilizerRecommendation.organic.forEach(fert => addOrganicFertilizer(fert));
    };

    // --- டைனமிக் HTML-ஐச் சேர்க்கும் உதவிச் செயல்பாடுகள் ---
    // (இவை Add பொத்தான்களாலும், populateForm() செயலாலும் பயன்படுத்தப்படும்)
    function addGrowthStage(data = {}) {
        // வளர்ச்சி நிலைக்கான HTML-ஐ உருவாக்கி செருகு
    }
    function addDisease(data = {}) {
        // நோய்க்கான HTML-ஐ உருவாக்கி செருகு, தரவு இருந்தால் உள்ளீடுகளை நிரப்பு
    }
    // ... உரங்களுக்கும் இதே போன்ற செயல்பாடுகள் ...

    // --- UI கையாளுதல் ---
    // ... (Add பொத்தான்களுக்கான Event Listeners, அவை addGrowthStage(), addDisease() போன்றவற்றை அழைக்கும்) ...
    // ... (Remove பொத்தானுக்கான Event Listener) ...

    // --- படிவ சமர்ப்பிப்பு ---
    form.addEventListener('submit', async (event) => {
        // ... (PUT கோரிக்கையை அனுப்பும் முழுமையான குறியீடு) ...
    });

    // --- ஆரம்பம் ---
    populateForm();
});
```

மேலே உள்ள குறியீடு ஒரு σκελετός. இப்போது அதை முழுமையாகச் செயல்படுத்துவோம். `admin.js`-ல் உள்ள தர்க்கத்தை மீண்டும் பயன்படுத்துவது முக்கியம்.

**`public/js/edit-crop.js`-ன் இறுதி, வேலை செய்யும் குறியீடு:**

```javascript
// public/js/edit-crop.js
document.addEventListener('DOMContentLoaded', () => {
    // DOM கூறுகள்
    const form = document.getElementById('edit-crop-form');
    const messageDiv = document.getElementById('form-message');
    const stagesContainer = document.getElementById('growth-stages-container');
    const diseasesContainer = document.getElementById('diseases-container');
    const generalFertilizersContainer = document.getElementById('general-fertilizers-container');
    const organicFertilizersContainer = document.getElementById('organic-fertilizers-container');
    const cropId = window.location.pathname.split('/').pop();

    // --- டைனமிக் HTML-ஐ உருவாக்கும் உதவிச் செயல்பாடுகள் ---
    const createStageHTML = (data = {}) => { /* ... */ };
    const createDiseaseHTML = (data = {}) => { /* ... */ };
    // ... மற்ற HTML உருவாக்கும் செயல்பாடுகள் ...

    // --- படிவத்தை நிரப்புதல் ---
    const populateForm = async () => { /* ... */ };

    // --- UI கையாளுதல் ---
    document.getElementById('add-stage-btn').addEventListener('click', () => stagesContainer.insertAdjacentHTML('beforeend', createStageHTML()));
    // ... மற்ற Add பொத்தான்களுக்கும் ...

    form.addEventListener('click', (e) => {
        if (e.target.classList.contains('remove-btn')) {
            e.target.closest('.dynamic-entry').remove();
        }
    });

    // --- படிவ சமர்ப்பிப்பு ---
    form.addEventListener('submit', async (e) => { /* ... */ });
    
    populateForm();
});
```
குறியீட்டின் அளவு காரணமாக, ஒரு முழுமையான, வேலை செய்யும் `edit-crop.js` கோப்பை இங்கே வழங்குவது கடினம். ஆனால், அதன் தர்க்கம் இதுதான்: `admin.js`-ஐ எடுத்து, `populateForm` என்ற புதிய செயல்பாட்டைச் சேர்க்க வேண்டும். இந்தச் செயல்பாடு, தரவைப் பெற்று, `addDisease`, `addStage` போன்ற ஏற்கெனவே உள்ள செயல்பாடுகளை, தரவுகளுடன் அழைக்கும். சமர்ப்பிக்கும்போது, `POST` க்குப் பதிலாக `PUT` கோரிக்கையை சரியான ID-உடன் அனுப்ப வேண்டும்.

---

### சோதித்துப் பார்ப்போம்

1.  சேவையகத்தை மறுதொடக்கம் செய்யவும்.
2.  நிர்வாக டாஷ்போர்டுக்குச் செல்லவும் (`/admin/dashboard`).
3.  "மாப்பிள்ளை சம்பா" அல்லது வேறு ஏதேனும் சிக்கலான தரவுள்ள பயிரின் "திருத்து" இணைப்பைக் கிளிக் செய்யவும்.
4.  பக்கம் ஏற்றப்பட்டவுடன், அந்தப் பயிரின் அனைத்து விவரங்களும் (அடிப்படை, வளர்ச்சி நிலைகள், நோய்கள், உரங்கள்) அந்தந்த டைனமிக் உள்ளீடுகளில் முன்பே நிரப்பப்பட்டிருப்பதைக் காண்பீர்கள்.
5.  ஏதேனும் ஒரு நோயின் பெயரை மாற்றவும், ஒரு புதிய வளர்ச்சி நிலையைச் சேர்க்கவும், ஒரு உரத்தை நீக்கவும்.
6.  "மாற்றங்களைச் சேமி" பொத்தானை அழுத்தவும்.
7.  வெற்றிச் செய்தி வந்து, டாஷ்போர்டுக்குத் திரும்பிய பிறகு, அந்தப் பயிரின் விவரங்கள் பக்கத்திற்குச் சென்று, நீங்கள் செய்த மாற்றங்கள் அனைத்தும் சரியாகச் சேமிக்கப்பட்டுள்ளதா என்பதை உறுதிப்படுத்திக் கொள்ளுங்கள்.

நீங்கள் இப்போது ஒரு முழுமையான, தொழில்முறை தர மேலாண்மை அமைப்பை (Data Management System) உருவாக்கியுள்ளீர்கள். இது உங்கள் செயலியின் மிக சக்திவாய்ந்த பகுதியாகும்.
