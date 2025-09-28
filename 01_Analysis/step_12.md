அடுத்ததாக, இதே போன்ற டைனமிக் தர்க்கத்தை **"திருத்தும்" (`Edit Crop`)** பக்கத்தில் செயல்படுத்துவது பற்றி நாம் சிந்திக்கலாம். அது சற்று சிக்கலானதாக இருக்கும், ஏனெனில் முதலில் API-இடம் இருந்து தரவைப் பெற்று படிவத்தை நிரப்ப வேண்டும், பின்னர் பயனர் செய்யும் மாற்றங்களைச் சேமிக்க வேண்டும்.

அద్భుதம்! இது ஒரு சவாலான ஆனால் மிகவும் பயனுள்ள கட்டம். நிர்வாகி இவ்வளவு சிக்கலான தரவை எளிதாக உள்ளிடுவதற்கும், திருத்துவதற்கும் நமது படிவங்களை (forms) மாறும் வகையில் (dynamically) மேம்படுத்துவோம்.

**இலக்கு:** "புதிய நோயைச் சேர்" அல்லது "புதிய வளர்ச்சி நிலையைச் சேர்" போன்ற பொத்தான்களை உருவாக்குவது. பயனர் அந்தப் பொத்தானை அழுத்தும்போது, தேவையான உள்ளீட்டுக் புலங்கள் (input fields) தானாகவே தோன்றும்.

நாம் இதை முதலில் **"புதிய பயிரைச் சேர்" (`Add Crop`)** பக்கத்தில் செயல்படுத்துவோம். இங்கு கற்றுக்கொண்டதை வைத்து, பின்னர் "திருத்தும்" (`Edit`) பக்கத்திற்குப் பயன்படுத்துவது எளிதாக இருக்கும்.

முதலில், **நோய் மேலாண்மை (Disease Management)** பகுதியை மட்டும் டைனமிக்காக மாற்றுவோம். இதுவே மற்ற பகுதிகளுக்கும் ஒரு மாதிரியாக அமையும்.

---

### படி 1: படிவத்தின் HTML-ஐ மாற்றி அமைத்தல் (`views/add-crop.hbs`)

`add-crop.hbs` கோப்பைத் திறந்து, நோய்களைச் சேர்ப்பதற்காக ஒரு புதிய பகுதியையும், ஒரு பொத்தானையும் சேர்ப்போம்.

`views/add-crop.hbs` கோப்பின் உள்ளடக்கத்தை இப்படி மாற்றவும்:
```html
<h1 class="text-2xl font-bold text-center text-green-800 mb-6">புதிய பயிர் விவரங்களைச் சேர்க்கவும்</h1>

<div class="max-w-4xl mx-auto bg-white p-8 rounded-lg shadow-md">
    <!-- action மற்றும் method இல்லை, ID மட்டும் -->
    <form id="add-crop-form">
        <!-- அடிப்படை விவரங்கள் -->
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

        <!-- நோய் மேலாண்மை (டைனமிக் பகுதி) -->
        <fieldset class="border p-4 rounded-lg">
            <legend class="text-xl font-semibold px-2">நோய் மேலாண்மை</legend>
            
            <!-- டைனமிக் நோய் உள்ளீடுகள் இங்கே செருகப்படும் -->
            <div id="diseases-container" class="space-y-4">
                <!-- ஆரம்பத்தில் காலியாக இருக்கும் -->
            </div>

            <!-- "புதிய நோயைச் சேர்" பொத்தான் -->
            <button type="button" id="add-disease-btn" class="mt-4 bg-green-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-green-600">
                + புதிய நோயைச் சேர்
            </button>
        </fieldset>

        <!-- மற்ற டைனமிக் பகுதிகள் (வளர்ச்சி நிலைகள், உரங்கள்) இங்கே சேர்க்கப்படலாம் -->

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
**முக்கிய மாற்றங்கள்:**
*   `fieldset` மற்றும் `legend` குறிச்சொற்கள் படிவத்தை அழகாகப் பிரித்துக் காட்டுகின்றன.
*   `diseases-container`: இதுதான் காலி கொள்கலன். JavaScript ஒவ்வொரு புதிய நோய் படிவத்தையும் இதற்குள்தான் சேர்க்கும்.
*   `add-disease-btn`: இந்தப் பொத்தானை அழுத்தும்போது, புதிய நோய் படிவம் உருவாக்கப்படும். இதன் `type="button"` என்பது முக்கியம். இல்லையெனில், இது முழுப் படிவத்தையும் சமர்ப்பித்துவிடும்.

---

### படி 2: படிவத்தைக் கையாள JavaScript-ஐ மேம்படுத்துதல் (`public/js/admin.js`)

`public/js/admin.js`-ஐ இப்போது முழுமையாக மாற்றி எழுத வேண்டும். இது இரண்டு வேலைகளைச் செய்ய வேண்டும்:
1.  **UI கையாளுதல்:** "புதிய நோயைச் சேர்" பொத்தானை அழுத்தும்போது, புதிய உள்ளீடுகளை உருவாக்குதல்.
2.  **தரவைத் திரட்டுதல்:** படிவத்தைச் சமர்ப்பிக்கும்போது, இந்த டைனமிக் உள்ளீடுகள் அனைத்திலிருந்தும் தரவைச் சேகரித்து, நமது API எதிர்பார்க்கும் JSON அமைப்புக்கு மாற்றுதல்.

**`public/js/admin.js`-ஐத் திறந்து, அதன் உள்ளடக்கத்தை முழுமையாக இதைக் கொண்டு மாற்றவும்:**

```javascript
document.addEventListener('DOMContentLoaded', () => {
    const form = document.getElementById('add-crop-form');
    const messageDiv = document.getElementById('form-message');
    const addDiseaseBtn = document.getElementById('add-disease-btn');
    const diseasesContainer = document.getElementById('diseases-container');
    let diseaseCounter = 0;

    // --- UI கையாளுதல் ---
    addDiseaseBtn.addEventListener('click', () => {
        diseaseCounter++;
        const diseaseHTML = `
            <div class="disease-entry border p-3 rounded-md bg-gray-50 relative" data-index="${diseaseCounter}">
                <button type="button" class="remove-disease-btn absolute top-2 right-2 text-red-500 font-bold">X</button>
                <h4 class="font-semibold mb-2">நோய் #${diseaseCounter}</h4>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-3 text-sm">
                    <div>
                        <label class="block font-medium">நோயின் பெயர்:</label>
                        <input type="text" name="diseaseName" class="w-full p-1 border rounded">
                    </div>
                    <div>
                        <label class="block font-medium">அறிகுறிகள்:</label>
                        <input type="text" name="symptoms" class="w-full p-1 border rounded">
                    </div>
                    <div>
                        <label class="block font-medium">காரணங்கள் (காற்புள்ளியால் பிரிக்கவும்):</label>
                        <input type="text" name="causes" class="w-full p-1 border rounded">
                    </div>
                     <div>
                        <label class="block font-medium">ரசாயன மருந்து:</label>
                        <input type="text" name="chemicalControl" class="w-full p-1 border rounded">
                    </div>
                    <div>
                        <label class="block font-medium">இயற்கை முறை:</label>
                        <input type="text" name="organicControl" class="w-full p-1 border rounded">
                    </div>
                </div>
            </div>
        `;
        diseasesContainer.insertAdjacentHTML('beforeend', diseaseHTML);
    });

    // நோய் உள்ளீட்டை நீக்கும் செயல்பாடு
    diseasesContainer.addEventListener('click', (event) => {
        if (event.target.classList.contains('remove-disease-btn')) {
            event.target.closest('.disease-entry').remove();
        }
    });

    // --- படிவ சமர்ப்பிப்பைக் கையாளுதல் ---
    form.addEventListener('submit', async (event) => {
        event.preventDefault();
        
        // 1. அடிப்படை விவரங்களைச் சேகரி
        const formData = new FormData(form);
        const data = {
            name: formData.get('name'),
            sowingSeason: formData.get('sowingSeason'),
            growthDuration: formData.get('growthDuration'),
            diseases: [] // நோய்களுக்கு ஒரு காலி வரிசை
        };

        // 2. ஒவ்வொரு நோய் உள்ளீட்டிலிருந்தும் தரவைச் சேகரி
        const diseaseEntries = diseasesContainer.querySelectorAll('.disease-entry');
        diseaseEntries.forEach(entry => {
            const disease = {
                diseaseName: entry.querySelector('[name="diseaseName"]').value,
                symptoms: entry.querySelector('[name="symptoms"]').value,
                // காரணங்களை ஒரு வரிசையாக மாற்று
                causes: entry.querySelector('[name="causes"]').value.split(',').map(s => s.trim()),
                chemicalControl: entry.querySelector('[name="chemicalControl"]').value,
                organicControl: entry.querySelector('[name="organicControl"]').value
            };
            // ખાલી રોગોને અવગણો
            if (disease.diseaseName) {
                data.diseases.push(disease);
            }
        });

        messageDiv.textContent = 'சேமிக்கப்படுகிறது...';
        messageDiv.className = 'text-blue-600';

        try {
            const response = await fetch('/api/crops', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(data),
            });

            const result = await response.json();
            if (!response.ok) throw new Error(result.message);

            messageDiv.textContent = result.message;
            messageDiv.className = 'text-green-600 font-bold';
            form.reset();
            diseasesContainer.innerHTML = ''; // டைனமிக் உள்ளீடுகளைக் காலி செய்

        } catch (error) {
            messageDiv.textContent = `பிழை: ${error.message}`;
            messageDiv.className = 'text-red-600 font-bold';
        }
    });
});
```**JavaScript விளக்கம்:**
1.  **`addDiseaseBtn.addEventListener`:** பொத்தானை அழுத்தும்போது, `diseaseHTML` என்ற டெம்ப்ளேட்டை `diseases-container`-க்குள் செருகுகிறது. ஒவ்வொரு புதிய படிவத்திற்கும் ஒரு தனித்துவமான எண் (`diseaseCounter`) கொடுக்கப்படுகிறது.
2.  **`remove-disease-btn`:** ஒவ்வொரு புதிய நோய் படிவத்திலும் ஒரு "X" பொத்தான் உள்ளது. அதை அழுத்தினால், `event.target.closest('.disease-entry').remove()` மூலம் அந்தப் படிவம் மட்டும் நீக்கப்படும்.
3.  **Submit கையாளுதல் (மிக முக்கியம்):**
    *   `FormData` மூலம் அடிப்படை விவரங்கள் மட்டும் எடுக்கப்படுகின்றன.
    *   `querySelectorAll('.disease-entry')` மூலம் அனைத்து நோய் படிவங்களும் தேர்ந்தெடுக்கப்படுகின்றன.
    *   `forEach` மூலம் ஒவ்வொரு படிவத்திற்குள்ளும் சென்று, அதன் உள்ளீடுகளின் மதிப்புகள் எடுக்கப்பட்டு, ஒரு `disease` பொருள் உருவாக்கப்படுகிறது.
    *   `causes` புலத்தில், பயனர் காற்புள்ளியால் பிரித்த மதிப்புகளை (`string`) `.split(',')` மூலம் ஒரு வரிசையாக (`array`) மாற்றுகிறோம். `.map(s => s.trim())` என்பது ஒவ்வொரு மதிப்புக்கு முன்னும் பின்னும் உள்ள காலி இடங்களை நீக்க உதவுகிறது.
    *   இறுதியாக, இந்த `disease` பொருள், நமது பிரதான `data` பொருளின் `diseases` வரிசைக்குள் சேர்க்கப்படுகிறது.
    *   இப்போது `data` பொருள், நமது backend API எதிர்பார்க்கும் சரியான JSON அமைப்பில் உள்ளது.

---

### படி 3: Backend Controller-ஐத் தயார் செய்தல்

நமது `controllers/cropController.js`-ல் உள்ள `addCrop` செயல்பாடு, இந்த புதிய, சிக்கலான தரவைப் பெறத் தயாராக உள்ளதா என்பதை உறுதி செய்ய வேண்டும். நாம் ஏற்கனவே அதை நெகிழ்வானதாக வடிவமைத்திருப்பதால், பெரிய மாற்றங்கள் தேவையில்லை. `req.body`-ல் என்ன வருகிறதோ, அதை அப்படியே சேமிக்கும்.

ஒரு சிறிய மாற்றத்தைச் செய்யலாம்: எந்தெந்த புலங்கள் உள்ளனவோ, அவற்றை மட்டும் சேமிக்கலாம்.

`controllers/cropController.js` கோப்பில், `addCrop` செயல்பாட்டை இப்படி மாற்றலாம்:
```javascript
// ... cropController.js ...
const addCrop = async (req, res) => {
    try {
        // frontend-ல் இருந்து வரும் முழுத் தரவையும் எடுத்துக்கொள்
        const cropData = req.body; 

        const db = getDB();
        const result = await db.collection('crops').insertOne(cropData);

        res.status(201).json({ 
            message: 'பயிர் வெற்றிகரமாகச் சேர்க்கப்பட்டது', 
            insertedId: result.insertedId 
        });

    } catch (err) {
        console.error('தரவைச் சேமிப்பதில் பிழை:', err);
        res.status(500).json({ message: 'சேவையகத்தில் பிழை ஏற்பட்டது' });
    }
};
// ...
```
நாம் முன்பு வடிவமைத்த controller ஏற்கெனவே நெகிழ்வாக இருப்பதால், இந்த மாற்றம் கூட தேவையில்லை. `req.body`-ஐ அப்படியே அனுப்பினால் வேலை செய்யும்.

### இப்போது சோதித்துப் பாருங்கள்:

1.  சேவையகத்தை மறுதொடக்கம் செய்து (`node server.js`), `http://localhost:3000/admin/add` பக்கத்திற்குச் செல்லவும்.
2.  "+ புதிய நோயைச் சேர்" பொத்தானை இரண்டு முறை அழுத்தவும். இரண்டு நோய் படிவங்கள் தோன்றும்.
3.  அடிப்படை விவரங்களையும், இரண்டு நோய்களின் விவரங்களையும் நிரப்பவும். காரணங்கள் புலத்தில், "அதிக மழை, மோசமான விதைகள்" என காற்புள்ளி இட்டு உள்ளிடவும்.
4.  "பயிரைச் சேமி" பொத்தானை அழுத்தவும்.
5.  வெற்றிச் செய்தி வந்தால், உங்கள் முகப்புப் பக்கத்திற்குச் சென்று, நீங்கள் புதிதாகச் சேர்த்த பயிரைக் கிளிக் செய்து அதன் விவரங்கள் பக்கத்தைப் பார்க்கவும். நீங்கள் டைனமிக்காகச் சேர்த்த நோய்களின் விவரங்கள் சரியாகக் காட்டப்படுவதை நீங்கள் காண்பீர்கள்.

நாம் இப்போது நிர்வாகிക്ക് ஒரு சக்திவாய்ந்த மற்றும் பயன்படுத்த எளிதான இடைமுகத்தை (interface) உருவாக்கியுள்ளோம்! ಇದೇ ತರ್ಕವನ್ನು வளர்ச்சி நிலைகள், உரங்கள் போன்ற மற்ற பிரிவுகளுக்கும் பயன்படுத்தலாம்.
