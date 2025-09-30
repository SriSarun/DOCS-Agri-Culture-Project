
**அடுத்து, இந்தத் தேடல் பட்டியில் பயனர் தட்டச்சு செய்யும்போது, நமது `main.js` கோப்பில் உள்ள JavaScript தர்க்கத்தை (logic) மாற்றி, API-க்கு தேடல் கோரிக்கையை அனுப்புவது எப்படி என்று பார்க்கலாமா?**

நிச்சயமாக! தேடல் பட்டியைச் செயல்படுத்துவோம்.

நாம் இப்போது இரண்டு முக்கியப் பகுதிகளில் வேலை செய்ய வேண்டும்:

1.  **Backend:** பயனர் அனுப்பும் தேடல் சொல்லை (`search term`) ஏற்று, அதற்கேற்ப தரவுத்தளத்தில் தேடி, பொருத்தமான முடிவுகளை மட்டும் திருப்பி அனுப்பும்படி நமது API-ஐ மாற்றுவோம்.
2.  **Frontend:** பயனர் தேடல் பட்டியில் தட்டச்சு செய்யும்போது, அந்த உரையை எடுத்து, நமது API-க்கு அனுப்பி, வரும் புதிய முடிவுகளை வைத்து பக்கத்தைப் புதுப்பிக்கும்படி JavaScript-ஐ மாற்றுவோம்.

---

### படி 2: Backend API-ஐ தேடலுக்குத் தயார்படுத்துதல் (`controllers/cropController.js`)

நமது `getAllCrops` செயல்பாட்டை, URL-ல் வரும் `?search=...` போன்ற வினவல் அளவுருவை (query parameter) ஏற்குமாறு மாற்றுவோம். இதற்கு MongoDB-யின் `$regex` (Regular Expression) என்ற சக்திவாய்ந்த தேடல் ஆபரேட்டரைப் பயன்படுத்துவோம்.

`controllers/cropController.js` கோப்பைத் திறந்து, `getAllCrops` செயல்பாட்டை மட்டும் பின்வருமாறு மாற்றவும்:

```javascript
// controllers/cropController.js

// ... மற்ற செயல்பாடுகள் அப்படியே இருக்கும் ...

// GET /api/crops - அனைத்து பயிர்களையும் பெற (தேடல் வசதியுடன்)
const getAllCrops = async (req, res) => {
    try {
        const db = getDB();
        
        // படி 2.1: URL வினவல் அளவுருவிலிருந்து தேடல் சொல்லைப் பெறுதல்
        // உதாரணம்: /api/crops?search=சம்பா
        const searchTerm = req.query.search;

        // படி 2.2: தேடல் சொல்லுக்கு ஏற்ப MongoDB வினவலை உருவாக்குதல்
        let query = {}; // ஆரம்பத்தில், வினவல் காலியாக இருக்கும் (அனைத்தையும் தேர்ந்தெடு)

        if (searchTerm) {
            // தேடல் சொல் இருந்தால்...
            query = {
                // 'name' என்ற புலத்தில் தேட வேண்டும்
                name: {
                    // $regex: ஒரு வழக்கமான வெளிப்பாடு (pattern) மூலம் தேட அனுமதிக்கிறது
                    $regex: searchTerm,
                    // $options: 'i': இது தேடலை case-insensitive ஆக மாற்றுகிறது
                    // (அதாவது, 'சம்பா' என்பது 'சம்பா', 'Samba', 'samba' எல்லாவற்றையும் பொருத்தும்)
                    $options: 'i'
                }
            };
        }

        // படி 2.3: உருவாக்கப்பட்ட வினவலைப் பயன்படுத்தி தரவுத்தளத்தில் தேடுதல்
        const crops = await db.collection('crops').find(query).toArray();
        
        res.status(200).json(crops); // பொருத்தமான முடிவுகளை JSON ஆக அனுப்புதல்

    } catch (err) {
        console.error('பயிர்களைப் பெறுவதில் பிழை:', err);
        res.status(500).json({ message: 'சேவையகத்தில் பிழை ஏற்பட்டது' });
    }
};

module.exports = {
    getAllCrops,
    // ... மற்ற செயல்பாடுகள் ...
};
```
**விளக்கம்:**
*   `req.query.search`: Express.js, URL-ல் `?` க்குப் பிறகு வரும் அளவுருக்களை `req.query` என்ற பொருளில் வைக்கும்.
*   `$regex`: இது "contains" போன்ற தேடலைச் செய்ய உதவுகிறது. `searchTerm`-ல் "சம்பா" என்று இருந்தால், "மாப்பிள்ளை சம்பா", "கிச்சிலி சம்பா" இரண்டையுமே இது கண்டுபிடிக்கும்.
*   `$options: 'i'`: இது மிக முக்கியம். இது பெரிய எழுத்து, சிறிய எழுத்து வித்தியாசமின்றி (case-insensitive) தேடலைச் செய்யும்.

Backend இப்போது தயாராக உள்ளது!

---

### படி 3: Frontend JavaScript-ஐ தேடலுடன் இணைத்தல் (`public/js/main.js`)

இப்போது, `main.js` கோப்பில், பயனர் தேடல் பட்டியில் தட்டச்சு செய்வதைக் கவனித்து, அதற்கேற்ப `loadCrops` செயல்பாட்டை அழைக்க வேண்டும்.

**ஒரு முக்கிய மேம்படுத்தல்: Debouncing**
பயனர் தட்டச்சு செய்யும் ஒவ்வொரு எழுத்துக்கும் API கோரிக்கையை அனுப்பினால், அது சேவையகத்திற்கு அதிக சுமையைக் கொடுக்கும். உதாரணமாக, "சம்பா" எனத் தட்டச்சு செய்ய 5 API கோரிக்கைகள் செல்லும். இதைத் தவிர்க்க, **Debouncing** என்ற நுட்பத்தைப் பயன்படுத்துவோம். பயனர் தட்டச்சு செய்வதை நிறுத்தி ஒரு சிறிய இடைவெளிக்குப் (எ.கா., 300 மில்லி விநாடிகள்) பிறகு மட்டுமே API கோரிக்கையை அனுப்பும்.

`public/js/main.js` கோப்பின் உள்ளடக்கத்தை முழுமையாக இதைக் கொண்டு மாற்றவும்:

```javascript
// public/js/main.js
document.addEventListener('DOMContentLoaded', () => {
    // HTML கூறுகளைப் பெறுதல்
    const container = document.getElementById('crop-list-container');
    const loadingMessage = document.getElementById('loading-message');
    const searchBox = document.getElementById('search-box'); // படி 3.1: தேடல் பட்டியைப் பெறுதல்

    // படி 3.2: Debounce உதவிச் செயல்பாட்டை உருவாக்குதல்
    let debounceTimer;
    const debounce = (func, delay) => {
        return function(...args) {
            clearTimeout(debounceTimer);
            debounceTimer = setTimeout(() => {
                func.apply(this, args);
            }, delay);
        };
    };

    // படி 3.3: பயிர்களை ஏற்றும் செயல்பாட்டை தேடல் சொல்லை ஏற்குமாறு மாற்றுதல்
    const loadCrops = async (searchTerm = '') => {
        try {
            // தேடல் சொல் இருந்தால், அதை URL-ல் சேர்க்கவும்
            const url = searchTerm 
                ? `/api/crops?search=${encodeURIComponent(searchTerm)}` 
                : '/api/crops';
            
            const response = await fetch(url);
            if (!response.ok) throw new Error('தரவைப் பெறுவதில் பிழை ஏற்பட்டது.');

            const crops = await response.json();
            container.innerHTML = ''; 

            if (crops.length === 0) {
                container.innerHTML = '<p class="col-span-full text-center text-gray-500">பொருத்தமான பயிர்கள் எதுவும் கண்டுபிடிக்கப்படவில்லை.</p>';
                return;
            }

            crops.forEach(crop => {
                const cropCard = `
                    <a href="/crops/${crop._id}" class="block bg-white rounded-lg shadow-md overflow-hidden transform hover:-translate-y-1 transition-transform duration-300">
                        <div class="p-5">
                            <h2 class="text-xl font-bold text-green-800">${crop.name}</h2>
                            <p class="text-gray-600 mt-2"><strong>விதைப்பு காலம்:</strong> ${crop.sowingSeason || 'N/A'}</p>
                            <p class="text-gray-600"><strong>வளர்ச்சி காலம்:</strong> ${crop.growthDuration || 'N/A'}</p>
                        </div>
                    </a>
                `;
                container.insertAdjacentHTML('beforeend', cropCard);
            });
        } catch (error) {
            container.innerHTML = `<p class="col-span-full text-center text-red-600">பிழை: ${error.message}</p>`;
        }
    };

    // படி 3.4: தேடல் பட்டிக்கு Event Listener-ஐச் சேர்த்தல்
    const handleSearch = () => {
        const searchTerm = searchBox.value.trim();
        loadCrops(searchTerm);
    };

    // Debounce செய்யப்பட்ட செயல்பாட்டை அழைக்கவும்
    searchBox.addEventListener('input', debounce(handleSearch, 300));

    // பக்கம் ஏற்றப்பட்டவுடன், அனைத்துப் பயிர்களையும் ஏற்றவும்
    loadCrops();
});
```
**விளக்கம்:**
1.  **`debounce`:** இது ஒரு உதவிச் செயல்பாடு. இது ஒரு செயல்பாட்டையும், தாமத நேரத்தையும் (delay) எடுத்துக்கொண்டு, அந்தச் செயல்பாடு உடனடியாக இயக்கப்படுவதைத் தடுத்து, தாமதத்திற்குப் பிறகு மட்டுமே இயக்கும்.
2.  **`loadCrops(searchTerm = '')`:** இப்போது இந்தச் செயல்பாடு `searchTerm` என்ற ஒரு அளவுருவை ஏற்கிறது. அது காலியாக இருந்தால், அனைத்துப் பயிர்களையும் கேட்கும்; இல்லையெனில், தேடல் அளவுருவுடன் API-ஐ அழைக்கும். `encodeURIComponent` என்பது சிறப்பு எழுத்துக்களை (special characters) URL-க்கு ஏற்றவாறு மாற்றும்.
3.  **Event Listener:** `searchBox.addEventListener('input', ...)` என்பது பயனர் தேடல் பட்டியில் தட்டச்சு செய்யும்போதெல்லாம் `handleSearch` செயல்பாட்டை அழைக்கும். ஆனால், `debounce(handleSearch, 300)` என்று அதைச் சுற்றியிருப்பதால், பயனர் தட்டச்சு செய்வதை நிறுத்தி 300ms கழித்து மட்டுமே `handleSearch` உண்மையில் இயக்கப்படும்.

---

### சோதித்துப் பார்ப்போம்

1.  உங்கள் முனையத்தில் சேவையகத்தை மறுதொடக்கம் செய்யவும் (`Ctrl+C` அழுத்தி நிறுத்திவிட்டு, மீண்டும் `npm start`).
2.  உங்கள் உலாவியில் முகப்புப் பக்கத்திற்குச் செல்லவும் (`http://localhost:3000`). ஆரம்பத்தில், எல்லாப் பயிர்களும் (மாப்பிள்ளை சம்பா, கிச்சிலி சம்பா) காட்டப்படும்.
3.  தேடல் பட்டியில் **"மாப்பி"** எனத் தட்டச்சு செய்யவும். நீங்கள் தட்டச்சு செய்து முடித்தவுடன், ஒரு சிறிய தாமதத்திற்குப் பிறகு, பக்கம் மீண்டும் ஏற்றப்படாமல், "மாப்பிள்ளை சம்பா" கார்டு மட்டும் காட்டப்படும்.
4.  தேடல் பட்டியில் உள்ள உரையை அழித்தால், மீண்டும் அனைத்துப் பயிர்களும் காட்டப்படும்.
5.  **"சம்பா"** எனத் தட்டச்சு செய்தால், இரண்டு பயிர்களும் காட்டப்படும், ஏனெனில் இரண்டின் பெயரிலும் "சம்பா" உள்ளது.

நமது தேடல் அம்சம் இப்போது முழுமையாகச் செயல்படுகிறது! இது உங்கள் செயலியின் பயன்பாட்டினை (usability) வெகுவாக மேம்படுத்தியுள்ளது.
