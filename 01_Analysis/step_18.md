**நாம் முதலில் Backend-ல் `multer`-ஐ வழித்தடங்களுடன் (routes) இணைத்து, நிர்வாகப் பக்கத்தில் (`Admin Panel`) புகைப்படங்களைப் பதிவேற்றும் படிவத்தை உருவாக்குவதில் இருந்து தொடங்கலாமா?**

நிச்சயமாக, தொடர்வோம்!

Backend-ல் `multer`-ஐ வழித்தடங்களுடன் இணைத்து, நிர்வாகப் பக்கத்தில் புகைப்படங்களை உள்ளிடுவதற்கான படிவத்தை உருவாக்குவோம். இது மிகவும் முக்கியமான படி.

### படி 2: வழித்தடங்களில் Multer-ஐ இணைத்தல் மற்றும் Controller-ஐ மேம்படுத்துதல்

முதலில், நிர்வாகி ஒரு பயிரைச் சேர்க்கும்போதும், திருத்தும்போதும் புகைப்படங்களைப் பதிவேற்றுவதை আমাদের Backend கையாள வேண்டும்.

#### 1. Router-ல் Multer Middleware-ஐச் சேர்த்தல் (`routes/cropRoutes.js`)

`routes/cropRoutes.js`-ஐத் திறந்து, `addCrop` மற்றும் `updateCrop` வழித்தடங்களில் நாம் உருவாக்கிய `upload` middleware-ஐப் பயன்படுத்த வேண்டும்.

ஒரு பயிருக்கு ஒரு `coverImage` (முக்கியப் புகைப்படம்) மற்றும் பல `galleryImages` (காட்சியகப் புகைப்படங்கள்) இருக்கலாம். மேலும், ஒவ்வொரு நோய்க்கும் ஒரு புகைப்படம் இருக்கலாம். `multer`-ல் உள்ள `.fields()` முறை, பலதரப்பட்ட கோப்பு உள்ளீடுகளை ஒரே நேரத்தில் கையாள உதவுகிறது.

`routes/cropRoutes.js`-ஐ பின்வருமாறு மாற்றவும்:
```javascript
// routes/cropRoutes.js
const express = require('express');
const router = express.Router();
const { /* ... controller functions ... */ } = require('../controllers/cropController');
const upload = require('../middleware/upload'); // upload middleware-ஐ இறக்குமதி செய்யவும்

// ... மற்ற வழித்தடங்கள் ...

// POST /api/crops - ஒரு புதிய பயிரைச் சேர்க்க
router.post(
    '/crops',
    upload.fields([
        { name: 'coverImage', maxCount: 1 },
        { name: 'galleryImages', maxCount: 5 },
        // நோய்களின் புகைப்படங்கள் டைனமிக்காக வருவதால், அவற்றை controller-ல் கையாளலாம்
    ]),
    addCrop
);

// PUT /api/crops/:id - ஒரு பயிரைப் புதுப்பிக்க
router.put(
    '/crops/:id',
    upload.fields([
        { name: 'coverImage', maxCount: 1 },
        { name: 'galleryImages', maxCount: 5 },
    ]),
    updateCrop
);

module.exports = router;
```
**விளக்கம்:**
*   `upload.fields([...])`: இது `multer`-க்கு, படிவத்தில் `coverImage` என்ற பெயரில் ஒரு கோப்பும், `galleryImages` என்ற பெயரில் 5 கோப்புகள் வரை வரலாம் என்று கூறுகிறது.
*   **முக்கிய குறிப்பு:** நோய்களின் புகைப்படங்களை இங்கே குறிப்பிடவில்லை. കാരണം அவை டைனமிக் ஆக `diseaseImage_0`, `diseaseImage_1` என வரலாம். இதை controller-ல் `req.files`-ஐ ஆய்வு செய்து கையாள வேண்டும்.

#### 2. Controller-ல் படப் பாதைகளைச் சேமித்தல் (`controllers/cropController.js`)

இப்போது, `addCrop` மற்றும் `updateCrop` செயல்பாடுகளை, `req.files`-ல் இருந்து வரும் கோப்புத் தகவல்களைப் பெற்று, వాటిని MongoDB-ல் சேமிக்குமாறு மாற்றுவோம்.

**`controllers/cropController.js`-ல் `addCrop` செயல்பாடு:**
```javascript
// controllers/cropController.js
// ...

const addCrop = async (req, res) => {
    try {
        // படி 2.1: AJAX FormData-விலிருந்து வரும் உரைத் தரவு JSON string ஆக இருக்கும்.
        // அதை JavaScript பொருளாக மாற்ற வேண்டும்.
        const cropData = JSON.parse(req.body.data);

        // படி 2.2: பதிவேற்றப்பட்ட கோப்புகளின் பாதைகளைச் சேர்
        if (req.files) {
            if (req.files.coverImage) {
                cropData.coverImage = req.files.coverImage[0].path.replace('public', '');
            }
            if (req.files.galleryImages) {
                cropData.galleryImages = req.files.galleryImages.map(file => file.path.replace('public', ''));
            }
            // நோய்களின் புகைப்படங்களைக் கையாளுதல்
            Object.keys(req.files).forEach(key => {
                if (key.startsWith('diseaseImage_')) {
                    const index = parseInt(key.split('_')[1]);
                    if (cropData.diseases && cropData.diseases[index]) {
                        cropData.diseases[index].image = req.files[key][0].path.replace('public', '');
                    }
                }
            });
        }
        
        const db = getDB();
        const result = await db.collection('crops').insertOne(cropData);

        res.status(201).json({ 
            message: 'பயிர் மற்றும் படங்கள் வெற்றிகரமாகச் சேர்க்கப்பட்டன', 
            insertedId: result.insertedId 
        });

    } catch (err) {
        console.error('தரவைச் சேமிப்பதில் பிழை:', err);
        res.status(500).json({ message: 'சேவையகத்தில் பிழை ஏற்பட்டது' });
    }
};

// updateCrop செயல்பாட்டிலும் இதே போன்ற மாற்றங்கள் தேவைப்படும்.
// ...
```
**விளக்கம்:**
*   **`JSON.parse(req.body.data)`:** frontend-ல் இருந்து `FormData` மூலம் அனுப்பும்போது, நமது JSON தரவை `data` என்ற ஒரு புலத்தில் string ஆக அனுப்புவோம். அதை இங்கே மீண்டும் பொருளாக மாற்றுகிறோம்.
*   **`req.files`:** `multer` பதிவேற்றிய கோப்புகளின் விவரங்களை `req.files` என்ற பொருளில் வைக்கும். `req.files.coverImage` ஒரு வரிசையாக (array) இருக்கும்.
*   **`.path.replace('public', '')`:** கோப்பின் முழுப் பாதை (`public/uploads/file.jpg`) சேமிக்கப்படும். `public`-ஐ நீக்குவதன் மூலம், frontend-ல் `/uploads/file.jpg` என எளிதாகப் பயன்படுத்தலாம்.
*   **நோய்களின் படங்கள்:** `Object.keys(req.files).forEach(...)` மூலம் `req.files`-ல் உள்ள அனைத்து கோப்புகளையும் சரிபார்த்து, `diseaseImage_0`, `diseaseImage_1` எனத் தொடங்கும் கோப்புகளைக் கண்டுபிடித்து, சரியான நோயுடன் அதன் படப் பாதையை இணைக்கிறோம்.

---

### படி 3: நிர்வாகப் பக்க படிவத்தில் கோப்பு உள்ளீடுகளைச் சேர்த்தல் (`views/add-crop.hbs`)

இப்போது, நிர்வாகி புகைப்படங்களைப் பதிவேற்ற, படிவத்தில் `<input type="file">`-ஐச் சேர்ப்போம்.

`views/add-crop.hbs`-ல், அடிப்படை விவரங்கள் `fieldset`-க்குள் இதைச் சேர்க்கவும்:
```html
<!-- views/add-crop.hbs -->
<fieldset class="border p-4 rounded-lg mb-6">
    <legend class="text-xl font-semibold px-2">அடிப்படை விவரங்கள் & படங்கள்</legend>
    <!-- ... பெயர், விதைப்பு காலம், வளர்ச்சி காலம் உள்ளீடுகள் ... -->

    <!-- படி 3.1: புதிய கோப்பு உள்ளீடுகள் -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-4">
        <div>
            <label for="coverImage" class="block text-gray-700 font-bold mb-1">முக்கியப் புகைப்படம் (Cover Image):</label>
            <input type="file" id="coverImage" name="coverImage" class="w-full">
        </div>
        <div>
            <label for="galleryImages" class="block text-gray-700 font-bold mb-1">காட்சியகப் புகைப்படங்கள் (Gallery - Max 5):</label>
            <input type="file" id="galleryImages" name="galleryImages" class="w-full" multiple>
        </div>
    </div>
</fieldset>

<!-- நோய் மேலாண்மை (டைனமிக்) -->
<!-- ... diseases-container மற்றும் add-disease-btn ... -->
```

மேலும், நோய் உள்ளீட்டை உருவாக்கும் JavaScript-ல், ஒவ்வொரு நோய்க்கும் ஒரு கோப்பு உள்ளீட்டைச் சேர்க்க வேண்டும்.

---

### படி 4: AJAX-ஐ `FormData` பயன்படுத்த மாற்றுதல் (`public/js/admin.js`)

இது frontend-ல் மிக முக்கியமான மாற்றம். `JSON.stringify()`-க்கு பதிலாக, கோப்புகள் மற்றும் உரைத் தரவை அனுப்ப `FormData`-ஐப் பயன்படுத்த வேண்டும்.

**`public/js/admin.js`-ல், `form.addEventListener('submit', ...)` செயல்பாட்டை முழுமையாக மாற்றவும்:**

```javascript
// public/js/admin.js
// ... submit listener-க்குள் ...
form.addEventListener('submit', async (event) => {
    event.preventDefault();
    
    // படி 4.1: உரைத் தரவை முதலில் ஒரு பொருளாகச் சேகரி
    const textData = {
        name: form.querySelector('[name="name"]').value,
        // ... மற்ற அனைத்து உரைத் தரவுகளையும் (வளர்ச்சி நிலைகள், நோய்கள், உரங்கள்) இங்கே சேகரி ...
    };

    // படி 4.2: ஒரு புதிய FormData பொருளை உருவாக்கு
    const formData = new FormData();

    // படி 4.3: உரைத் தரவை ஒரு JSON string ஆக FormData-வில் சேர்
    formData.append('data', JSON.stringify(textData));

    // படி 4.4: கோப்புகளை FormData-வில் சேர்
    const coverImageInput = form.querySelector('#coverImage');
    if (coverImageInput.files[0]) {
        formData.append('coverImage', coverImageInput.files[0]);
    }

    const galleryImagesInput = form.querySelector('#galleryImages');
    for (const file of galleryImagesInput.files) {
        formData.append('galleryImages', file);
    }
    
    // நோய்களின் புகைப்படங்களைச் சேர்
    diseasesContainer.querySelectorAll('.disease-entry').forEach((entry, index) => {
        const imageInput = entry.querySelector('[name="diseaseImage"]');
        if (imageInput && imageInput.files[0]) {
            formData.append(`diseaseImage_${index}`, imageInput.files[0]);
        }
    });

    messageDiv.textContent = 'சேமிக்கப்படுகிறது...';
    
    try {
        // படி 4.5: fetch API அழைப்பை மாற்று
        const response = await fetch('/api/crops', {
            method: 'POST',
            // Content-Type header-ஐ நீக்கவும். உலாவி FormData-விற்கான சரியான header-ஐ தானாகவே அமைக்கும்.
            body: formData, 
        });
        
        const result = await response.json();
        // ... மீதமுள்ள வெற்றி/பிழை கையாளுதல் தர்க்கம் ...

    } catch (error) {
        // ...
    }
});

// நோயைச் சேர்க்கும் HTML-ல், கோப்பு உள்ளீட்டைச் சேர்க்கவும்
function createDiseaseHTML(data = {}) {
    // ...
    // <input type="file" name="diseaseImage" class="w-full mt-1">
    // ...
}
```

### சோதித்துப் பார்ப்போம்

1.  உங்கள் சேவையகத்தை மறுதொடக்கம் செய்யவும்.
2.  `public` கோப்புறையில் `uploads` என்ற புதிய கோப்புறையை உருவாக்கவும்.
3.  நிர்வாகப் பக்கத்தில் (`/admin/add`), ஒரு புதிய பயிரின் விவரங்களை நிரப்பவும்.
4.  ஒரு முக்கியப் புகைப்படம் மற்றும் சில காட்சியகப் புகைப்படங்களைத் தேர்ந்தெடுக்கவும்.
5.  ஒரு புதிய நோயைச் சேர்த்து, அதற்கும் ஒரு புகைப்படத்தைத் தேர்ந்தெடுக்கவும்.
6.  "பயிரைச் சேமி" பொத்தானை அழுத்தவும்.
7.  உங்கள் முனையத்தில் பிழை ஏதும் வரவில்லை என்றால், மற்றும் `public/uploads` கோப்புறையில் நீங்கள் பதிவேற்றிய கோப்புகள் சேமிக்கப்பட்டிருந்தால், நீங்கள் வெற்றிகரமாகச் செய்துவிட்டீர்கள்!
8.  MongoDB Compass-ல், அந்தப் பயிரின் ஆவணத்தைப் பார்த்தால், `coverImage`, `galleryImages`, மற்றும் `diseases.image` புலங்களில் கோப்புப் பாதைகள் சேமிக்கப்பட்டிருப்பதைக் காணலாம்.

நாம் இப்போது Backend மற்றும் நிர்வாகப் பக்கத்தில் புகைப்படங்களைப் பதிவேற்றும் செயல்பாட்டை முழுமையாக முடித்துவிட்டோம்.

