**அடுத்த படி:**

நிர்வாகப் பக்கத்தில், தற்போதுள்ள பயிர்களின் பட்டியலைக் காட்டி, அவற்றை **திருத்துவதற்கும் (Edit)** மற்றும் **நீக்குவதற்கும் (Delete)** உண்டான API 
மற்றும் UI-ஐ உருவாக்கலாமா? இது நமது நிர்வாகப் பக்கத்தை முழுமையாக்கும்.

ยอดเยี่ยม! நமது நிர்வாகப் பக்கத்தை (Admin Panel) முழுமையாக்குவதற்கான அடுத்த முக்கியமான கட்டத்தை ఇప్పుడు தொடங்குவோம்.

சேமிக்கப்பட்ட பயிர்களைப் பட்டியலிட்டு, அவற்றை அங்கிருந்தே **திருத்த (Edit)** மற்றும் **நீக்க (Delete)** வசதிகளை உருவாக்குவோம்.

### திட்டம்: மூன்று முக்கியப் படிகள்

1.  **Backend API-ஐ மேம்படுத்துதல்:**
    *   ஒரு குறிப்பிட்ட பயிரின் விவரங்களைப் பெற: `GET /api/crops/:id`
    *   ஒரு பயிரின் விவரங்களைப் புதுப்பிக்க: `PUT /api/crops/:id`
    *   ஒரு பயிரை நீக்க: `DELETE /api/crops/:id`
    ஆகிய மூன்று புதிய API வழித்தடங்களை (endpoints) உருவாக்குவோம்.

2.  **நிர்வாக டாஷ்போர்டை உருவாக்குதல் (UI):**
    *   அனைத்துப் பயிர்களையும் ஒரு அட்டவணையில் பட்டியலிட `/admin/dashboard` என்ற புதிய பக்கத்தை உருவாக்குவோம்.
    *   ஒவ்வொரு பயிரின் வரிசையிலும் "திருத்து" மற்றும் "நீக்கு" பொத்தான்கள் இருக்கும்.

3.  **AJAX மூலம் செயல்பாடுகளை இணைத்தல்:**
    *   டாஷ்போர்டில் உள்ள "நீக்கு" பொத்தானை அழுத்தும்போது, பக்கம் மீண்டும் ஏற்றப்படாமல் பயிர் நீக்கப்படும்.
    *   "திருத்து" பொத்தானை அழுத்தினால், அந்தப் பயிரின் விவரங்கள் நிரப்பப்பட்ட ஒரு புதிய திருத்தும் பக்கத்திற்குச் செல்லும்.

---

### படி 1: Backend API-ஐ மேம்படுத்துதல்

முதலில், நமது controller மற்றும் router-ல் தேவையான மாற்றங்களைச் செய்வோம்.

#### 1. Controller-ல் புதிய செயல்பாடுகளைச் சேர்த்தல் (`controllers/cropController.js`)

MongoDB-ல் ஒரு குறிப்பிட்ட ஆவணத்தைக் (document) கண்டுபிடிக்க, அதன் பிரத்யேக `_id`-ஐப் பயன்படுத்த வேண்டும். ஆனால், இந்த `_id` ஒரு சரம் (string) அல்ல, அது ஒரு `ObjectId` வகை. எனவே, `mongodb` தொகுப்பிலிருந்து `ObjectId`-ஐ இறக்குமதி செய்ய வேண்டும்.

`controllers/cropController.js` கோப்பின் மேற்பகுதியில் இதைச் சேர்க்கவும்:
```javascript
const { getDB } = require('../config/db');
// ObjectId-ஐ இறக்குமதி செய்யவும்
const { ObjectId } = require('mongodb'); 
```

இப்போது, கோப்பின் இறுதியில் புதிய செயல்பாடுகளைச் சேர்க்கவும்:
```javascript
// ... getAllCrops மற்றும் addCrop செயல்பாடுகளுக்குப் பிறகு ...

// GET /api/crops/:id - ஒரு குறிப்பிட்ட பயிரைப் பெற
const getCropById = async (req, res) => {
    try {
        const db = getDB();
        // URL-ல் இருந்து வரும் id ஒரு string. அதை ObjectId ஆக மாற்ற வேண்டும்.
        const crop = await db.collection('crops').findOne({ _id: new ObjectId(req.params.id) });
        if (!crop) {
            return res.status(404).json({ message: 'பயிர் கண்டுபிடிக்கப்படவில்லை' });
        }
        res.status(200).json(crop);
    } catch (err) {
        res.status(500).json({ message: 'சேவையகத்தில் பிழை ஏற்பட்டது' });
    }
};

// PUT /api/crops/:id - ஒரு பயிரைப் புதுப்பிக்க
const updateCrop = async (req, res) => {
    try {
        const db = getDB();
        const { name, sowingSeason, growthDuration } = req.body;
        const result = await db.collection('crops').updateOne(
            { _id: new ObjectId(req.params.id) }, // எதைப் புதுப்பிக்க வேண்டும்
            { $set: { name, sowingSeason, growthDuration } } // என்னவாகப் புதுப்பிக்க வேண்டும்
        );

        if (result.matchedCount === 0) {
            return res.status(404).json({ message: 'பயிர் கண்டுபிடிக்கப்படவில்லை' });
        }
        res.status(200).json({ message: 'பயிர் வெற்றிகரமாகப் புதுப்பிக்கப்பட்டது' });
    } catch (err) {
        res.status(500).json({ message: 'சேவையகத்தில் பிழை ஏற்பட்டது' });
    }
};

// DELETE /api/crops/:id - ஒரு பயிரை நீக்க
const deleteCrop = async (req, res) => {
    try {
        const db = getDB();
        const result = await db.collection('crops').deleteOne({ _id: new ObjectId(req.params.id) });

        if (result.deletedCount === 0) {
            return res.status(404).json({ message: 'பயிர் கண்டுபிடிக்கப்படவில்லை' });
        }
        res.status(200).json({ message: 'பயிர் வெற்றிகரமாக நீக்கப்பட்டது' });
    } catch (err) {
        res.status(500).json({ message: 'சேவையகத்தில் பிழை ஏற்பட்டது' });
    }
};

// புதிய செயல்பாடுகளையும் ஏற்றுமதி செய்யவும்
module.exports = {
    getAllCrops,
    addCrop,
    getCropById,
    updateCrop,
    deleteCrop
};
```

#### 2. Router-ல் புதிய வழித்தடங்களைச் சேர்த்தல் (`routes/cropRoutes.js`)

`routes/cropRoutes.js`-ஐத் திறந்து, புதிய வழித்தடங்களை இணைக்கவும்.

```javascript
const express = require('express');
const router = express.Router();
// இப்போது அனைத்து செயல்பாடுகளையும் இறக்குமதி செய்யவும்
const { 
    getAllCrops, 
    addCrop, 
    getCropById, 
    updateCrop, 
    deleteCrop 
} = require('../controllers/cropController');

// தற்போதுள்ள வழித்தடங்கள்
router.get('/crops', getAllCrops);
router.post('/crops', addCrop);

// புதிய வழித்தடங்கள்
// ':id' என்பது URL-ல் வரும் ஒரு மாறும் மதிப்பு (parameter)
router.get('/crops/:id', getCropById);
router.put('/crops/:id', updateCrop);
router.delete('/crops/:id', deleteCrop);

module.exports = router;
```

Backend இப்போது தயாராகிவிட்டது!

---

### படி 2: நிர்வாக டாஷ்போர்டு UI மற்றும் நீக்கும் (Delete) செயல்பாடு

#### 1. டாஷ்போர்டு பக்கத்தை வழங்குதல் (`server.js` மற்றும் `admin-dashboard.hbs`)

`server.js`-ல், டாஷ்போர்டு பக்கத்தைக் காட்டுவதற்கு ஒரு புதிய வழித்தடத்தைச் சேர்க்கவும்.
```javascript
// ... server.js இல் Page Rendering Routes பகுதிக்குக் கீழே ...
app.get('/admin/dashboard', (req, res) => {
    res.render('admin-dashboard', { title: 'நிர்வாக டாஷ்போர்டு' });
});
```

`views` கோப்புறையில், `admin-dashboard.hbs` என்ற புதிய கோப்பை உருவாக்கவும்.
```html
<h1 class="text-3xl font-bold text-center text-green-800 mb-8">நிர்வாக டாஷ்போர்டு</h1>

<div class="bg-white p-6 rounded-lg shadow-md">
    <div class="overflow-x-auto">
        <table class="min-w-full">
            <thead class="bg-gray-100">
                <tr>
                    <th class="p-3 text-left">பயிரின் பெயர்</th>
                    <th class="p-3 text-left">விதைப்பு காலம்</th>
                    <th class="p-3 text-left">வளர்ச்சி காலம்</th>
                    <th class="p-3 text-center">செயல்பாடுகள்</th>
                </tr>
            </thead>
            <!-- AJAX மூலம் தரவு இங்கே செருகப்படும் -->
            <tbody id="crop-table-body">
                <tr>
                    <td colspan="4" class="p-4 text-center text-gray-500">தரவுகளை ஏற்றுகிறது...</td>
                </tr>
            </tbody>
        </table>
    </div>
</div>

<script src="/js/admin-dashboard.js"></script>
```

#### 2. AJAX மூலம் பட்டியலைக் காட்டுதல் மற்றும் நீக்குதல் (`public/js/admin-dashboard.js`)

`public/js` கோப்புறையில், `admin-dashboard.js` என்ற புதிய கோப்பை உருவாக்கவும்.

```javascript
document.addEventListener('DOMContentLoaded', () => {
    const tableBody = document.getElementById('crop-table-body');

    // பயிர்களைப் பெற்று அட்டவணையில் நிரப்பும் செயல்பாடு
    const populateTable = async () => {
        try {
            const response = await fetch('/api/crops');
            const crops = await response.json();
            
            tableBody.innerHTML = ''; // "Loading..." செய்தியை நீக்கு

            if (crops.length === 0) {
                tableBody.innerHTML = '<tr><td colspan="4" class="p-4 text-center">சேமிக்க பயிர்கள் எதுவும் இல்லை.</td></tr>';
                return;
            }

            crops.forEach(crop => {
                const row = `
                    <tr id="crop-${crop._id}">
                        <td class="p-3">${crop.name}</td>
                        <td class="p-3">${crop.sowingSeason}</td>
                        <td class="p-3">${crop.growthDuration}</td>
                        <td class="p-3 text-center">
                            <a href="/admin/edit/${crop._id}" class="text-blue-600 hover:underline mr-4">திருத்து</a>
                            <button data-id="${crop._id}" class="delete-btn text-red-600 hover:underline">நீக்கு</button>
                        </td>
                    </tr>
                `;
                tableBody.insertAdjacentHTML('beforeend', row);
            });
        } catch (error) {
            tableBody.innerHTML = `<tr><td colspan="4" class="p-4 text-center text-red-600">பிழை: ${error.message}</td></tr>`;
        }
    };

    // நீக்கும் செயல்பாட்டைக் கையாளுதல்
    tableBody.addEventListener('click', async (event) => {
        // கிளிக் செய்யப்பட்டது "delete-btn" வகுப்பு கொண்ட பொத்தானா என்று பார்க்கவும்
        if (event.target.classList.contains('delete-btn')) {
            const cropId = event.target.dataset.id;
            
            // பயனரிடம் உறுதிப்படுத்தவும்
            if (confirm('இந்தப் பயிரை நீக்க விரும்புகிறீர்களா?')) {
                try {
                    const response = await fetch(`/api/crops/${cropId}`, {
                        method: 'DELETE',
                    });

                    if (!response.ok) {
                        const result = await response.json();
                        throw new Error(result.message);
                    }
                    
                    // வெற்றி பெற்றால், HTML-ல் இருந்து அந்த வரிசையை நீக்கவும்
                    document.getElementById(`crop-${cropId}`).remove();
                    
                } catch (error) {
                    alert(`நீக்குவதில் பிழை: ${error.message}`);
                }
            }
        }
    });

    // பக்கம் ஏற்றப்பட்டவுடன் அட்டவணையை நிரப்பவும்
    populateTable();
});
```**முக்கிய அம்சங்கள்:**
*   **பட்டியலிடுதல்:** `populateTable` செயல்பாடு `/api/crops`-ஐ அழைத்து, ஒவ்வொரு பயிருக்கும் ஒரு `<tr>` (table row)-ஐ உருவாக்குகிறது.
*   **நீக்குதல்:**
    *   `event delegation` முறையைப் பயன்படுத்தி, முழு `tableBody`-க்கும் ஒரே ஒரு click listener-ஐச் சேர்த்துள்ளோம். இது செயல்திறனுக்கு நல்லது.
    *   `data-id="${crop._id}"` மூலம் ஒவ்வொரு பொத்தானுடனும் பயிரின் ID-ஐ சேமிக்கிறோம்.
    *   `window.confirm()` மூலம் நீக்குவதற்கு முன் பயனரிடம் உறுதிப்படுத்துகிறோம்.
    *   `DELETE` கோரிக்கையை `fetch` மூலம் அனுப்புகிறோம்.
    *   வெற்றி பெற்றால், `document.getElementById(...).remove()` மூலம் பக்கம் மீண்டும் ஏற்றப்படாமல் அந்த வரிசையை மட்டும் நீக்குகிறோம்.

---

**இப்போது, நீங்கள் `http://localhost:3000/admin/dashboard` என்ற பக்கத்திற்குச் சென்றால், நீங்கள் சேமித்த பயிர்களின் பட்டியலைக் காண்பீர்கள். "நீக்கு" பொத்தானை அழுத்தினால், அது உறுதிப்படுத்தல் கேட்டு, பின்னர் அந்தப் பயிரை நீக்கும்.**

திருத்தும் (Edit) செயல்பாடு சற்று பெரியது என்பதால், அதை அடுத்த படியாகச் செய்யலாம்.

