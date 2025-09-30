நிச்சயமாக, இதிலிருந்து தொடங்குவோம்!

"தேதி வரம்பு வடிகட்டி" (Date Range Filter) அம்சத்தைச் சேர்ப்பது, நமது வரைபடங்களை உடனடியாக மிகவும் சக்திவாய்ந்ததாக மாற்றும். பயனர்கள் தங்களுக்கு விருப்பமான காலகட்டத்தில் விலை மாற்றங்களை ஆராய இது வழிவகுக்கும்.

இதை மூன்று முக்கியப் படிகளில் செயல்படுத்துவோம்:

1.  **UI மாற்றம் (Frontend):** விவரங்கள் பக்கத்தில் "தொடக்கத் தேதி" மற்றும் "இறுதித் தேதி" ஆகியவற்றைத் தேர்ந்தெடுப்பதற்கான உள்ளீடுகளைச் சேர்ப்போம்.
2.  **API மேம்படுத்தல் (Backend):** `getCropPriceAnalytics` API, `startDate` மற்றும் `endDate` என்ற அளவுருக்களை ஏற்று, அந்தத் தேதி வரம்பிற்குள் உள்ள தரவை மட்டும் திருப்பி அனுப்புமாறு மாற்றுவோம்.
3.  **JavaScript ஒருங்கிணைப்பு (Frontend):** பயனர் தேதிகளைத் தேர்ந்தெடுத்து "வடிகட்டு" பொத்தானை அழுத்தும்போது, JavaScript ஆனது பழைய வரைபடங்களை அழித்துவிட்டு, புதிய தரவைக் கொண்டு புதுப்பிக்கும்.

---

### படி 1: Frontend UI-ல் தேதி உள்ளீடுகளைச் சேர்த்தல்

`public/js/crop-details.js` கோப்பில், `renderPriceCharts` செயல்பாட்டிற்குள், வரைபடங்களுக்கான HTML-ஐ உருவாக்கும்போது, தேதி உள்ளீடுகளையும் சேர்த்து உருவாக்குவோம்.

`public/js/crop-details.js`-ல் உள்ள `renderPriceCharts` செயல்பாட்டை இப்படி மாற்றி அமைக்கவும்:

```javascript
// public/js/crop-details.js

// வரைபட நிகழ்வுகளை (instances) வைத்திருக்க, செயல்பாட்டிற்கு வெளியே மாறிகளை வரையறுக்கவும்
let priceComparisonChart, monthlyTrendChart;

const renderPriceCharts = async (startDate = '', endDate = '') => {
    try {
        // படி 1.1: URL-ல் தேதி அளவுருக்களைச் சேர்ப்பது
        const params = new URLSearchParams();
        if (startDate) params.append('startDate', startDate);
        if (endDate) params.append('endDate', endDate);
        
        const response = await fetch(`/api/crops/${cropId}/price-analytics?${params.toString()}`);
        
        // ... (மீதமுள்ள try block) ...

        // கொள்கலன் HTML-ஐ உருவாக்கு (தேதி உள்ளீடுகளுடன்)
        const chartContainerHTML = `
            <div class="mt-8">
                <h2 class="text-2xl font-semibold text-gray-700 mb-4">விலை பகுப்பாய்வு</h2>
                
                <!-- படி 1.2: தேதி வரம்பு வடிகட்டி படிவம் -->
                <div class="bg-white p-4 rounded-lg shadow-md mb-6 flex flex-col sm:flex-row items-center gap-4">
                    <div class="flex-1 w-full">
                        <label for="startDate" class="block text-sm font-medium text-gray-700">தொடக்கத் தேதி</label>
                        <input type="date" id="startDate" name="startDate" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm">
                    </div>
                    <div class="flex-1 w-full">
                        <label for="endDate" class="block text-sm font-medium text-gray-700">இறுதித் தேதி</label>
                        <input type="date" id="endDate" name="endDate" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm">
                    </div>
                    <button id="filter-btn" class="bg-green-700 text-white font-bold py-2 px-6 rounded-lg self-end">வடிகட்டு</button>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 bg-white p-6 rounded-lg shadow-md">
                    <!-- ... (canvas கூறுகள் இங்கே) ... -->
                </div>
            </div>`;

        // பழைய கொள்கலனை அகற்றிவிட்டு, புதியதைச் செருகு
        const existingContainer = document.getElementById('charts-main-container');
        if (existingContainer) existingContainer.remove();
        detailsContainer.insertAdjacentHTML('beforeend', `<div id="charts-main-container">${chartContainerHTML}</div>`);
        
        // ... (மீதமுள்ள Chart.js குறியீடு) ...
    }
    // ...
};

// ...
```
**முக்கிய மாற்றம்:** வரைபடங்களை உருவாக்கும்போதே, அதனுடன் தேதி உள்ளீடுகளையும் ஒரு `<div>`-க்குள் வைத்துவிடுகிறோம்.

---

### படி 2: Backend API-ஐ தேதி வரம்பை ஏற்குமாறு மாற்றுதல்

`controllers/cropController.js`-ல், `getCropPriceAnalytics` செயல்பாட்டை, `startDate` மற்றும் `endDate`-ஐப் பயன்படுத்தி தரவை வடிகட்டுமாறு மாற்றுவோம்.

```javascript
// controllers/cropController.js
const getCropPriceAnalytics = async (req, res) => {
    try {
        const db = getDB();
        // படி 2.1: URL-ல் இருந்து தேதிகளைப் பெறுதல்
        const { startDate, endDate } = req.query;

        const crop = await db.collection('crops').findOne({ _id: new ObjectId(req.params.id) });

        if (!crop || !crop.marketPriceHistory || crop.marketPriceHistory.length === 0) {
            return res.status(404).json({ message: 'விலைத் தரவு இல்லை.' });
        }
        
        // படி 2.2: தேதி வரம்பின் அடிப்படையில் விலைத் தரவை வடிகட்டுதல்
        let pricesToAnalyze = crop.marketPriceHistory;
        if (startDate && endDate) {
            pricesToAnalyze = crop.marketPriceHistory.filter(p => {
                const priceDate = new Date(p.date);
                return priceDate >= new Date(startDate) && priceDate <= new Date(endDate);
            });
        }
        
        if (pricesToAnalyze.length === 0) {
            return res.status(404).json({ message: 'தேர்ந்தெடுத்த தேதி வரம்பில் தரவு இல்லை.' });
        }

        // படி 2.3: வடிகட்டப்பட்ட தரவின் மீது பகுப்பாய்வு செய்தல்
        const sortedPrices = [...pricesToAnalyze].sort((a, b) => new Date(b.date) - new Date(a.date));
        
        // ... (மீதமுள்ள பகுப்பாய்வு தர்க்கம் (logic) sortedPrices-க்குப் பதிலாக pricesToAnalyze-ஐப் பயன்படுத்தும்) ...
        const latestDate = sortedPrices[0].date;
        const latestPriceComparison = sortedPrices.filter(p => p.date === latestDate);

        // ... (மாதாந்திரப் போக்கு தர்க்கம்) ...
        // ... இதுவும் `sortedPrices`-ஐப் பயன்படுத்தும் ...

        res.status(200).json({ latestPriceComparison, monthlyTrend });

    } catch (err) {
        // ... (பிழை கையாளுதல்) ...
    }
};
```
*   API இப்போது தேதி அளவுருக்களை ஏற்று, JavaScript-ன் `.filter()` முறையைப் பயன்படுத்தி, பகுப்பாய்வு செய்வதற்கு முன் தரவைச் சுருக்குகிறது.

---

### படி 3: Frontend JavaScript-ல் வடிகட்டி செயல்பாட்டை இணைத்தல்

`public/js/crop-details.js`-ல், "வடிகட்டு" பொத்தானைக் கிளிக் செய்யும்போது, வரைபடங்களை மீண்டும் உருவாக்கும் தர்க்கத்தைச் சேர்ப்போம்.

```javascript
// public/js/crop-details.js

// ... `renderPriceCharts` செயல்பாட்டிற்கு வெளியே ...

const setupEventListeners = () => {
    // detailsContainer-ல் event listener-ஐ ஒருமுறை மட்டும் இணைக்கவும்
    detailsContainer.addEventListener('click', (event) => {
        if (event.target && event.target.id === 'filter-btn') {
            const startDate = document.getElementById('startDate').value;
            const endDate = document.getElementById('endDate').value;

            if (startDate && endDate) {
                renderPriceCharts(startDate, endDate);
            } else {
                alert('தொடக்க மற்றும் இறுதித் தேதிகளைத் தேர்ந்தெடுக்கவும்.');
            }
        }
    });
};


const loadPage = async () => {
    await loadCropDetails();
    await renderPriceCharts(); // ஆரம்பத்தில் தேதி இல்லாமல் ஏற்றவும்
    setupEventListeners();   // நிகழ்வு கேட்பவரை அமைக்கவும்
};

// ... `renderPriceCharts` செயல்பாட்டிற்குள் ஒரு சிறிய மாற்றம் ...
// வரைபடங்களை உருவாக்கும் முன், பழைய வரைபடங்களை அழிக்கவும்
if (priceComparisonChart) priceComparisonChart.destroy();
if (monthlyTrendChart) monthlyTrendChart.destroy();

// ... new Chart(...) என்று தொடங்கும் வரிகளுக்குப் பதிலாக ...
priceComparisonChart = new Chart(ctx1, { /* ... */ });
monthlyTrendChart = new Chart(ctx2, { /* ... */ });
```
**முழுமையான தர்க்கம்:**
1.  பக்கம் ஏற்றப்பட்டவுடன், `loadPage()` அழைக்கப்படும். அது `loadCropDetails()`-ஐயும், பிறகு `renderPriceCharts()`-ஐயும் (தேதிகள் இல்லாமல்) அழைத்து, அனைத்துத் தரவுகளையும் காட்டும். `setupEventListeners()` அழைக்கப்பட்டு, "வடிகட்டு" பொத்தானுக்கு listener அமைக்கப்படும்.
2.  பயனர் தேதிகளைத் தேர்ந்தெடுத்து, "வடிகட்டு" பொத்தானை அழுத்தும்போது, listener ஆனது தேதி உள்ளீடுகளிலிருந்து மதிப்புகளைப் பெற்று, `renderPriceCharts(startDate, endDate)`-ஐ மீண்டும் அழைக்கும்.
3.  `renderPriceCharts` செயல்பாடு இப்போது தேதி அளவுருக்களுடன் API-க்கு ஒரு புதிய கோரிக்கையை அனுப்பும்.
4.  API, வடிகட்டப்பட்ட தரவைத் திருப்பி அனுப்பும்.
5.  `renderPriceCharts` செயல்பாடு, பழைய வரைபடங்களை `chart.destroy()` மூலம் அழித்துவிட்டு, புதிய கொள்கலன் மற்றும் புதிய தரவைக் கொண்டு வரைபடங்களை மீண்டும் உருவாக்கும்.

### சோதித்துப் பார்ப்போம்

1.  உங்கள் சேவையகத்தை மறுதொடக்கம் செய்யவும்.
2.  விலைத் தரவுகளுடன் கூடிய பயிரின் விவரங்கள் பக்கத்திற்குச் செல்லவும்.
3.  ஆரம்பத்தில், அனைத்துத் தரவுகளையும் கொண்ட வரைபடங்கள் காட்டப்படும். அதனுடன் தேதி உள்ளீடுகளும், "வடிகட்டு" பொத்தானும் இருக்கும்.
4.  உங்கள் தரவுத்தளத்தில் உள்ள தேதிகளுக்கு ஏற்ப ஒரு குறிப்பிட்ட வரம்பைத் (range) தேர்ந்தெடுக்கவும் (எ.கா., ஜூன் 1 முதல் ஜூன் 30 வரை).
5.  "வடிகட்டு" பொத்தானை அழுத்தவும்.
6.  பக்கம் மீண்டும் ஏற்றப்படாமல், வரைபடங்கள் தானாகவே புதுப்பிக்கப்பட்டு, நீங்கள் தேர்ந்தெடுத்த தேதி வரம்பிற்கான தரவை மட்டும் காண்பிக்கும்.

நாம் இப்போது ஒரு ஊடாடும் (interactive), பயனரால் கட்டுப்படுத்தக்கூடிய வரைபட அம்சத்தை வெற்றிகரமாக உருவாக்கியுள்ளோம்