நிச்சயமாக, அடுத்த மேம்பாட்டிற்குச் செல்வோம்!

**வரைபட வகை மாற்றி (Chart Type Toggle)** அம்சத்தைச் சேர்ப்போம். இது பயனர்கள் மாதாந்திர விலைப்போக்கை ஒரு பட்டை வரைபடமாகவோ (Bar Chart) அல்லது கோட்டு வரைபடமாகவோ (Line Chart) பார்க்க அனுமதிக்கும். கோட்டு வரைபடங்கள் காலப்போக்கில் ஏற்படும் தொடர்ச்சியான மாற்றங்களை மிக அழகாகக் காட்டும்.

இந்த அம்சம் முற்றிலும் Frontend-ல் செய்யக்கூடியது. நமது Backend API-ல் எந்த மாற்றமும் செய்யத் தேவையில்லை, ஏனெனில் அது ஏற்கெனவே தேவையான தரவை வழங்குகிறது.

### படி 1: UI-ல் வரைபட வகை மாற்றி பொத்தான்களைச் சேர்த்தல்

`public/js/crop-details.js`-ல், மாதாந்திரப் போக்கு வரைபடத்திற்கான HTML-ஐ உருவாக்கும்போது, அதனுடன் இரண்டு புதிய பொத்தான்களையும் சேர்ப்போம்.

`renderPriceCharts` செயல்பாட்டில், `chartContainerHTML`-ஐ உருவாக்கும் பகுதியை இப்படி மாற்றியமைக்கவும்:

```javascript
// public/js/crop-details.js

// ... `renderPriceCharts` செயல்பாட்டிற்குள் ...

// கொள்கலன் HTML-ஐ உருவாக்கு
const chartContainerHTML = `
    <div class="mt-8">
        <h2 class="text-2xl font-semibold text-gray-700 mb-4">விலை பகுப்பாய்வு</h2>
        <!-- ... தேதி வரம்பு வடிகட்டி படிவம் ... -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 bg-white p-6 rounded-lg shadow-md">
            <!-- ... சமீபத்திய ஒப்பீட்டு வரைபடம் ... -->
            
            <!-- மாதாந்திரப் போக்கு வரைபடம் (மாற்றிகளுடன்) -->
            <div>
                <div class="flex justify-between items-center mb-2">
                    <h3 class="text-lg font-semibold text-center">மாதாந்திர சராசரி விலைப்போக்கு</h3>
                    
                    <!-- படி 1.1: வரைபட வகை மாற்றி பொத்தான்கள் -->
                    <div id="chart-type-toggle" class="flex border border-gray-300 rounded-md p-1">
                        <button data-chart-type="bar" class="chart-toggle-btn active bg-green-700 text-white px-2 py-1 text-xs rounded-sm">Bar</button>
                        <button data-chart-type="line" class="chart-toggle-btn px-2 py-1 text-xs rounded-sm">Line</button>
                    </div>
                </div>
                <canvas id="monthlyTrendChart"></canvas>
            </div>
        </div>
    </div>`;

// ...
```
**முக்கிய மாற்றங்கள்:**
*   `chart-type-toggle` என்ற `id` உடன் ஒரு `div` உருவாக்கப்பட்டுள்ளது.
*   அதற்குள், `data-chart-type` என்ற பண்புடன் (attribute) இரண்டு பொத்தான்கள் உள்ளன. `bar` பொத்தான் ஆரம்பத்தில் `active` வகுப்பைக் கொண்டுள்ளது.

---

### படி 2: JavaScript-ல் வரைபடத்தை மீண்டும் வரையும் தர்க்கத்தைச் சேர்த்தல்

`public/js/crop-details.js` கோப்பில், வரைபடத்தை மீண்டும் உருவாக்கும் தர்க்கத்தை இப்போது சேர்ப்போம்.

```javascript
// public/js/crop-details.js

// ... கோப்பின் மேற்பகுதியில் ...
let priceComparisonChart, monthlyTrendChart;
let currentChartType = 'bar'; // படி 2.1: தற்போதைய வரைபட வகையைச் சேமிக்க ஒரு மாறி
let monthlyTrendData = null; // API தரவைச் சேமிக்க

// ... `renderPriceCharts` செயல்பாட்டிற்குள் ...

const renderPriceCharts = async (startDate = '', endDate = '') => {
    // ... API அழைப்பு ...
    const data = await response.json();
    monthlyTrendData = data.monthlyTrend; // படி 2.2: தரவை சேமித்துக்கொள்

    // ... HTML-ஐ உருவாக்குதல் ...

    // வரைபடங்களை வரையும் செயல்பாடுகளை அழைக்கவும்
    drawComparisonChart(data.latestPriceComparison);
    drawMonthlyTrendChart(); // இப்போது இது அளவுருக்கள் இல்லாமல் அழைக்கப்படும்
};

// படி 2.3: மாதாந்திரப் போக்கு வரைபடத்தை வரையும் ஒரு தனிச் செயல்பாடு
const drawMonthlyTrendChart = () => {
    if (!monthlyTrendData) return;

    // பழைய வரைபடம் இருந்தால், அதை அழிக்கவும்
    if (monthlyTrendChart) {
        monthlyTrendChart.destroy();
    }

    const ctx = document.getElementById('monthlyTrendChart').getContext('2d');
    
    // கோட்டு வரைபடத்திற்கான பிரத்யேக உள்ளமைவுகள்
    const lineChartOptions = {
        backgroundColor: 'rgba(22, 101, 52, 0.2)',
        borderColor: '#166534',
        tension: 0.1, // கோட்டின் வளைவு
        fill: true,
    };

    monthlyTrendChart = new Chart(ctx, {
        type: currentChartType, // 'bar' அல்லது 'line'
        data: {
            labels: monthlyTrendData.map(m => m.month),
            datasets: [{
                label: 'சராசரி விலை (₹/kg)',
                data: monthlyTrendData.map(m => m.averagePrice),
                // வரைபட வகையைப் பொறுத்து உள்ளமைவுகளை மாற்று
                ...(currentChartType === 'line' ? lineChartOptions : { backgroundColor: '#166534' })
            }]
        },
        options: { scales: { y: { beginAtZero: false } } }
    });
};

// ஒப்பீட்டு வரைபடத்தையும் ஒரு தனிச் செயல்பாடாக மாற்றுவது நல்லது
const drawComparisonChart = (data) => {
    // ... (ஒப்பீட்டு வரைபடத்தை உருவாக்கும் தர்க்கம் இங்கே) ...
};

// படி 2.4: மாற்றி பொத்தான்களுக்கான Event Listener
const setupEventListeners = () => {
    // ... தேதி வடிகட்டி listener ...

    detailsContainer.addEventListener('click', (event) => {
        const toggleBtn = event.target.closest('.chart-toggle-btn');
        if (toggleBtn) {
            const newType = toggleBtn.dataset.chartType;
            if (newType !== currentChartType) {
                currentChartType = newType;
                drawMonthlyTrendChart(); // புதிய வகையுடன் வரைபடத்தை மீண்டும் வரை

                // UI-ல் active வகுப்பை மாற்று
                document.querySelectorAll('.chart-toggle-btn').forEach(btn => btn.classList.remove('active', 'bg-green-700', 'text-white'));
                toggleBtn.classList.add('active', 'bg-green-700', 'text-white');
            }
        }
    });
};

// ... `loadPage` செயல்பாடு ...
```

### என்ன செய்துள்ளோம்?

1.  **`currentChartType`:** இது தற்போது எந்த வகை வரைபடம் காட்டப்படுகிறது ('bar' அல்லது 'line') என்பதை நினைவில் வைத்திருக்கும் ஒரு மாறி.
2.  **`monthlyTrendData`:** API-இடம் இருந்து ஒருமுறை பெற்ற தரவை இந்த மாறியில் சேமித்துக்கொள்கிறோம். இதனால், பயனர் வரைபட வகையை மாற்றும்போது, மீண்டும் API-ஐ அழைக்கத் தேவையில்லை. இது செயல்திறனை மேம்படுத்தும்.
3.  **`drawMonthlyTrendChart()`:** இது ஒரு பிரத்யேக செயல்பாடு. இது `currentChartType`-ஐப் பார்த்து, அதற்கேற்ப ஒரு பட்டை அல்லது கோட்டு வரைபடத்தை உருவாக்கும். ஒரு புதிய வரைபடத்தை உருவாக்கும் முன், `monthlyTrendChart.destroy()` மூலம் பழைய வரைபடத்தை அழிப்பது மிகவும் முக்கியம்.
4.  **Event Listener:** `chart-type-toggle` கொள்கலனுக்கு ஒரு event listener-ஐச் சேர்த்துள்ளோம். பயனர் ஒரு பொத்தானை அழுத்தும்போது:
    *   அது ஏற்கெனவே `active`-ஆக இல்லையென்றால், `currentChartType`-ஐப் புதுப்பிக்கும்.
    *   `drawMonthlyTrendChart()`-ஐ அழைத்து, புதிய வகையுடன் வரைபடத்தை மீண்டும் உருவாக்கும்.
    *   பொத்தான்களின் CSS வகுப்புகளை மாற்றி, எது `active` என்பதைப் பயனருக்குக் காட்டும்.

### சோதித்துப் பார்ப்போம்

1.  உங்கள் சேவையகத்தை மறுதொடக்கம் செய்யவும்.
2.  விலைத் தரவுகளுடன் கூடிய பயிரின் விவரங்கள் பக்கத்திற்குச் செல்லவும்.
3.  மாதாந்திர விலைப்போக்கு வரைபடத்தின் மேலே, "Bar" மற்றும் "Line" என்ற இரண்டு பொத்தான்களைக் காண்பீர்கள். ஆரம்பத்தில் "Bar" தேர்ந்தெடுக்கப்பட்டிருக்கும்.
4.  "Line" பொத்தானை அழுத்தவும்.
5.  பக்கம் மீண்டும் ஏற்றப்படாமல், பட்டை வரைபடம் உடனடியாக ஒரு அழகான கோட்டு வரைபடமாக மாறும்.
6.  மீண்டும் "Bar" பொத்தானை அழுத்தினால், அது பட்டை வரைபடமாக மாறும்.
7.  தேதி வரம்பு வடிகட்டியைப் பயன்படுத்தினால், வரைபடம் புதிய தரவுகளுடன் புதுப்பிக்கப்படும், மேலும் நீங்கள் அப்போதும் Bar/Line வகையை மாற்ற முடியும்.

நாம் இப்போது பயனர்களுக்குத் தரவை ஆராய்வதற்கான மற்றொரு சிறந்த கருவியை வெற்றிகரமாக வழங்கியுள்ளோம். இது உங்கள் செயலியை இன்னும் ஊடாடும் மற்றும் பயனுள்ளதாக்குகிறது.