**USER**

சூப்பர் 👌
"ஆம், அடுத்த செயலையை நாம் தொடர்வோம்."

**RESPONSE**
நிச்சயமாக! அடுத்த கட்டத்திற்குச் செல்லலாம்.

நாம் சேமித்த பயிர் விவரங்களை, முகப்புப் பக்கத்தில் பயனர்களுக்குக் காட்டுவோம். இதையும் AJAX முறையிலேயே செய்வோம். பக்கம் ஏற்றப்பட்டவுடன், JavaScript நமது `GET /api/crops` API-ஐ அழைத்து, கிடைக்கும் தரவுகளை வைத்து கார்டுகளை (Cards) உருவாக்கும்.

### படி 3: முகப்புப் பக்கத்தில் பயிர்களை AJAX மூலம் காட்டுதல்

#### 1. முகப்புப் பக்கத்தை தயார் செய்தல் (`views/home.hbs`)

முதலில், `home.hbs` கோப்பில், பயிர் கார்டுகளைக் காண்பிப்பதற்காக ஒரு கொள்கலன் (container) `div`-ஐ உருவாக்குவோம். மேலும், தரவு ஏற்றப்படும் வரை ஒரு "Loading..." செய்தியைக் காட்டலாம்.

`views/home.hbs` கோப்பின் உள்ளடக்கத்தை முழுமையாக மாற்றவும்:

```html
<h1 class="text-3xl font-bold text-center text-green-800 mb-8">பயிர்கள் வழிகாட்டி</h1>

<!-- பயிர் கார்டுகளைக் காண்பிப்பதற்கான கொள்கலன் -->
<div id="crop-list-container" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
    <!-- தரவு ஏற்றப்படும்போது இது காட்டப்படும். JavaScript இதை அகற்றிவிடும். -->
    <p id="loading-message" class="col-span-full text-center text-gray-500">பயிர்களை ஏற்றுகிறது...</p>
</div>

<!-- முகப்புப் பக்கத்திற்கான பிரத்யேக JavaScript கோப்பை இணைக்கவும் -->
<script src="/js/main.js"></script>
```
**விளக்கம்:**
*   `crop-list-container`: இது ஒரு காலி `div`. நமது AJAX கோரிக்கை வெற்றிகரமாக முடிந்ததும், இந்த `div`-க்குள் ஒவ்வொரு பயிருக்கும் ஒரு கார்டை JavaScript உருவாக்கும்.
*   `grid`, `grid-cols-*`, `gap-6`: இவை Tailwind CSS வகுப்புகள். இது கார்டுகளை ஒரு அழகான கட்டம் (grid) அமைப்பில் வெவ்வேறு திரை அளவுகளுக்கு ஏற்றவாறு காட்ட உதவும்.
*   `<script src="/js/main.js"></script>`: இந்தப் பக்கத்தின் AJAX செயல்பாடுகளைக் கையாள `main.js` என்ற புதிய கோப்பை இணைத்துள்ளோம்.

#### 2. AJAX மூலம் தரவைப் பெற்று காட்டுதல் (`public/js/main.js`)

இப்போது, `public/js/` கோப்புறையில் `main.js` என்ற புதிய கோப்பை உருவாக்கவும். இந்தக் கோப்புதான் நமது API-இடம் இருந்து தரவைப் பெற்று, அதை HTML ஆக மாற்றப் போகிறது.

`public/js/main.js` கோப்பில் இந்த குறியீட்டைச் சேர்க்கவும்:

```javascript
document.addEventListener('DOMContentLoaded', () => {
    // HTML கூறுகளைப் பெறுதல்
    const container = document.getElementById('crop-list-container');
    const loadingMessage = document.getElementById('loading-message');

    // பயிர்களை ஏற்றும் செயல்பாடு
    const loadCrops = async () => {
        try {
            // நமது backend API-க்கு GET கோரிக்கையை அனுப்புதல்
            const response = await fetch('/api/crops');

            if (!response.ok) {
                throw new Error('தரவைப் பெறுவதில் பிழை ஏற்பட்டது.');
            }

            // JSON பதிலை JavaScript வரிசையாக மாற்றுதல்
            const crops = await response.json();

            // ஏற்றுதல் செய்தியை அகற்றுதல்
            container.innerHTML = ''; 

            // தரவு இல்லை என்றால், ஒரு செய்தியைக் காட்டு
            if (crops.length === 0) {
                container.innerHTML = '<p class="col-span-full text-center text-gray-500">காண்பிக்க பயிர்கள் எதுவும் இல்லை.</p>';
                return;
            }

            // ஒவ்வொரு பயிருக்கும் ஒரு கார்டை உருவாக்குதல்
            crops.forEach(crop => {
                const cropCard = `
                    <div class="bg-white rounded-lg shadow-md overflow-hidden transform hover:-translate-y-1 transition-transform duration-300">
                        <div class="p-5">
                            <h2 class="text-xl font-bold text-green-800">${crop.name}</h2>
                            <p class="text-gray-600 mt-2"><strong>விதைப்பு காலம்:</strong> ${crop.sowingSeason}</p>
                            <p class="text-gray-600"><strong>வளர்ச்சி காலம்:</strong> ${crop.growthDuration}</p>
                        </div>
                    </div>
                `;
                // உருவாக்கப்பட்ட கார்டை கொள்கலனில் செருகுதல்
                container.insertAdjacentHTML('beforeend', cropCard);
            });

        } catch (error) {
            // பிழை ஏற்பட்டால், பிழை செய்தியைக் காட்டு
            loadingMessage.textContent = `பிழை: ${error.message}`;
            loadingMessage.className = 'col-span-full text-center text-red-600 font-bold';
        }
    };

    // பக்கம் ஏற்றப்பட்டவுடன், பயிர்களை ஏற்றும் செயல்பாட்டை அழைக்கவும்
    loadCrops();
});
```
**விளக்கம்:**
*   `DOMContentLoaded`: பக்கம் முழுமையாக ஏற்றப்பட்ட பின்னரே இந்த ஸ்கிரிப்ட் இயங்கும் என்பதை உறுதி செய்கிறது.
*   `fetch('/api/crops')`: இது நமது `GET /api/crops` வழித்தடத்தை அழைத்து, தரவுத்தளத்தில் உள்ள அனைத்து பயிர்களின் பட்டியலையும் கேட்கிறது.
*   `container.innerHTML = '';`: தரவு வருவதற்கு முன் இருந்த "Loading..." செய்தியை இது நீக்குகிறது.
*   `crops.forEach(...)`: API-இடம் இருந்து வந்த ஒவ்வொரு பயிர் பொருளுக்கும் (crop object), ஒரு HTML கார்டை `template literal` (``) பயன்படுத்தி உருவாக்குகிறோம்.
*   `container.insertAdjacentHTML(...)`: உருவாக்கப்பட்ட HTML கார்டை பக்கத்தில், பயனருக்குத் தெரியும்படி சேர்க்கிறது.

---

### இப்போது சோதித்துப் பார்ப்போம்

1.  உங்கள் முனையத்தில் `node server.js` கட்டளையை இயக்கி, சேவையகத்தைத் தொடங்கவும்.
2.  உங்கள் உலாவியில் முகப்புப் பக்கத்திற்குச் செல்லவும்: `http://localhost:3000`.
3.  நீங்கள் முதலில் "பயிர்களை ஏற்றுகிறது..." என்ற செய்தியைக் காண்பீர்கள். ஒரு நொடிக்குப் பிறகு, அது மறைந்து, நீங்கள் முன்பு சேர்த்த ஒவ்வொரு பயிருக்கும் அழகான கார்டுகள் தோன்றும்.
4.  நீங்கள் இன்னும் எந்தப் பயிரையும் சேர்க்கவில்லை என்றால், "காண்பிக்க பயிர்கள் எதுவும் இல்லை." என்ற செய்தி தோன்றும்.

இப்போது, நாம் ஒரு முழுமையான சுழற்சியை (full cycle) முடித்துவிட்டோம்:
*   **நிர்வாகி (Admin):** `/admin/add` பக்கத்தில் AJAX மூலம் ஒரு பயிரைச் சேர்க்கிறார்.
*   **பயனர் (User):** முகப்புப் பக்கத்தில் AJAX மூலம் அந்தப் பயிர்களின் பட்டியலைப் பார்க்கிறார்.

அனைத்தும் பக்கம் மீண்டும் ஏற்றப்படாமல் (without page reload) நடக்கிறது!


