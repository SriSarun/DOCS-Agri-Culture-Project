**USER** Yes

**RESPONDE**
அருமை, தொடங்குவோம்!

நிர்வாகப் பக்கத்தில் (Admin Panel) ஒரு படிவத்தை உருவாக்கி, அதன் மூலம் பெறும் தரவுகளை நேட்டிவ் MongoDB கட்டளைகளைப் பயன்படுத்தி சேமிப்பதற்கான படிநிலைகள் கீழே கொடுக்கப்பட்டுள்ளன.

### படி 1: பயிர் விவரங்களை உள்ளிட ஒரு HTML படிவத்தை உருவாக்குதல்

முதலில், நிர்வாகி புதிய பயிர் விவரங்களை உள்ளிடுவதற்கான ஒரு பக்கத்தை உருவாக்குவோம். Handlebars ஐப் பயன்படுத்தி இதைச் செய்வோம்.

1.  உங்கள் `views` கோப்புறையில் `add-crop.hbs` என்ற புதிய கோப்பை உருவாக்கவும்.
2.  அந்தக் கோப்பில், கீழே உள்ள HTML குறியீட்டைச் சேர்க்கவும். இது Tailwind CSS ஐப் பயன்படுத்தி ஒரு எளிமையான படிவத்தை உருவாக்கும்.

**`views/add-crop.hbs`**
```html
<div class="max-w-2xl mx-auto bg-white p-8 rounded-lg shadow-md mt-10">
    <h1 class="text-2xl font-bold text-green-800 mb-6 text-center">புதிய பயிர் விவரங்களைச் சேர்க்கவும்</h1>

    <form action="/admin/add" method="POST">
        <!-- பயிரின் பெயர் -->
        <div class="mb-4">
            <label for="cropName" class="block text-gray-700 font-semibold mb-2">பயிரின் பெயர்</label>
            <input type="text" id="cropName" name="cropName" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-green-500" required>
        </div>

        <!-- விதைப்பு காலம் -->
        <div class="mb-4">
            <label for="plantationSeason" class="block text-gray-700 font-semibold mb-2">விதைப்பு காலம் (எ.கா: ஆடி மாசம்)</label>
            <input type="text" id="plantationSeason" name="plantationSeason" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-green-500" required>
        </div>

        <!-- வளர்ச்சி நாட்கள் -->
        <div class="mb-6">
            <label for="growthDuration" class="block text-gray-700 font-semibold mb-2">வளர்ச்சி காலம் (நாட்கள்)</label>
            <input type="text" id="growthDuration" name="growthDuration" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-green-500" placeholder="எ.கா: 120-135">
        </div>

        <!-- சமர்ப்பி பொத்தான் -->
        <div class="text-center">
            <button type="submit" class="bg-green-700 text-white font-bold py-2 px-6 rounded-lg hover:bg-green-800 transition duration-300">
                சேமி
            </button>
        </div>
    </form>
</div>
```
**முக்கிய குறிப்புகள்:**
*   `<form action="/admin/add" method="POST">`: இந்தப் படிவம் சமர்ப்பிக்கப்படும்போது, `/admin/add` என்ற URL க்கு `POST` கோரிக்கையை அனுப்பும் என்று இது கூறுகிறது.
*   `name="cropName"`: ஒவ்வொரு உள்ளீட்டிற்கும் `name` பண்பு மிக முக்கியம். இதை வைத்துத்தான் Node.js இல் நாம் தரவைப் பெறுவோம்.

### படி 2: படிவத்தைக் கையாள்வதற்கான Routes ஐ உருவாக்குதல்

இப்போது, `server.js` கோப்பில் இரண்டு புதிய Routes களைச் சேர்க்க வேண்டும்:
1.  **GET `/admin/add`**: மேலே உருவாக்கிய படிவப் பக்கத்தைக் காண்பிப்பதற்காக.
2.  **POST `/admin/add`**: படிவத்திலிருந்து வரும் தரவைப் பெற்று, அதை MongoDB இல் சேமிப்பதற்காக.

உங்கள் `server.js` கோப்பை பின்வருமாறு மாற்றி அமைக்கவும்:

**`server.js`**
```javascript
const express = require('express');
const { engine } = require('express-handlebars');
const { connectDB, getDB } = require('./config/db');

const app = express();
const PORT = 3000;

// Middleware to parse form data
// படிவத் தரவைப் படிக்க இந்த middleware அவசியம்
app.use(express.urlencoded({ extended: true }));

// Handlebars and static files setup... (இது ஏற்கனவே இருக்கும்)
app.engine('hbs', engine({ extname: '.hbs', defaultLayout: 'main' }));
app.set('view engine', 'hbs');
app.set('views', './views');
app.use(express.static('public'));


// --- புதிய Routes ---

// 1. படிவப் பக்கத்தைக் காண்பிப்பதற்கான GET route
app.get('/admin/add', (req, res) => {
    res.render('add-crop', { title: 'புதிய பயிர் சேர்' });
});

// 2. படிவத் தரவை MongoDB இல் சேமிப்பதற்கான POST route
app.post('/admin/add', async (req, res) => {
    try {
        // படிவத்தில் இருந்து வரும் தரவை req.body இல் இருந்து பெறுகிறோம்
        const cropData = req.body;
        console.log('படிவத்திலிருந்து பெறப்பட்ட தரவு:', cropData);

        // தரவை ஒரு αντικειμένουவாக வடிவமைத்தல்
        const newCrop = {
            name: cropData.cropName,
            season: cropData.plantationSeason,
            duration: cropData.growthDuration,
            // எதிர்காலத்தில் நோய்கள், உரங்கள் போன்றவற்றை இங்கே சேர்க்கலாம்
            // diseases: [],
            createdAt: new Date() // தரவு உருவாக்கப்பட்ட நேரத்தைச் சேமித்தல்
        };

        // தரவுத்தள இணைப்பைப் பெற்று, 'crops' கலெக்ஷனைத் தேர்ந்தெடுக்கவும்
        const db = getDB();
        const collection = db.collection('crops');

        // நேட்டிவ் insertOne கட்டளையைப் பயன்படுத்தி தரவைச் செருகவும்
        const result = await collection.insertOne(newCrop);
        console.log(`${result.insertedId} உடன் புதிய பயிர் வெற்றிகரமாகச் சேர்க்கப்பட்டது.`);

        // வெற்றிகரமாகச் சேர்த்த பிறகு, முகப்புப் பக்கத்திற்குத் திருப்பி விடவும்
        res.redirect('/');

    } catch (err) {
        console.error('தரவைச் சேமிப்பதில் பிழை:', err);
        res.status(500).send('சேவையகத்தில் பிழை ஏற்பட்டது.');
    }
});


// --- பழைய Route ---

app.get('/', (req, res) => {
    res.render('home', { title: 'விவசாய வழிகாட்டி' });
});


// --- சேவையகத்தைத் தொடங்குதல் ---

connectDB().then(() => {
    app.listen(PORT, () => {
        console.log(`Server is running at http://localhost:${PORT}`);
    });
});
```

### என்ன செய்துள்ளோம்?

1.  **`express.urlencoded` Middleware:** படிவத்திலிருந்து `POST` செய்யப்படும் தரவை `req.body` என்ற αντικειμένουவில் எளிதாகப் படிக்க இந்த வரி (`app.use(...)`) உதவுகிறது. இது ஒரு απαραίτητο படியாகும்.
2.  **GET Route (`/admin/add`):** ஒரு பயனர் `http://localhost:3000/admin/add` என்ற URL க்குச் சென்றால், நாம் உருவாக்கிய `add-crop.hbs` கோப்பை இது காண்பிக்கும்.
3.  **POST Route (`/admin/add`):**
    *   இது ஒரு `async` செயல்பாடு, કારણ કે தரவுத்தள செயல்பாடுகள் நேரம் எடுக்கும்.
    *   `req.body`-இலிருந்து தரவை எடுத்து, அதை `newCrop` என்ற ஒரு நேர்த்தியான αντικειμένουவாக மாற்றுகிறோம்.
    *   `getDB()` மூலம் தரவுத்தளத்தைப் பெற்று, `crops` என்ற கலெக்ஷனைத் தேர்ந்தெடுக்கிறோம் (கலெக்ஷன் இல்லை என்றால், MongoDB தானாகவே அதை உருவாக்கும்).
    *   `collection.insertOne(newCrop)` என்ற நேட்டிவ் MongoDB கட்டளையைப் பயன்படுத்தி அந்த αντικειμένουவை தரவுத்தளத்தில் சேமிக்கிறோம்.
    *   வெற்றி பெற்றால், பயனரை முகப்புப் பக்கத்திற்கு (`/`) அனுப்புகிறோம்.

### இப்போது இதை எப்படிச் சோதிப்பது?

1.  உங்கள் சேவையகத்தை மீண்டும் தொடங்கவும் (Ctrl+C அழுத்தி நிறுத்திவிட்டு, மீண்டும் `node server.js` என இயக்கவும்).
2.  உங்கள் உலாவியில் `http://localhost:3000/admin/add` என்ற முகவரிக்குச் செல்லவும்.
3.  நீங்கள் உருவாக்கிய படிவம் தெரியும். அதில் "மாப்பிள்ளை சம்பா", "ஆடி மாசம்", "120-135 நாட்கள்" போன்ற விவரங்களை உள்ளிட்டு "சேமி" பொத்தானை அழுத்தவும்.
4.  பக்கம் முகப்புப் பக்கத்திற்குத் திரும்பினால், உங்கள் தரவு வெற்றிகரமாகச் சேமிக்கப்பட்டுவிட்டது என்று அர்த்தம்!

**சரிபார்ப்பது எப்படி?**
உங்கள் முனையத்தில் (terminal) "New crop added successfully..." என்ற செய்தி வந்திருப்பதைப் பார்க்கலாம். மேலும், MongoDB Compass அல்லது `mongosh` கட்டளை வரி கருவியைப் பயன்படுத்தி `farmingDB` தரவுத்தளத்தில் உள்ள `crops` கலெக்ஷனைப் பார்த்தால், நீங்கள் உள்ளிட்ட தரவு ஒரு புதிய ஆவணமாக (document) சேமிக்கப்பட்டிருப்பதை உறுதி செய்யலாம்.

அடுத்து, நாம் சேர்த்த இந்த பயிர் விவரங்களை முகப்புப் பக்கத்தில் (homepage) ஒரு கார்டு வடிவில் காட்டலாமா?
