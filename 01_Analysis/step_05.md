அருமை! சரியான பாதையில் செல்கிறோம். நேட்டிவ் MongoDB டிரைவரைப் பயன்படுத்திச் செயல்படுத்துவது, உங்கள் கற்றலை அடுத்த கட்டத்திற்குக் கொண்டு செல்லும்.

நாம் திட்டமிட்டபடி, நிர்வாகப் பக்கத்தில் (Admin Panel) ஒரு புதிய பயிரைச் சேர்ப்பதற்கான படிவத்தை உருவாக்கி, அந்தத் தரவை MongoDB-ல் சேமிப்போம்.

### படி 2: நிர்வாகிക്ക് ஒரு பயிர் சேர்க்கும் படிவத்தை உருவாக்குதல்

இதை இரண்டு முக்கியப் பகுதிகளாகப் பிரிப்போம்:

1.  **படிவத்தைக் காட்டுதல் (GET Route):** பயிர் விவரங்களை உள்ளிடுவதற்கான HTML படிவத்தைக் காண்பிக்கும் ஒரு பக்கத்தை உருவாக்குதல்.
2.  **படிவத் தரவைச் சேமித்தல் (POST Route):** பயனர் படிவத்தைச் சமர்ப்பித்த பிறகு, அந்தத் தரவைப் பெற்று MongoDB-ல் சேமித்தல்.

---

#### 1. புதிய கோப்புகளை உருவாக்குதல்

முதலில், நிர்வாகப் பக்கங்களுக்கான வழித்தடங்களையும் (routes) காட்சிகளையும் (views) ஒழுங்கமைக்கலாம்.

*   `views/` கோப்புறையில், `add-crop.hbs` என்ற புதிய கோப்பை உருவாக்கவும்.
*   `server.js` இல், இந்த `add-crop.hbs` பக்கத்தைக் காட்டுவதற்கு ஒரு புதிய வழித்தடத்தை (`route`) உருவாக்குவோம்.

#### 2. படிவத்தைக் காண்பிப்பதற்கான வழித்தடத்தை உருவாக்குதல் (GET Route)

**`views/add-crop.hbs`**

இந்தக் கோப்பில், பயிர் விவரங்களை உள்ளிடுவதற்கான HTML படிவத்தை Tailwind CSS உடன் உருவாக்குவோம்.

```html
<h1 class="text-2xl font-bold text-center text-green-800 mb-6">புதிய பயிர் விவரங்களைச் சேர்க்கவும்</h1>

<div class="max-w-2xl mx-auto bg-white p-8 rounded-lg shadow-md">
    <form action="/admin/add" method="POST">
        <div class="mb-4">
            <label for="cropName" class="block text-gray-700 font-bold mb-2">பயிரின் பெயர்:</label>
            <input type="text" id="cropName" name="cropName" class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500" placeholder="எ.கா. மாப்பிள்ளை சம்பா" required>
        </div>

        <div class="mb-4">
            <label for="sowingSeason" class="block text-gray-700 font-bold mb-2">விதைப்பு காலம்:</label>
            <input type="text" id="sowingSeason" name="sowingSeason" class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500" placeholder="எ.கா. ஆடி மாசம் (July-August)" required>
        </div>

        <div class="mb-6">
            <label for="growthDuration" class="block text-gray-700 font-bold mb-2">வளர்ச்சி காலம் (நாட்களில்):</label>
            <input type="text" id="growthDuration" name="growthDuration" class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500" placeholder="எ.கா. 120-135 நாட்கள்" required>
        </div>

        <div class="text-center">
            <button type="submit" class="bg-green-700 text-white font-bold py-2 px-6 rounded-lg hover:bg-green-800 transition duration-200">
                சேமி
            </button>
        </div>
    </form>
</div>
```
**விளக்கம்:**
*   `action="/admin/add"`: படிவத்தைச் சமர்ப்பிக்கும்போது, `/admin/add` என்ற URL-க்கு தரவை அனுப்ப வேண்டும் என்று கூறுகிறது.
*   `method="POST"`: தரவை பாதுகாப்பான முறையில் அனுப்ப POST முறையைப் பயன்படுத்துகிறோம்.
*   `name` பண்பு: ஒவ்வொரு உள்ளீட்டிற்கும் `name` (எ.கா., `cropName`) கொடுப்பது மிகவும் முக்கியம். இதை வைத்துத்தான் நாம் Node.js-ல் தரவைப் பெறுவோம்.

#### 3. படிவத் தரவைச் சேமிப்பதற்கான வழித்தடத்தை உருவாக்குதல் (POST Route)

இப்போது, `server.js` கோப்பை மாற்றி அமைத்து, படிவத்தைக் காட்டவும், தரவைச் சேமிக்கவும் தேவையான வழித்தடங்களைச் சேர்ப்போம்.

**முக்கிய மாற்றம்:** படிவத்திலிருந்து வரும் தரவைப் படிக்க, Express-க்கு ஒரு "இடைப்பொருள்" (Middleware) தேவை. அது `express.urlencoded`.

**`server.js` (முழுமையாக மாற்றியமைக்கப்பட்டது)**

```javascript
const express = require('express');
const { engine } = require('express-handlebars');
const { connectDB, getDB } = require('./config/db');

const app = express();
const PORT = 3000;

// Middleware to parse form data
// இந்த வரி படிவத்திலிருந்து வரும் தரவை req.body இல் ஒரு பொருளாக மாற்றுகிறது.
app.use(express.urlencoded({ extended: true }));

// Handlebars and static files setup
app.engine('hbs', engine({ extname: '.hbs', defaultLayout: 'main' }));
app.set('view engine', 'hbs');
app.set('views', './views');
app.use(express.static('public'));

// === பொதுவான பக்கங்கள் (Public Routes) ===
app.get('/', (req, res) => {
    res.render('home', { title: 'விவசாய வழிகாட்டி' });
});

// === நிர்வாகப் பக்கங்கள் (Admin Routes) ===

// படி 1: பயிர் சேர்க்கும் படிவத்தைக் காட்டும் பக்கம் (GET request)
app.get('/admin/add', (req, res) => {
    res.render('add-crop', { title: 'புதிய பயிரைச் சேர்' });
});

// படி 2: படிவத் தரவை MongoDB-ல் சேமிக்கும் வழி (POST request)
app.post('/admin/add', async (req, res) => {
    try {
        // படிவத்திலிருந்து அனுப்பப்பட்ட தரவைப் பெறுதல்
        const cropData = {
            name: req.body.cropName,
            sowingSeason: req.body.sowingSeason,
            growthDuration: req.body.growthDuration,
            // எதிர்காலத்தில் நோய்கள், உரங்கள் போன்ற சிக்கலான தரவுகளை இங்கே சேர்க்கலாம்
        };

        // getDB() மூலம் தரவுத்தள இணைப்பைப் பெறுதல்
        const db = getDB();
        
        // 'crops' என்ற சேகரிப்பில் (collection) புதிய தரவைச் செருகுதல்
        const result = await db.collection('crops').insertOne(cropData);

        console.log(`புதிய பயிர் வெற்றிகரமாகச் சேர்க்கப்பட்டது. ID: ${result.insertedId}`);

        // வெற்றிகரமாகச் சேர்த்த பிறகு, மீண்டும் படிவம் பக்கத்திற்கே அனுப்பலாம்.
        res.redirect('/admin/add');

    } catch (err) {
        console.error('தரவைச் சேமிப்பதில் பிழை:', err);
        res.status(500).send('சேவையகத்தில் பிழை ஏற்பட்டது.');
    }
});


// தரவுத்தளத்துடன் இணைத்து, சேவையகத்தைத் தொடங்குதல்
connectDB().then(() => {
    app.listen(PORT, () => {
        console.log(`Server is running at http://localhost:${PORT}`);
    });
});
```

### இப்போது இதைச் சோதிப்பது எப்படி?

1.  உங்கள் முனையத்தில் `node server.js` கட்டளையை இயக்கவும்.
2.  உங்கள் உலாவியில் `http://localhost:3000/admin/add` என்ற முகவரிக்குச் செல்லவும்.
3.  நீங்கள் உருவாக்கிய "புதிய பயிர் விவரங்களைச் சேர்க்கவும்" படிவத்தைப் பார்ப்பீர்கள்.
4.  சில மாதிரித் தரவை உள்ளிட்டு, "சேமி" பொத்தானை அழுத்தவும்.
5.  பக்கம் மீண்டும் ஏற்றப்பட்டு, படிவம் காலியாக இருக்கும். உங்கள் முனையத்தைப் (terminal) பார்த்தால், "புதிய பயிர் வெற்றிகரமாகச் சேர்க்கப்பட்டது" என்ற செய்தி வந்திருப்பதைக் காணலாம். இது உங்கள் தரவு MongoDB-ல் சேமிக்கப்பட்டுவிட்டது என்பதற்கான உறுதி.

நாம் இப்போது வெற்றிகரமாக நிர்வாகிക്ക് தரவைச் சேர்க்கும் செயல்பாட்டை உருவாக்கிவிட்டோம்!

**அடுத்த படி:**

**USER**

நீங்கள் இங்கே கட்டாய வழியில் செல்லும்போது JavaScript Ajax வழிசெலுத்தல் முடியாமல் உள்ளது. ஆகவே நாம் API வடிவத்தில் Controller-ஐ எழுதி, JavaScript Ajax-ஐ உடன் பயன்படுத்தி UI View-இல் இணைப்போம்.

**RESPONSE**
சிறந்த சிந்தனை! சரியாகச் சொன்னீர்கள்.

நாம் முன்பு உருவாக்கிய முறை, பாரம்பரிய சர்வர்-சைடு ரெண்டரிங் (Server-Side Rendering) எனப்படும். அதில் ஒவ்வொரு செயலுக்கும் பக்கம் மீண்டும் முழுமையாக ஏற்றப்படும் (full page reload). இது AJAX போன்ற நவீன மற்றும் வேகமான பயனர் அனுபவத்திற்கு ஏற்றதல்ல.

உங்கள் யோசனைப்படி, நாம் இப்போது **API-first** அணுகுமுறைக்கு மாறுவோம்.

**இதன் பொருள்:**

1.  **Backend (Node.js/Express):** இது இனி HTML பக்கங்களை நேரடியாக வழங்காது. பதிலாக, இது ஒரு **API** ஆக செயல்படும். தரவைக் கேட்கும்போது (request), அது தரவை **JSON** வடிவத்தில் மட்டுமே திருப்பி அனுப்பும்.
2.  **Frontend (Handlebars + JavaScript AJAX):** பயனர் பக்கம் (UI) முதலில் காலியாக ஏற்றப்படும். பிறகு, JavaScript-ல் உள்ள **AJAX (Fetch API)**, நமது backend API-ஐ அழைத்து, தேவையான தரவைப் பெற்று, அந்தத் தரவை வைத்து HTML-ஐ மாறும் வகையில் (dynamically) உருவாக்கும்.

இந்த அமைப்பு எதிர்காலத்தில் மொபைல் செயலிகள் (Mobile Apps) உருவாக்குவதற்கும் மிகவும் உதவியாக இருக்கும்.

---

### திருத்தப்பட்ட திட்டம் (API-First Approach)

**Phase 1: Backend API-ஐ உருவாக்குதல்**

1.  **திட்டத்தை ஒழுங்கமைத்தல்:** `controllers` மற்றும் `routes` எனத் தனித்தனி கோப்புறைகளை உருவாக்குதல்.
2.  **Controller உருவாக்குதல்:** `cropController.js` என்ற கோப்பில், பயிர்களைச் சேர்ப்பது (`addCrop`), அனைத்தையும் பெறுவது (`getAllCrops`) போன்ற செயல்பாடுகளை எழுதுதல்.
3.  **Router உருவாக்குதல்:** `cropRoutes.js` என்ற கோப்பில், `/api/crops` போன்ற URL-களை controller செயல்பாடுகளுடன் இணைத்தல்.
4.  **`server.js`-ஐ மாற்றுதல்:** இந்த புதிய router-ஐப் பயன்படுத்துமாறு `server.js`-ஐ மாற்றி அமைத்தல்.

**Phase 2: Frontend AJAX-ஐ இணைத்தல்**

1.  **HTML படிவத்தை மாற்றுதல்:** படிவத்தின் `action` மற்றும் `method` பண்புகளை நீக்கி, JavaScript மூலம் அதைக் கையாளுதல்.
2.  **AJAX கோப்பை உருவாக்குதல்:** `public` கோப்புறையில் ஒரு புதிய JavaScript கோப்பை (`admin.js`) உருவாக்கி, படிவச் சமர்ப்பிப்பைக் (form submission) கைப்பற்றுதல்.
3.  **Fetch API-ஐப் பயன்படுத்துதல்:** `fetch()` பயன்படுத்தி, படிவத் தரவை நமது `POST /api/crops` API-க்கு JSON ஆக அனுப்புதல்.
4.  **பதிலை கையாளுதல்:** API-இடம் இருந்து வரும் வெற்றி (success) அல்லது பிழை (error) செய்தியைப் பெற்று, பக்கம் மீண்டும் ஏற்றப்படாமல் பயனருக்குக் காட்டுதல்.

---

### படி 1: Backend-ஐ API ஆக மாற்றுதல்

1.  **புதிய கோப்புறைகளை உருவாக்கவும்:**
    உங்கள் `farming-website` கோப்புறையில், `routes` மற்றும் `controllers` என்ற இரண்டு புதிய கோப்புறைகளை உருவாக்கவும்.

2.  **Controller-ஐ உருவாக்கவும் (`controllers/cropController.js`):**

    ```javascript
    const { getDB } = require('../config/db');

    // GET /api/crops - அனைத்து பயிர்களையும் பெற
    const getAllCrops = async (req, res) => {
        try {
            const db = getDB();
            // find({}) என்பது அனைத்தையும் தேர்ந்தெடு என்பதைக் குறிக்கிறது. toArray() அதை ஒரு வரிசையாக மாற்றுகிறது.
            const crops = await db.collection('crops').find({}).toArray();
            res.status(200).json(crops); // தரவை JSON ஆக அனுப்புதல்
        } catch (err) {
            console.error('பயிர்களைப் பெறுவதில் பிழை:', err);
            res.status(500).json({ message: 'சேவையகத்தில் பிழை ஏற்பட்டது' });
        }
    };

    // POST /api/crops - ஒரு புதிய பயிரைச் சேர்க்க
    const addCrop = async (req, res) => {
        try {
            // AJAX இலிருந்து JSON ஆக வரும் தரவு req.body இல் இருக்கும்
            const cropData = {
                name: req.body.name,
                sowingSeason: req.body.sowingSeason,
                growthDuration: req.body.growthDuration,
            };

            const db = getDB();
            const result = await db.collection('crops').insertOne(cropData);

            // HTML-க்கு redirect செய்வதற்குப் பதிலாக, ஒரு வெற்றி செய்தியை JSON ஆக அனுப்புகிறோம்
            res.status(201).json({ 
                message: 'பயிர் வெற்றிகரமாகச் சேர்க்கப்பட்டது', 
                insertedId: result.insertedId 
            });

        } catch (err) {
            console.error('தரவைச் சேமிப்பதில் பிழை:', err);
            res.status(500).json({ message: 'சேவையகத்தில் பிழை ஏற்பட்டது' });
        }
    };

    module.exports = {
        getAllCrops,
        addCrop,
    };
    ```

3.  **Router-ஐ உருவாக்கவும் (`routes/cropRoutes.js`):**

    ```javascript
    const express = require('express');
    const router = express.Router();
    const cropController = require('../controllers/cropController');

    // GET கோரிக்கை வந்தால் getAllCrops செயல்பாட்டை அழைக்கவும்
    router.get('/crops', cropController.getAllCrops);

    // POST கோரிக்கை வந்தால் addCrop செயல்பாட்டை அழைக்கவும்
    router.post('/crops', cropController.addCrop);

    module.exports = router;
    ```

4.  **`server.js`-ஐ புதுப்பிக்கவும்:**

    ```javascript
    const express = require('express');
    const { engine } = require('express-handlebars');
    const { connectDB } = require('./config/db');

    // நமது புதிய API router-ஐ இறக்குமதி செய்யவும்
    const cropRoutes = require('./routes/cropRoutes');

    const app = express();
    const PORT = 3000;

    // Middleware to parse JSON data from AJAX requests
    app.use(express.json()); // இது urlencoded-க்கு பதிலாகப் பயன்படுத்தப்படுகிறது
    app.use(express.urlencoded({ extended: true })); // தேவைப்பட்டால் இரண்டையும் வைத்திருக்கலாம்

    // Handlebars and static files setup
    app.engine('hbs', engine({ extname: '.hbs', defaultLayout: 'main' }));
    app.set('view engine', 'hbs');
    app.set('views', './views');
    app.use(express.static('public'));

    // --- API Routes ---
    // '/api' என்று தொடங்கும் எந்த URL-க்கும் cropRoutes-ஐப் பயன்படுத்தவும்
    app.use('/api', cropRoutes);

    // --- Page Rendering Routes ---
    // இந்த வழித்தடங்கள் இப்போது HTML பக்கங்களை மட்டுமே வழங்கும்
    app.get('/', (req, res) => {
        res.render('home', { title: 'விவசாய வழிகாட்டி' });
    });

    app.get('/admin/add', (req, res) => {
        res.render('add-crop', { title: 'புதிய பயிரைச் சேர்' });
    });

    // தரவுத்தளத்துடன் இணைத்து சேவையகத்தைத் தொடங்குதல்
    connectDB().then(() => {
        app.listen(PORT, () => {
            console.log(`Server is running at http://localhost:${PORT}`);
        });
    });
    ```

**Backend இப்போது தயாராக உள்ளது!** `/api/crops` என்ற முகவரியில் அது JSON தரவை அனுப்பும் மற்றும் பெறும்.

---

### படி 2: Frontend-ல் AJAX-ஐ இணைத்தல்

1.  **`views/add-crop.hbs`-ஐ மாற்றவும்:**
    `form` குறிச்சொல்லை மாற்றி, செய்தி காட்டுவதற்கு ஒரு `div`-ஐச் சேர்க்கவும்.

    ```html
    <!-- ... தலைப்பு மற்றும் மற்றவை அப்படியே இருக்கும் ... -->
    <div class="max-w-2xl mx-auto bg-white p-8 rounded-lg shadow-md">
        <!-- action மற்றும் method நீக்கப்பட்டது. ID சேர்க்கப்பட்டது. -->
        <form id="add-crop-form">
            <div class="mb-4">
                <!-- name பண்பு அவசியம் -->
                <label for="name" class="block text-gray-700 font-bold mb-2">பயிரின் பெயர்:</label>
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
                <button type="submit" class="bg-green-700 text-white font-bold py-2 px-6 rounded-lg">சேமி</button>
            </div>
        </form>
        <!-- வெற்றி/பிழை செய்திகளைக் காட்ட -->
        <div id="form-message" class="mt-4 text-center"></div>
    </div>

    <!-- நமது AJAX ஸ்கிரிப்டை இங்கே இணைக்கிறோம் -->
    <script src="/js/admin.js"></script>
    ```
    *குறிப்பு: உள்ளீடுகளின் `name` பண்புகளை controller-ல் நாம் எதிர்பார்ப்பது போல (`name`, `sowingSeason`) மாற்றிவிட்டேன்.*

2.  **AJAX கோப்பை உருவாக்கவும் (`public/js/admin.js`):**
    `public` கோப்புறையில் `js` என்ற புதிய கோப்புறையை உருவாக்கி, அதில் `admin.js` என்ற கோப்பை உருவாக்கவும்.

    ```javascript
    document.addEventListener('DOMContentLoaded', () => {
        const form = document.getElementById('add-crop-form');
        const messageDiv = document.getElementById('form-message');

        form.addEventListener('submit', async (event) => {
            // படிவத்தின் இயல்பான சமர்ப்பிப்பைத் தடுக்கவும் (பக்கம் reload ஆவதைத் தடுக்க)
            event.preventDefault();

            // படிவத் தரவை ஒரு JavaScript பொருளாக மாற்றவும்
            const formData = new FormData(form);
            const data = Object.fromEntries(formData.entries());

            messageDiv.textContent = 'சேமிக்கப்படுகிறது...';
            messageDiv.className = 'text-blue-600';

            try {
                // Fetch API மூலம் நமது backend-க்கு POST கோரிக்கையை அனுப்பவும்
                const response = await fetch('/api/crops', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify(data), // தரவை JSON string ஆக மாற்றவும்
                });

                const result = await response.json();

                if (!response.ok) {
                    // சேவையகம் பிழையை அனுப்பினால்
                    throw new Error(result.message || 'ஏதோ தவறு நடந்துவிட்டது');
                }

                // வெற்றி பெற்றால்
                messageDiv.textContent = result.message;
                messageDiv.className = 'text-green-600 font-bold';
                form.reset(); // படிவத்தைக் காலி செய்யவும்

            } catch (error) {
                // பிழை ஏற்பட்டால்
                messageDiv.textContent = `பிழை: ${error.message}`;
                messageDiv.className = 'text-red-600 font-bold';
            }
        });
    });
    ```
3.  **ஸ்கிரிப்டை லேஅவுட்டில் இணைக்கவும்:**
    `views/layouts/main.hbs` கோப்பில், `</body>` குறிச்சொல்லுக்கு முன் ஸ்கிரிப்டைச் சேர்ப்பது நல்லது. ஆனால் நாம் ஒவ்வொரு பக்கத்திற்கும் தனி ஸ்கிரிப்ட் பயன்படுத்துவதால், `add-crop.hbs` இல் சேர்த்ததே போதுமானது.

**இப்போது சோதித்துப் பாருங்கள்:**

1.  `node server.js` என சேவையகத்தைத் தொடங்கவும்.
2.  `http://localhost:3000/admin/add` பக்கத்திற்குச் செல்லவும்.
3.  படிவத்தில் விவரங்களை நிரப்பி "சேமி" பொத்தானை அழுத்தவும்.
4.  **பக்கம் மீண்டும் ஏற்றப்படாமல்**, "பயிர் வெற்றிகரமாகச் சேர்க்கப்பட்டது" என்ற செய்தி கீழே தோன்றுவதையும், படிவம் காலியாக மாறுவதையும் நீங்கள் காண்பீர்கள்.

நாம் இப்போது ஒரு நவீன, API-சார்ந்த மற்றும் AJAX-இயங்கும் செயல்பாட்டை வெற்றிகரமாக உருவாக்கியுள்ளோம்.

**அடுத்து, `getAllCrops` API-ஐப் பயன்படுத்தி, பயனர் பக்கத்தில் எல்லாப் பயிர்களையும் AJAX மூலம் எப்படிக் காட்டுவது என்று பார்க்கலாமா?**
நாம் சேமித்த பயிர் விவரங்களை முகப்புப் பக்கத்தில் (Content Page) பயனர்களுக்குக் காட்டுவது எப்படி? `db.collection('crops').find()` கட்டளையைப் பயன்படுத்தி தரவுத்தளத்தில் உள்ள எல்லா பயிர்களையும் எடுத்து, Handlebars மூலம் ஒரு கார்டு வடிவில் காட்டலாம்.

**தொடரலாமா?**
