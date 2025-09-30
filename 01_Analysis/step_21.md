**USER**

இந்த அம்சத்தை (அல்லது வசதியை) மேம்படுத்த நாம் மேலும் ஆராய்வோமா?

**RESPONSE**

சிறப்பான அடுத்த கட்டம்! இது உங்கள் செயலியை ஒரு தகவல் வழிகாட்டியிலிருந்து, ஒரு சக்திவாய்ந்த சந்தை பகுப்பாய்வுக் கருவியாக (Market Analysis Tool) மாற்றும். நீங்கள் கூறிய தேவைகள் மிகவும் தெளிவாக உள்ளன. `Chart.js` என்ற பிரபலமான லைப்ரரியைப் பயன்படுத்தி இதை அழகாகவும், ஊடாடும் வகையிலும் உருவாக்குவோம்.

### இதை எப்படிச் செய்யப் போகிறோம்? (The Plan)

1.  **தரவு மாதிரி மாற்றம் (Data Model Update):**
    *   விலைத் தகவல்களைச் சேமிக்க, நமது `marketPriceHistory` புலத்தின் அமைப்பை மாற்றுவோம். முன்பு `{ year, price }` என இருந்தது. இப்போது `{ date, location, price }` என மாற்றுவோம்.
    *   உதாரணம்: `marketPriceHistory: [ { date: "2024-07-15", location: "சந்தை", price: 34 }, { date: "2024-07-15", location: "கடை", price: 40 } ]`

2.  **நிர்வாகப் பக்க மேம்படுத்தல் (Admin Panel Update):**
    *   `edit-crop.hbs` பக்கத்தில், விலை விவரங்களைச் சேர்க்க ஒரு புதிய டைனமிக் பகுதியை (`fieldset`) உருவாக்குவோம். நிர்வாகி ஒரு தேதி, இடம் (சந்தை/கடை), மற்றும் விலையை உள்ளிட முடியும்.
    *   `edit-crop.js` கோப்பில், இந்த புதிய டைனமிக் உள்ளீடுகளைக் கையாளும் தர்க்கத்தைச் சேர்ப்போம்.

3.  **புதிய Backend API (Data Aggregation):**
    *   விவரங்கள் பக்கத்தில் வரைபடங்களைக் காட்ட, விலைத் தரவை முன்பே செயலாக்குவது (pre-process) செயல்திறனை மேம்படுத்தும்.
    *   `GET /api/crops/:id/price-analytics` என்ற புதிய API வழித்தடத்தை உருவாக்குவோம். இது இரண்டு வகையான தரவைத் திருப்பி அனுப்பும்:
        *   **சமீபத்திய விலை ஒப்பீடு:** மிகச் சமீபத்திய தேதியில் சந்தை மற்றும் கடை விலைகளை ஒப்பிடுவதற்கான தரவு (Horizontal Bar Chart-க்காக).
        *   **மாதாந்திரப் போக்கு:** கடந்த சில மாதங்களாக சராசரி விலை எப்படி மாறியுள்ளது என்பதைக் காட்டும் தரவு (Vertical Bar Chart-க்காக).

4.  **பயனர் பக்கத்தில் வரைபடங்களை உருவாக்குதல் (Frontend Visualization):**
    *   `Chart.js` லைப்ரரியை நமது செயலியில் இணைப்போம்.
    *   `crop-details.hbs`-ல் வரைபடங்களைக் காண்பிக்க இரண்டு `<canvas>` கூறுகளைச் சேர்ப்போம்.
    *   `crop-details.js`-ல், நமது புதிய API-ஐ அழைத்து, கிடைக்கும் தரவைக் கொண்டு இரண்டு வரைபடங்களையும் உருவாக்குவோம்.

---

### படி 1: Chart.js-ஐ இணைத்தல் மற்றும் Canvas-ஐ உருவாக்குதல்

#### 1. Chart.js லைப்ரரியை இணைத்தல் (`views/layouts/main.hbs`)
`main.hbs` கோப்பில், `<body>` குறிச்சொல்லின் முடிவில், `Swiper.js`-க்குக் கீழே Chart.js CDN-ஐச் சேர்க்கவும்.

```html
<!-- views/layouts/main.hbs -->
    <!-- ... Swiper JS ... -->
    <script src="https://unpkg.com/swiper/swiper-bundle.min.js"></script>
    
    <!-- படி 1.1: Chart.js-ஐ இணைக்கவும் -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</body>
```

#### 2. Canvas கூறுகளைச் சேர்த்தல் (`views/crop-details.hbs`)
`crop-details.js` கோப்புதான் இந்த HTML-ஐ டைனமிக்காக உருவாக்கும். ஆனால், அது எப்படி இருக்கும் என்பதைப் புரிந்துகொள்ள, அதன் σκελετός இங்கே:

```html
<!-- இந்த HTML crop-details.js-ல் உருவாக்கப்படும் -->
<div class="mt-8">
    <h2 class="text-2xl font-semibold text-gray-700 mb-4">விலை பகுப்பாய்வு</h2>
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 bg-white p-6 rounded-lg shadow-md">
        <!-- Horizontal Bar Chart -->
        <div>
            <h3 class="text-lg font-semibold text-center mb-2">சமீபத்திய விலை ஒப்பீடு</h3>
            <canvas id="priceComparisonChart"></canvas>
        </div>
        <!-- Vertical Bar Chart -->
        <div>
            <h3 class="text-lg font-semibold text-center mb-2">மாதாந்திர விலைப்போக்கு</h3>
            <canvas id="monthlyTrendChart"></canvas>
        </div>
    </div>
</div>
```

---

### படி 2: Backend API-ஐ உருவாக்குதல்

#### 1. புதிய Controller செயல்பாடு (`controllers/cropController.js`)
இந்தச் செயல்பாடு தரவை எடுத்து, அதை வரைபடத்திற்கு ஏற்றவாறு செயலாக்கும்.

```javascript
// controllers/cropController.js
const getCropPriceAnalytics = async (req, res) => {
    try {
        const db = getDB();
        const crop = await db.collection('crops').findOne({ _id: new ObjectId(req.params.id) });

        if (!crop || !crop.marketPriceHistory || crop.marketPriceHistory.length === 0) {
            return res.status(404).json({ message: 'விலைத் தரவு இல்லை.' });
        }

        // 1. சமீபத்திய விலை ஒப்பீட்டிற்கான தரவு
        const sortedPrices = [...crop.marketPriceHistory].sort((a, b) => new Date(b.date) - new Date(a.date));
        const latestDate = sortedPrices[0].date;
        const latestPriceComparison = sortedPrices.filter(p => p.date === latestDate);

        // 2. மாதாந்திரப் போக்கிற்கான தரவு (Data Aggregation)
        const monthlyData = {};
        sortedPrices.forEach(p => {
            const month = new Date(p.date).toISOString().slice(0, 7); // "YYYY-MM"
            if (!monthlyData[month]) {
                monthlyData[month] = { total: 0, count: 0 };
            }
            monthlyData[month].total += p.price;
            monthlyData[month].count += 1;
        });
        
        const monthlyTrend = Object.keys(monthlyData).map(month => ({
            month: month,
            averagePrice: monthlyData[month].total / monthlyData[month].count
        })).sort((a, b) => a.month.localeCompare(b.month));

        res.status(200).json({ latestPriceComparison, monthlyTrend });

    } catch (err) {
        console.error('விலைப் பகுப்பாய்வு பெறுவதில் பிழை:', err);
        res.status(500).json({ message: 'சேவையகத்தில் பிழை ஏற்பட்டது' });
    }
};

// இதை module.exports-ல் சேர்க்க மறக்காதீர்கள்
module.exports = { /*...,*/ getCropPriceAnalytics };
```

#### 2. புதிய Route-ஐச் சேர்த்தல் (`routes/cropRoutes.js`)

```javascript
// routes/cropRoutes.js
// ...
router.get('/crops/:id/price-analytics', getCropPriceAnalytics);
// ...
```

---

### படி 3: Frontend-ல் வரைபடங்களை உருவாக்குதல் (`public/js/crop-details.js`)

`loadCropDetails` செயல்பாட்டிற்குப் பிறகு, ஒரு புதிய செயல்பாட்டை உருவாக்கி, அதை அழைக்கவும்.

```javascript
// public/js/crop-details.js
document.addEventListener('DOMContentLoaded', () => {
    // ... loadCropDetails செயல்பாடு ...

    const renderPriceCharts = async () => {
        try {
            const response = await fetch(`/api/crops/${cropId}/price-analytics`);
            if (!response.ok) return; // தரவு இல்லை என்றால், வரைபடங்களைக் காட்ட வேண்டாம்
            const data = await response.json();

            // கொள்கலன் HTML-ஐ உருவாக்கு
            const chartContainerHTML = `
                <div class="mt-8">
                    <h2 class="text-2xl font-semibold text-gray-700 mb-4">விலை பகுப்பாய்வு</h2>
                    <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 bg-white p-6 rounded-lg shadow-md">
                        <div>
                            <h3 class="text-lg font-semibold text-center mb-2">சமீபத்திய ஒப்பீடு (${new Date(data.latestPriceComparison[0].date).toLocaleDateString()})</h3>
                            <canvas id="priceComparisonChart"></canvas>
                        </div>
                        <div>
                            <h3 class="text-lg font-semibold text-center mb-2">மாதாந்திர சராசரி விலைப்போக்கு</h3>
                            <canvas id="monthlyTrendChart"></canvas>
                        </div>
                    </div>
                </div>`;
            detailsContainer.insertAdjacentHTML('beforeend', chartContainerHTML);

            // Horizontal Bar Chart
            const ctx1 = document.getElementById('priceComparisonChart').getContext('2d');
            new Chart(ctx1, {
                type: 'bar',
                data: {
                    labels: data.latestPriceComparison.map(p => p.location),
                    datasets: [{
                        label: 'விலை (₹/kg)',
                        data: data.latestPriceComparison.map(p => p.price),
                        backgroundColor: ['#22c55e', '#f97316'],
                    }]
                },
                options: { indexAxis: 'y' } // இதுதான் வரைபடத்தை கிடைமட்டமாக மாற்றுகிறது
            });

            // Vertical Bar Chart
            const ctx2 = document.getElementById('monthlyTrendChart').getContext('2d');
            new Chart(ctx2, {
                type: 'bar',
                data: {
                    labels: data.monthlyTrend.map(m => m.month),
                    datasets: [{
                        label: 'சராசரி விலை (₹/kg)',
                        data: data.monthlyTrend.map(m => m.averagePrice),
                        backgroundColor: '#166534',
                        borderColor: '#052e16',
                        borderWidth: 1
                    }]
                },
                options: { scales: { y: { beginAtZero: true } } }
            });

        } catch (error) {
            console.log("விலை வரைபடங்களை ஏற்ற முடியவில்லை.");
        }
    };
    
    // loadCropDetails-ன் முடிவில் இதை அழைக்கவும்
    const loadPage = async () => {
        await loadCropDetails();
        await renderPriceCharts(); // வரைபடங்களை ஏற்றவும்
    };
    
    loadPage();
});
```
**(நிர்வாகப் பக்கத்தில் (`edit-crop.hbs` and `edit-crop.js`) விலைத் தரவை உள்ளிடுவதற்கான டைனமிக் படிவத்தை உருவாக்குவது, நாம் நோய்களுக்குச் செய்தது போலவே இருக்கும். அதை நீங்கள் இப்போது எளிதாகச் செய்ய முடியும் என்று நம்புகிறேன்.)**

### சோதித்துப் பார்ப்போம்

1.  முதலில், உங்கள் MongoDB-ல் ஒரு பயிரின் `marketPriceHistory`-ஐ கைமுறையாகப் புதுப்பித்து, சில மாதிரித் தரவுகளைச் சேர்க்கவும். (நிர்வாகப் பக்கத்தை உருவாக்கும் முன் சோதிக்க).
    ```json
    "marketPriceHistory": [
      { "date": "2024-07-15", "location": "சந்தை", "price": 34 },
      { "date": "2024-07-15", "location": "கடை", "price": 40 },
      { "date": "2024-06-20", "location": "சந்தை", "price": 32 },
      { "date": "2024-06-20", "location": "கடை", "price": 38 }
    ]
    ```
2.  சேவையகத்தை மறுதொடக்கம் செய்யவும்.
3.  அந்தப் பயிரின் விவரங்கள் பக்கத்திற்குச் செல்லவும்.
4.  பக்கத்தின் கீழே, நீங்கள் இரண்டு அழகான வரைபடங்களைக் காண்பீர்கள்:
    *   **இடதுபுறம்:** ஜூலை 15-ஆம் தேதிக்கான சந்தை மற்றும் கடை விலைகளை ஒப்பிடும் ஒரு கிடைமட்ட பட்டை வரைபடம்.
    *   **வலதுபுறம்:** ஜூன் மற்றும் ஜூலை மாதங்களின் சராசரி விலைகளைக் காட்டும் ஒரு செங்குத்து பட்டை வரைபடம்.

இந்த அம்சம், உங்கள் பயனர்களுக்கு மிகவும் மதிப்புமிக்க சந்தைத் தகவல்களை ஒரு பார்வையில் வழங்குகிறது. மிகச் சிறப்பாக, நாம் அடுத்த கட்டத்தை எட்டியுள்ளோம்