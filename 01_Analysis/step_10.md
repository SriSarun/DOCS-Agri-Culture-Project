நிச்சயமாக! நாம் இப்போது இந்தச் செயலின் ஒரு மிக முக்கியமான, தொழில்முறை அம்சத்தை ஆராய்வோம்: **தரவு விதைத்தல் (Data Seeding).**

"மாப்பிள்ளை சம்பா" தரவை நாம் கைமுறையாக (manually) MongoDB Compass மூலம் செருகினோம். இது ஒரு பயிருக்குச் சரி. ஆனால், நாம் செயலியை உருவாக்கும்போது, பத்து அல்லது இருபது பயிர்களின் மாதிரித் தரவுகளுடன் சோதிக்க வேண்டியிருக்கும். ஒவ்வொரு முறையும் அவற்றைக் கைமுறையாகச் சேர்ப்பது மிகவும் கடினம்.

### தரவு விதைத்தல் (Data Seeding) என்றால் என்ன?

இது, நமது செயலியை முதன்முதலில் தொடங்கும்போதோ அல்லது சோதனை செய்வதற்காகவோ, தேவையான ஆரம்பத் தரவுகளை (initial data) ஒரு கோப்பிலிருந்து (file) படித்து, தானாகவே தரவுத்தளத்தில் நிரப்பும் ஒரு செயல்முறையாகும்.

**இதன் நன்மைகள்:**
*   **மீண்டும் பயன்படுத்தலாம்:** ஒரே கட்டளையில் எல்லா மாதிரித் தரவுகளையும் நீக்கலாம் அல்லது சேர்க்கலாம்.
*   **சுத்தமான தொடக்கம்:** ஒவ்வொரு சோதனைக்கும் முன், பழைய தரவை அழித்துவிட்டு, புதிய, சுத்தமான தரவுகளுடன் தொடங்கலாம்.
*   **குழுப்பணிக்கு எளிது:** குழுவில் உள்ள மற்ற டெவலப்பர்கள் தங்கள் கணினியில் திட்டத்தை அமைக்கும்போது, மாதிரித் தரவுகளைப் பெற இந்த ஸ்கிரிப்டை இயக்கினால் போதும்.

---

### படி 1: விதை தரவுக் கோப்பை உருவாக்குதல்

முதலில், நமது எல்லா மாதிரிப் பயிர்களின் விவரங்களையும் வைத்திருக்க ஒரு JSON கோப்பை உருவாக்குவோம்.

1.  உங்கள் திட்டத்தின் மூலக் கோப்புறையில் (root folder) `data` என்ற புதிய கோப்புறையை உருவாக்கவும்.
2.  அந்த `data` கோப்புறையில், `crops.json` என்ற புதிய கோப்பை உருவாக்கவும்.
3.  இந்த `crops.json` கோப்பில், "மாப்பிள்ளை சம்பா" மற்றும் உதாரணமாக "கிச்சிலி சம்பா" ஆகிய இரண்டு பயிர்களின் தரவை ஒரு JSON வரிசையாக (array) வைப்போம்.

**`data/crops.json`**
```json
[
    {
        "name": "மாப்பிள்ளை சம்பா",
        "sowingSeason": "ஆடி மாசம் (July-August)",
        "growthDuration": "120-135 நாட்கள்",
        "growthStages": [
            { "stageName": "நாற்றங்கால் (Nursery)", "durationDays": "0-25" },
            { "stageName": "வளர்ச்சிப் பருவம் (Vegetative)", "durationDays": "26-70" },
            { "stageName": "பூக்கும் பருவம் (Flowering)", "durationDays": "71-100" },
            { "stageName": "அறுவடை (Harvest)", "durationDays": "101-135" }
        ],
        "diseases": [
            {
                "diseaseName": "குலை நோய் (Blast)",
                "symptoms": "இலை மீது மஞ்சள் மையம் மற்றும் கருப்பு ஓரங்கள் கொண்ட புள்ளிகள்.",
                "causes": ["அதிக ஈரப்பதம்", "அடர்த்தியான விதைப்பு"],
                "chemicalControl": "Tricyclazole 75 WP",
                "organicControl": "நீம் எண்ணெய் 5%"
            },
            {
                "diseaseName": "பழுப்பு புள்ளி நோய் (Brown Spot)",
                "symptoms": "இலைகளில் சிறிய கருப்பு புள்ளிகள்.",
                "causes": ["பொட்டாசியம் குறைபாடு"],
                "chemicalControl": "Mancozeb 75 WP",
                "organicControl": "каоலின் (Kaolin) தெளிப்பு"
            }
        ],
        "fertilizerRecommendation": {
            "general": [{ "stage": 2, "fertilizer": "DAP", "quantityPerAcre": "25kg" }],
            "organic": [{ "name": "ஜீவாமிர்தம்", "quantityPerAcre": "20 லிட்டர்" }]
        },
        "marketPriceHistory": [{ "year": 2023, "pricePerKg": 32 }]
    },
    {
        "name": "கிச்சிலி சம்பா",
        "sowingSeason": "புரட்டாசி பட்டம் (September-October)",
        "growthDuration": "110-120 நாட்கள்",
        "growthStages": [
            { "stageName": "நாற்றங்கால்", "durationDays": "0-20" },
            { "stageName": "வளர்ச்சி", "durationDays": "21-60" },
            { "stageName": "பூக்கும் பருவம்", "durationDays": "61-90" },
            { "stageName": "அறுவடை", "durationDays": "91-120" }
        ],
        "diseases": [
            {
                "diseaseName": "துங்ரோ வைரஸ் (Tungro Virus)",
                "symptoms": "இலைகள் மஞ்சள் அல்லது ஆரஞ்சு நிறமாக மாறுதல், வளர்ச்சி குன்றுதல்.",
                "causes": ["பச்சை தத்துப்பூச்சி மூலம் பரவுகிறது"],
                "chemicalControl": "Thiamethoxam 25 WG",
                "organicControl": "மஞ்சள் ஒட்டுப்பொறிகள்"
            }
        ],
        "fertilizerRecommendation": {
            "general": [{ "stage": 2, "fertilizer": "Urea", "quantityPerAcre": "20kg" }],
            "organic": [{ "name": "பன்சகவ்யம்", "quantityPerAcre": "10 லிட்டர்" }]
        },
        "marketPriceHistory": [{ "year": 2023, "pricePerKg": 40 }]
    }
]
```

---

### படி 2: விதைக்கும் ஸ்கிரிப்டை (Seeder Script) உருவாக்குதல்

இப்போது, இந்த `crops.json` கோப்பைப் படித்து, தரவை MongoDB-ல் சேர்க்கும் அல்லது நீக்கும் ஒரு தனி Node.js ஸ்கிரிப்டை உருவாக்குவோம்.

1.  உங்கள் திட்டத்தின் மூலக் கோப்புறையில் `seeder.js` என்ற புதிய கோப்பை உருவாக்கவும்.

**`seeder.js`**
```javascript
const fs = require('fs');
const { MongoClient } = require('mongodb');

// இணைப்பு URL மற்றும் தரவுத்தள பெயர்
const url = 'mongodb://localhost:27017';
const dbName = 'farmingDB';

// தரவுத்தளத்துடன் இணைக்க ஒரு கிளையண்ட்
const client = new MongoClient(url);

// JSON கோப்பைப் படித்து JavaScript பொருளாக மாற்று
const crops = JSON.parse(
  fs.readFileSync(`${__dirname}/data/crops.json`, 'utf-8')
);

// தரவை இறக்குமதி செய்யும் செயல்பாடு
const importData = async () => {
  try {
    await client.connect();
    const db = client.db(dbName);
    const cropsCollection = db.collection('crops');

    // 1. பழைய பயிர்களை நீக்கு (duplicates-ஐத் தவிர்க்க)
    await cropsCollection.deleteMany();

    // 2. JSON கோப்பிலிருந்து புதிய பயிர்களைச் சேர்
    await cropsCollection.insertMany(crops);

    console.log('\x1b[32m%s\x1b[0m', '✅ தரவு வெற்றிகரமாக இறக்குமதி செய்யப்பட்டது!'); // பச்சை நிறத்தில் செய்தி
    await client.close();
  } catch (error) {
    console.error('\x1b[31m%s\x1b[0m', `❌ பிழை: ${error.message}`); // சிவப்பு நிறத்தில் செய்தி
    process.exit(1);
  }
};

// தரவை நீக்கும் செயல்பாடு
const deleteData = async () => {
  try {
    await client.connect();
    const db = client.db(dbName);
    const cropsCollection = db.collection('crops');

    // எல்லா பயிர்களையும் நீக்கு
    await cropsCollection.deleteMany();

    console.log('\x1b[33m%s\x1b[0m', '🗑️ தரவு வெற்றிகரமாக நீக்கப்பட்டது!'); // மஞ்சள் நிறத்தில் செய்தி
    await client.close();
  } catch (error) {
    console.error('\x1b[31m%s\x1b[0m', `❌ பிழை: ${error.message}`);
    process.exit(1);
  }
};

// கட்டளை வரி வாதங்களை (command line arguments) சரிபார்
if (process.argv[2] === '-i') {
  importData();
} else if (process.argv[2] === '-d') {
  deleteData();
} else {
  console.log('செயல்பாட்டைத் தேர்ந்தெடுக்கவும்: -i (இறக்குமதி செய்ய) அல்லது -d (நீக்க)');
}
```

---

### படி 3: `package.json`-ல் ஸ்கிரிப்ட்களைச் சேர்த்தல்

ஒவ்வொரு முறையும் `node seeder.js -i` என்று தட்டச்சு செய்வதற்குப் பதிலாக, `npm`-ல் எளிதான குறுக்குவழிகளை உருவாக்கலாம்.

உங்கள் `package.json` கோப்பைத் திறந்து, `scripts` பகுதிக்குள் இந்த இரண்டு வரிகளைச் சேர்க்கவும்:

```json
// package.json
"scripts": {
    "start": "node server.js",
    "data:import": "node seeder -i",
    "data:destroy": "node seeder -d"
  },
```

---

### இப்போது இதை எப்படிப் பயன்படுத்துவது?

1.  உங்கள் முனையத்தில் (terminal), நீங்கள் கைமுறையாகச் சேர்த்த "மாப்பிள்ளை சம்பா" மற்றும் வேறு ஏதேனும் பயிர்களை முதலில் அழிப்போம்.
    ```bash
    npm run data:destroy
    ```
    "🗑️ தரவு வெற்றிகரமாக நீக்கப்பட்டது!" என்ற செய்தி வரும்.

2.  இப்போது, நமது `crops.json` கோப்பில் உள்ள இரண்டு பயிர்களையும் தரவுத்தளத்தில் விதைப்போம்.
    ```bash
    npm run data:import
    ```
    "✅ தரவு வெற்றிகரமாக இறக்குமதி செய்யப்பட்டது!" என்ற செய்தி வரும்.

3.  இப்போது, உங்கள் செயலியைத் தொடங்கவும்:
    ```bash
    npm start
    ```
    `http://localhost:3000` முகவரிக்குச் சென்றால், முகப்புப் பக்கத்தில் "மாப்பிள்ளை சம்பா" மற்றும் "கிச்சிலி சம்பா" ஆகிய இரண்டு கார்டுகளையும் நீங்கள் காண்பீர்கள். "மாப்பிள்ளை சம்பா"வைக் கிளிக் செய்தால், அதன் முழு விவரங்களும் இப்போது காட்டப்படும்.

நாம் இப்போது ஒரு சக்திவாய்ந்த, மீண்டும் பயன்படுத்தக்கூடிய தரவு விதைப்பு முறையை உருவாக்கியுள்ளோம். இது உங்கள் மேம்பாட்டு செயல்முறையை மிகவும் எளிதாகவும், வேகமாகவும் மாற்றும்.

