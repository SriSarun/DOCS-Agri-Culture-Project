**நிர்வாக டாஷ்போர்டில் திருத்தும் பக்கத்தை (`/admin/edit/:id`) உருவாக்கி, ஒரு பயிரின் விவரங்களைப் புதுப்பிக்கும் செயல்பாட்டைத் தொடங்கலாமா?**

மிகவும் அருமை! நிர்வாகப் பக்கத்தின் இறுதி மற்றும் மிக முக்கியமான பகுதியான "திருத்தும்" (Edit) செயல்பாட்டை இப்போது உருவாக்குவோம்.

ஒரு பயனர் "திருத்து" பொத்தானை அழுத்தும்போது, அந்தப் பயிரின் தற்போதைய விவரங்கள் ஒரு படிவத்தில் முன்பே நிரப்பப்பட்டு, அதை அவர்கள் புதுப்பிக்க முடியும்.

### திட்டம்: மூன்று முக்கியப் படிகள்

1.  **திருத்தும் பக்கத்தை வழங்குதல் (Page Rendering):**
    *   `/admin/edit/:id` என்ற URL-க்கு ஒரு பயனர் செல்லும்போது, ஒரு திருத்தும் பக்கத்தைக் காட்ட `server.js`-ல் ஒரு புதிய வழித்தடத்தை உருவாக்குவோம்.

2.  **படிவத்தில் தரவை நிரப்புதல் (AJAX GET):**
    *   திருத்தும் பக்கம் ஏற்றப்பட்டவுடன், JavaScript ஆனது URL-ல் இருந்து பயிரின் ID-ஐப் பெற்று, நமது `GET /api/crops/:id` API-ஐப் பயன்படுத்தி அந்தப் பயிரின் விவரங்களைப் பெறும்.
    *   பெற்ற விவரங்களைக் கொண்டு படிவத்தின் உள்ளீடுகளை (input fields) நிரப்பும்.

3.  **படிவத்தைப் புதுப்பித்தல் (AJAX PUT):**
    *   பயனர் படிவத்தைச் சமர்ப்பிக்கும்போது, JavaScript ஆனது படிவத் தரவைப் பெற்று, நமது `PUT /api/crops/:id` API-க்கு அனுப்பி, தரவுத்தளத்தில் உள்ள விவரங்களைப் புதுப்பிக்கும்.

---

### படி 1: திருத்தும் பக்கத்தை உருவாக்குதல்

#### 1. திருத்தும் பக்கத்திற்கான வழித்தடத்தைச் சேர்த்தல் (`server.js`)

`server.js` கோப்பில், `admin/dashboard` வழித்தடத்திற்குப் பிறகு, இந்த புதிய வழித்தடத்தைச் சேர்க்கவும்:

```javascript
// ... server.js இல் Page Rendering Routes பகுதியில் ...

app.get('/admin/edit/:id', (req, res) => {
    // இந்த வழித்தடம் ஒரு எளிய HTML பக்கத்தை மட்டுமே வழங்கும்.
    // தரவை நிரப்பும் வேலையை frontend JavaScript பார்த்துக் கொள்ளும்.
    res.render('edit-crop', { title: 'பயிர் விவரங்களைத் திருத்து' });
});
```

#### 2. திருத்தும் பக்கத்திற்கான View-ஐ உருவாக்குதல் (`views/edit-crop.hbs`)

`add-crop.hbs`-ஐப் போலவே ஒரு புதிய கோப்பை `views` கோப்புறையில் `edit-crop.hbs` என்ற பெயரில் உருவாக்கவும். இது கிட்டத்தட்ட ஒரே மாதிரியாக இருக்கும், ஆனால் தலைப்பு மற்றும் பொத்தான் உரை மாற்றப்பட்டிருக்கும்.

```html
<h1 class="text-2xl font-bold text-center text-green-800 mb-6">பயிர் விவரங்களைத் திருத்து</h1>

<div class="max-w-2xl mx-auto bg-white p-8 rounded-lg shadow-md">
    <!-- படிவத்திற்கு ஒரு புதிய ID கொடுக்கப்பட்டுள்ளது -->
    <form id="edit-crop-form">
        <div class="mb-4">
            <label for="name" class="block text-gray-700 font-bold mb-2">பயிரின் பெயர்:</label>
            <!-- உள்ளீடுகள் αρχικά காலியாக இருக்கும் -->
            <input type="text" id="name" name="name" class="w-full px-3 py-2 border rounded-lg" required>
        </div>

        <div class="mb-4">
            <label for="sowingSeason" class="block text-gray-700 font-bold mb-2">விதைப்பு காலம்:</label>
            <input type="text" id="sowingSeason" name="sowingSeason" class="w-full px-3 py-2 border rounded-lg" required>
        </div>

        <div class="mb-6">
            <label for="growthDuration" class="block text-gray-700 font-bold mb-2">வளர்ச்சி காலம் (நாட்களில்):</label>
            <input type="text" id="growthDuration" name="growthDuration" class="w-full px-3 py-2 border rounded-lg" required>
        </div>

        <div class="text-center">
            <button type="submit" class="bg-blue-600 text-white font-bold py-2 px-6 rounded-lg hover:bg-blue-700 transition duration-200">
                புதுப்பி
            </button>
        </div>
    </form>
    <!-- செய்திகளைக் காட்டுவதற்கான இடம் -->
    <div id="form-message" class="mt-4 text-center"></div>
</div>

<!-- இந்தப் பக்கத்திற்கான பிரத்யேக JavaScript கோப்பு -->
<script src="/js/edit-crop.js"></script>```

---

### படி 2 & 3: AJAX மூலம் தரவை நிரப்புதல் மற்றும் புதுப்பித்தல்

#### 1. புதிய JavaScript கோப்பை உருவாக்குதல் (`public/js/edit-crop.js`)

`public/js` கோப்புறையில் `edit-crop.js` என்ற புதிய கோப்பை உருவாக்கவும். இதுதான் திருத்தும் பக்கத்தின் முழு செயல்பாட்டையும் கையாளும்.

```javascript
document.addEventListener('DOMContentLoaded', () => {
    // HTML கூறுகளைப் பெறுதல்
    const form = document.getElementById('edit-crop-form');
    const nameInput = document.getElementById('name');
    const sowingSeasonInput = document.getElementById('sowingSeason');
    const growthDurationInput = document.getElementById('growthDuration');
    const messageDiv = document.getElementById('form-message');

    // 1. URL-ல் இருந்து பயிரின் ID-ஐப் பெறுதல்
    const pathParts = window.location.pathname.split('/');
    const cropId = pathParts[pathParts.length - 1];

    // 2. படிவத்தில் தரவை நிரப்பும் செயல்பாடு
    const fetchAndPopulateForm = async () => {
        try {
            const response = await fetch(`/api/crops/${cropId}`);
            if (!response.ok) {
                throw new Error('பயிர் விவரங்களைப் பெற முடியவில்லை.');
            }
            const crop = await response.json();

            // பெற்ற தரவை வைத்து படிவத்தை நிரப்புதல்
            nameInput.value = crop.name;
            sowingSeasonInput.value = crop.sowingSeason;
            growthDurationInput.value = crop.growthDuration;

        } catch (error) {
            messageDiv.textContent = `பிழை: ${error.message}`;
            messageDiv.className = 'text-red-600 font-bold';
        }
    };

    // 3. படிவச் சமர்ப்பிப்பைக் கையாளும் செயல்பாடு
    form.addEventListener('submit', async (event) => {
        event.preventDefault(); // பக்கம் reload ஆவதைத் தடுக்கவும்

        // படிவத்திலிருந்து புதுப்பிக்கப்பட்ட தரவைப் பெறுதல்
        const updatedData = {
            name: nameInput.value,
            sowingSeason: sowingSeasonInput.value,
            growthDuration: growthDurationInput.value,
        };

        messageDiv.textContent = 'புதுப்பிக்கப்படுகிறது...';
        messageDiv.className = 'text-blue-600';

        try {
            // PUT கோரிக்கையை API-க்கு அனுப்புதல்
            const response = await fetch(`/api/crops/${cropId}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(updatedData),
            });

            const result = await response.json();

            if (!response.ok) {
                throw new Error(result.message);
            }
            
            // வெற்றி பெற்றால்
            messageDiv.textContent = result.message;
            messageDiv.className = 'text-green-600 font-bold';

            // 2 வினாடிகளுக்குப் பிறகு டாஷ்போர்டுக்குத் திரும்பு
            setTimeout(() => {
                window.location.href = '/admin/dashboard';
            }, 2000);

        } catch (error) {
            messageDiv.textContent = `பிழை: ${error.message}`;
            messageDiv.className = 'text-red-600 font-bold';
        }
    });

    // பக்கம் ஏற்றப்பட்டவுடன், தரவைப் பெற்று படிவத்தை நிரப்பவும்
    fetchAndPopulateForm();
});
```
**விளக்கம்:**
*   **ID-ஐப் பெறுதல்:** `window.location.pathname` மூலம் தற்போதைய URL (`/admin/edit/635a...`) பெறப்பட்டு, `.split('/')` மூலம் பிரிக்கப்பட்டு, கடைசிப் பகுதியான ID எடுக்கப்படுகிறது.
*   **`fetchAndPopulateForm`:** இந்த செயல்பாடு `GET /api/crops/:id`-ஐ அழைத்து, கிடைத்த பதிலைக் கொண்டு `nameInput.value`, `sowingSeasonInput.value` போன்றவற்றை நிரப்புகிறது.
*   **Submit Listener:**
    *   இது `PUT` கோரிக்கையை நமது API-க்கு அனுப்புகிறது. `body` பகுதியில் புதுப்பிக்கப்பட்ட தரவு JSON ஆக அனுப்பப்படுகிறது.
    *   வெற்றி பெற்றால், ஒரு செய்தியைக் காட்டிவிட்டு, `setTimeout` மூலம் 2 வினாடிகள் கழித்து, பயனரை மீண்டும் டாஷ்போர்டு பக்கத்திற்கு (`/admin/dashboard`) அழைத்துச் செல்கிறது. இது பயனர் வெற்றிச் செய்தியைப் படிக்க நேரம் கொடுக்கும்.

---

### சோதித்துப் பார்ப்போம்!

1.  சேவையகத்தை `node server.js` மூலம் இயக்கவும்.
2.  முதலில், நிர்வாக டாஷ்போர்டுக்குச் செல்லவும்: `http://localhost:3000/admin/dashboard`.
3.  ஏதேனும் ஒரு பயிரின் வரிசையில் உள்ள "திருத்து" இணைப்பைக் கிளிக் செய்யவும்.
4.  நீங்கள் `/admin/edit/<அந்த பயிரின் ID>` என்ற URL உடன் புதிய பக்கத்திற்கு அழைத்துச் செல்லப்படுவீர்கள்.
5.  அந்தப் பக்கத்தில் உள்ள படிவம், நீங்கள் தேர்ந்தெடுத்த பயிரின் விவரங்களுடன் முன்பே நிரப்பப்பட்டிருப்பதைக் காண்பீர்கள்.
6.  ஏதேனும் ஒரு விவரத்தை மாற்றி, "புதுப்பி" பொத்தானை அழுத்தவும்.
7.  "பயிர் வெற்றிகரமாகப் புதுப்பிக்கப்பட்டது" என்ற செய்தி தோன்றி, 2 வினாடிகளுக்குப் பிறகு, நீங்கள் தானாகவே டாஷ்போர்டு பக்கத்திற்குத் திருப்பி அனுப்பப்படுவீர்கள்.
8.  டாஷ்போர்டில், நீங்கள் செய்த மாற்றம் புதுப்பிக்கப்பட்டிருப்பதை உறுதிப்படுத்திக் கொள்ளலாம்.

**வாழ்த்துக்கள்!** நீங்கள் இப்போது ஒரு முழுமையான CRUD (Create, Read, Update, Delete) செயல்பாடுகளைக் கொண்ட, API-சார்ந்த நிர்வாகப் பக்கத்தை வெற்றிகரமாக உருவாக்கியுள்ளீர்கள்.

இது ஒரு வலுவான அடித்தளம். இதன் மீது தேடல் (Search), வடிகட்டுதல் (Filtering), மற்றும் உங்கள் எதிர்கால AI அம்சங்களை எளிதாக உருவாக்க முடியும்.
