அடுத்து, பயிரின் வகை அல்லது விதைப்பு காலம் போன்றவற்றை வைத்து **வடிகட்டுதல் (Filtering)** அம்சத்தைச் சேர்க்கலாமா? (உதாரணமாக, "ஆடி மாசம்" பயிர்களை மட்டும் காட்டுவது).

நிச்சயமாக, தொடர்வோம்.

தேடல் வசதியுடன், **வடிகட்டுதல் (Filtering)** அம்சத்தையும் சேர்ப்பது, உங்கள் செயலியை ஒரு தரவு சார்ந்த கருவியாக (data-driven tool) மாற்றும். பயனர்கள் தங்களுக்குத் தேவையான தகவல்களை மிகத் துல்லியமாகக் கண்டுபிடிக்க இது உதவும்.

**திட்டம்:**

1.  **புதிய API வழித்தடம் (Backend):** வடிகட்டி விருப்பங்களைக் (filter options) காட்டுவதற்காக, தரவுத்தளத்தில் உள்ள பிரத்யேக (unique) "விதைப்பு காலங்கள்" மற்றும் "வளர்ச்சி காலங்கள்" ஆகியவற்றின் பட்டியலை வழங்கும் ஒரு புதிய API (`GET /api/crops/filters`) உருவாக்குவோம்.
2.  **UI மாற்றம் (Frontend):** முகப்புப் பக்கத்தில், தேடல் பட்டிக்கு அருகில் இரண்டு கீழிறங்கு மெனுக்களை (Dropdown Menus) சேர்ப்போம். ఒకటి "விதைப்பு காலம்" மற்றொன்று "வளர்ச்சி காலம்" ஆகியவற்றிற்காக.
3.  **API மேம்படுத்தல் (Backend):** நமது `getAllCrops` செயல்பாட்டை, `?season=...` மற்றும் `?duration=...` போன்ற புதிய வினவல் அளவுருக்களை ஏற்குமாறு மேலும் மேம்படுத்துவோம்.
4.  **JavaScript தர்க்கம் (Frontend):** பயனர் ஒரு வடிப்பானைத் (filter) தேர்ந்தெடுக்கும்போது, JavaScript ஆனது தேடல் பட்டி மற்றும் அனைத்து வடிப்பான்களின் தற்போதைய மதிப்புகளையும் சேகரித்து, API-க்கு ஒரு புதிய கோரிக்கையை அனுப்பி, பக்கத்தைப் புதுப்பிக்கும்.

---

### படி 1: வடிகட்டி விருப்பங்களுக்கான புதிய API-ஐ உருவாக்குதல்

முதலில், கீழிறங்கு மெனுக்களில் காட்டுவதற்கான விருப்பங்களை (options) தரவுத்தளத்திலிருந்து பெறுவோம். MongoDB-யின் `distinct()` முறை இதற்கு மிகவும் பயனுள்ளதாக இருக்கும்.

#### 1. Controller-ல் புதிய செயல்பாட்டைச் சேர்த்தல் (`controllers/cropController.js`)

`cropController.js`-ல், `getAllCrops`-க்குக் கீழே இந்த புதிய செயல்பாட்டைச் சேர்க்கவும்:

```javascript
// controllers/cropController.js

// ...

// GET /api/crops/filters - வடிகட்டி விருப்பங்களைப் பெற
const getFilterOptions = async (req, res) => {
    try {
        const db = getDB();
        
        // Promise.all ஐப் பயன்படுத்தி இரண்டு தனித்தனி வினவல்களை ஒரே நேரத்தில் இயக்கவும்
        const [seasons, durations] = await Promise.all([
            db.collection('crops').distinct('sowingSeason'),
            db.collection('crops').distinct('growthDuration')
        ]);

        res.status(200).json({ seasons, durations });

    } catch (err) {
        console.error('வடிகட்டி விருப்பங்களைப் பெறுவதில் பிழை:', err);
        res.status(500).json({ message: 'சேவையகத்தில் பிழை ஏற்பட்டது' });
    }
};

// புதிய செயல்பாட்டையும் ஏற்றுமதி செய்யவும்
module.exports = {
    getAllCrops,
    getFilterOptions, // இதைச் சேர்க்கவும்
    // ... மற்ற செயல்பாடுகள் ...
};
```

#### 2. Router-ல் புதிய வழித்தடத்தைச் சேர்த்தல் (`routes/cropRoutes.js`)

`routes/cropRoutes.js`-ல், இந்த புதிய வழித்தடத்தை இணைக்கவும்:

```javascript
// routes/cropRoutes.js
// ...
const { 
    getAllCrops, 
    getFilterOptions, // இதை இறக்குமதி செய்யவும்
    // ...
} = require('../controllers/cropController');

// ...
router.get('/crops', getAllCrops);
router.get('/crops/filters', getFilterOptions); // புதிய வழித்தடத்தைச் சேர்க்கவும்
// ...
```

---

### படி 2: UI-ல் வடிகட்டி கீழிறங்கு மெனுக்களைச் சேர்த்தல் (`views/home.hbs`)

`views/home.hbs`-ல், தேடல் பட்டிக்குக் கீழே, வடிப்பான்களுக்கான ஒரு புதிய பகுதியைச் சேர்ப்போம்.

```html
<!-- ... தேடல் பட்டிக்குப் பிறகு ... -->
<!-- படி 2: வடிகட்டி கீழிறங்கு மெனுக்களைச் சேர்த்தல் -->
<div class="flex flex-col sm:flex-row justify-center items-center gap-4 mb-8">
    <select id="season-filter" class="filter-select w-full sm:w-auto px-4 py-2 border border-gray-300 rounded-md bg-white focus:outline-none focus:ring-2 focus:ring-green-500">
        <option value="">அனைத்து விதைப்பு காலங்கள்</option>
        <!-- JS மூலம் விருப்பங்கள் இங்கே சேர்க்கப்படும் -->
    </select>
    <select id="duration-filter" class="filter-select w-full sm:w-auto px-4 py-2 border border-gray-300 rounded-md bg-white focus:outline-none focus:ring-2 focus:ring-green-500">
        <option value="">அனைத்து வளர்ச்சி காலங்கள்</option>
         <!-- JS மூலம் விருப்பங்கள் இங்கே சேர்க்கப்படும் -->
    </select>
</div>

<!-- பயிர் கார்டுகளுக்கான கொள்கலன் ... -->
```
*   `value=""` என்பது "அனைத்தையும் காட்டு" என்பதைக் குறிக்கிறது.

---

### படி 3: Backend API-ஐ வடிகட்டலுக்குத் தயார்படுத்துதல் (`controllers/cropController.js`)

`getAllCrops` செயல்பாட்டை மீண்டும் மாற்றி, `season` மற்றும் `duration` அளவுருக்களையும் ஏற்குமாறு செய்வோம்.

```javascript
// controllers/cropController.js
const getAllCrops = async (req, res) => {
    try {
        const db = getDB();
        const { search, season, duration } = req.query;

        let query = {};

        // தேடல் வினவலைச் சேர்
        if (search) {
            query.name = { $regex: search, $options: 'i' };
        }
        
        // விதைப்பு கால வடிகட்டியைச் சேர்
        if (season) {
            query.sowingSeason = season;
        }

        // வளர்ச்சி கால வடிகட்டியைச் சேர்
        if (duration) {
            query.growthDuration = duration;
        }
        
        const crops = await db.collection('crops').find(query).toArray();
        res.status(200).json(crops);

    } catch (err) {
        // ... பிழை கையாளுதல் ...
    }
};
```
*   MongoDB-ல், `query` பொருளில் பல பண்புகளைச் சேர்ப்பது, அவற்றை `AND` தர்க்கத்துடன் இணைப்பதற்குச் சமம். (பெயர் **மற்றும்** விதைப்பு காலம் **மற்றும்** வளர்ச்சி காலம்).

---

### படி 4: Frontend JavaScript-ஐ முழுமையாக ஒருங்கிணைத்தல் (`public/js/main.js`)

இது இறுதிப் படி. `main.js` கோப்பை, கீழிறங்கு மெனுக்களை நிரப்பவும், அவற்றின் மாற்றங்களைக் கேட்கவும், அனைத்து வடிப்பான்களையும் ஒன்றாக இணைத்து API-க்கு அனுப்பவும் மாற்றுவோம்.

`public/js/main.js`-ன் முழுமையான, புதுப்பிக்கப்பட்ட குறியீடு:

```javascript
// public/js/main.js
document.addEventListener('DOMContentLoaded', () => {
    // DOM கூறுகள்
    const container = document.getElementById('crop-list-container');
    const searchBox = document.getElementById('search-box');
    const seasonFilter = document.getElementById('season-filter');
    const durationFilter = document.getElementById('duration-filter');

    // Debounce உதவிச் செயல்பாடு
    let debounceTimer;
    const debounce = (func, delay) => { /* ... (முந்தைய படியில் உள்ள அதே குறியீடு) ... */ };

    // படி 4.1: வடிகட்டி விருப்பங்களை நிரப்பும் செயல்பாடு
    const populateFilters = async () => {
        try {
            const response = await fetch('/api/crops/filters');
            const options = await response.json();

            options.seasons.forEach(season => {
                const option = document.createElement('option');
                option.value = season;
                option.textContent = season;
                seasonFilter.appendChild(option);
            });
            options.durations.forEach(duration => {
                const option = document.createElement('option');
                option.value = duration;
                option.textContent = duration;
                durationFilter.appendChild(option);
            });
        } catch (error) {
            console.error('வடிகட்டி விருப்பங்களை ஏற்றுவதில் பிழை:', error);
        }
    };

    // படி 4.2: பயிர்களை ஏற்றும் செயல்பாட்டை URLSearchParams ஐப் பயன்படுத்த மாற்றுதல்
    const loadCrops = async () => {
        // அனைத்து வடிப்பான்களின் தற்போதைய மதிப்புகளையும் சேகரி
        const params = new URLSearchParams();
        if (searchBox.value) params.append('search', searchBox.value);
        if (seasonFilter.value) params.append('season', seasonFilter.value);
        if (durationFilter.value) params.append('duration', durationFilter.value);
        
        const queryString = params.toString();
        const url = `/api/crops?${queryString}`;
        
        try {
            const response = await fetch(url);
            // ... (மீதமுள்ள `loadCrops` தர்க்கம் முந்தைய படியில் உள்ளதைப் போலவே இருக்கும்) ...
             const crops = await response.json();
             container.innerHTML = '';
             // ... (கார்டுகளை உருவாக்கும் forEach சுழற்சி) ...
        } catch (error) {
            // ... (பிழை கையாளுதல்) ...
        }
    };

    // படி 4.3: அனைத்து உள்ளீடுகளுக்கும் Event Listeners
    const debouncedLoadCrops = debounce(loadCrops, 400);

    searchBox.addEventListener('input', debouncedLoadCrops);
    seasonFilter.addEventListener('change', loadCrops); // கீழிறங்கு மெனுக்களுக்கு debounce தேவையில்லை
    durationFilter.addEventListener('change', loadCrops);

    // படி 4.4: ஆரம்பத்தில் பக்கத்தை ஏற்றுதல்
    populateFilters(); // முதலில் வடிப்பான்களை நிரப்பு
    loadCrops();       // பின்னர் அனைத்துப் பயிர்களையும் காட்டு
});
```

### சோதித்துப் பார்ப்போம்

1.  உங்கள் சேவையகத்தை மறுதொடக்கம் செய்யவும் (`npm start`).
2.  முகப்புப் பக்கத்திற்குச் செல்லவும் (`http://localhost:3000`).
3.  "விதைப்பு காலம்" மற்றும் "வளர்ச்சி காலம்" கீழிறங்கு மெனுக்கள், உங்கள் தரவுத்தளத்தில் உள்ள பிரத்யேக மதிப்புகளுடன் நிரப்பப்பட்டிருப்பதை நீங்கள் காண்பீர்கள்.
4.  "விதைப்பு காலம்" கீழிறங்கு மெனுவில், "ஆடி மாசம் (July-August)" என்பதைத் தேர்ந்தெடுக்கவும். பக்கம் மீண்டும் ஏற்றப்படாமல், "மாப்பிள்ளை சம்பா" மட்டும் காட்டப்படும்.
5.  இப்போது, தேடல் பட்டியில் "மாப்பி" எனத் தட்டச்சு செய்யவும். வடிகட்டி இன்னும் செயலில் இருப்பதால், "மாப்பிள்ளை சம்பா" மட்டுமே காட்டப்படும்.
6.  தேடல் பட்டியை அழித்து, வடிப்பான்களை "அனைத்து..." என்பதற்கு மாற்றினால், மீண்டும் அனைத்துப் பயிர்களும் காட்டப்படும்.

வாழ்த்துக்கள்! நீங்கள் இப்போது ஒரு சக்திவாய்ந்த, ஊடாடும் தேடல் மற்றும் வடிகட்டுதல் அமைப்பை வெற்றிகரமாக உருவாக்கியுள்ளீர்கள். ఇది உங்கள் செயலியை பயனர்களுக்கு மிகவும் பயனுள்ளதாக மாற்றும்.