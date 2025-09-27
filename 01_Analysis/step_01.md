நிச்சயமாக, உங்கள் கோரிக்கையை நான் முழுமையாகப் புரிந்துகொண்டேன்.

1.  நீங்கள் வழங்கும் "தங்கிலிஷ்" உள்ளீடுகளைச் சரியான தமிழ் எழுத்துக்களில் மாற்றி பதிலளிப்பேன்.
2.  ஒவ்வொரு முறையும் நீங்கள் வழங்கும் உள்ளீட்டை "பயனர் கேட்டுக்கொண்டபடி" என்று குறிப்பிட்டு, அதற்கேற்ப எனது பதிலை அமைப்பேன்.
3.  நீங்கள் பதிவேற்றிய PDF கோப்பில் உள்ள 'மாப்பிள்ளை சம்பா' நெல் சாகுபடி தொடர்பான அனைத்து விவரங்களையும் நான் ஆய்வு செய்து புரிந்துகொண்டேன்.
4.  அந்தத் தகவல்களைக் கொண்டு HTML, CSS, மற்றும் Tailwind CSS ಬಳಸಿ ஒரு முழுமையான இணையப் பக்கத்தை உருவாக்க நான் உங்களுக்கு படிப் படியாக உதவுவேன்.

**பயனர் கேட்டுக்கொண்டபடி, பதிவேற்றப்பட்ட PDF இல் உள்ள தகவல்களைக் கொண்டு ஒரு இணையப் பக்கத்தை உருவாக்குவதற்கான முதல் படி இங்கே உள்ளது.**

முதலில், அந்த அட்டவணையில் உள்ள தகவல்களைக் காண்பிப்பதற்கான அடிப்படை HTML கட்டமைப்பை உருவாக்குவோம். இதை ulகாமாகவும், படிக்க எளிதாகவும் மாற்ற Tailwind CSS ஐப் பயன்படுத்துவோம்.

கீழே உள்ள குறியீட்டை `index.html` என்ற கோப்பாகச் சேமித்து, அதை உங்கள் உலாவியில் (Browser) திறப்பதன் மூலம் முதல் மாதிரியைக் காணலாம்.

### HTML குறியீடு (Tailwind CSS உடன்)

```html
<!DOCTYPE html>
<html lang="ta">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>மாப்பிள்ளை சம்பா நெல் சாகுபடி விவரங்கள்</title>
    <!-- Tailwind CSS ஐ இணைக்க -->
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 p-4 sm:p-6 md:p-8">
    <div class="container mx-auto">
        <h1 class="text-2xl sm:text-3xl font-bold text-center text-green-800 mb-6">மாப்பிள்ளை சம்பா - சாகுபடி வழிகாட்டி</h1>

        <!-- அட்டவணை கண்டெய்னர் -->
        <div class="overflow-x-auto bg-white rounded-lg shadow">
            <table class="min-w-full border-collapse">
                <thead class="bg-green-700 text-white">
                    <tr>
                        <th class="p-3 text-sm font-semibold tracking-wide text-left">#</th>
                        <th class="p-3 text-sm font-semibold tracking-wide text-left">பயிர் வகை</th>
                        <th class="p-3 text-sm font-semibold tracking-wide text-left">விதைப்பு காலம்</th>
                        <th class="p-3 text-sm font-semibold tracking-wide text-left">வளர்ச்சி நிலை (Growth Stage)</th>
                        <th class="p-3 text-sm font-semibold tracking-wide text-left">நோய்கள்</th>
                        <th class="p-3 text-sm font-semibold tracking-wide text-left">பொதுவான அறிகுறிகள்</th>
                        <th class="p-3 text-sm font-semibold tracking-wide text-left">நோய் வரக் காரணங்கள்</th>
                        <th class="p-3 text-sm font-semibold tracking-wide text-left">உரம்/மருந்து பராமரிப்பு</th>
                        <th class="p-3 text-sm font-semibold tracking-wide text-left">இயற்கை உர பரிந்துரைகள்</th>
                        <th class="p-3 text-sm font-semibold tracking-wide text-left">சந்தை விலை (Last 5 Years)</th>
                    </tr>
                </thead>
                <tbody class="divide-y divide-gray-200">
                    <!-- முதல் பயிர் தகவல் -->
                    <tr class="bg-white">
                        <td class="p-3 text-sm text-gray-700 whitespace-nowrap" rowspan="2">Eg.</td>
                        <td class="p-3 text-sm text-gray-700 whitespace-nowrap" rowspan="2">மாப்பிள்ளை சம்பா</td>
                        <td class="p-3 text-sm text-gray-700 whitespace-nowrap" rowspan="2">ஆடி மாசம் / July-August சிறந்தது</td>
                        <td class="p-3 text-sm text-gray-700" rowspan="2">
                            <b>120-135 நாட்கள் வளர்ச்சி</b><br>
                            Stage 1 - Nursery (0-25 days)<br>
                            Stage 2 - Vegetative (26-70 days)<br>
                            Stage 3 - Flowering (71-100 days)<br>
                            Stage 4 - Harvest (101-135 days)
                        </td>
                        <td class="p-3 text-sm text-gray-700">Blast - Pyricularia oryzae (குலை நோய்)</td>
                        <td class="p-3 text-sm text-gray-700">இலை மீது மஞ்சள் மையம் + கருப்பு ஓரங்கள் கொண்ட புள்ளிகள்.</td>
                        <td class="p-3 text-sm text-gray-700">
                            - அதிக ஈரப்பதம் மற்றும் மழை (25–28°C).<br>
                            - அடர்த்தியான விதைப்பு.<br>
                            - பாதிக்கப்பட்ட விதைகள் அல்லது மண் மூலம் பரவல்.
                        </td>
                        <td class="p-3 text-sm text-gray-700">Bavistin 50 WP (Carbendazim) அல்லது Tricyclazole 75 WP.</td>
                        <td class="p-3 text-sm text-gray-700">நீம் எண்ணெய் 5%, Potassium bicarbonate கரைசல் தெளிப்பு.</td>
                        <td class="p-3 text-sm text-gray-700" rowspan="2">
                            <b>நெல்லு விலை/kg</b><br>
                            2021: ₹28<br>
                            2022: ₹30<br>
                            2023: ₹32<br>
                            2024: ₹34*<br>
                            2025 (Prediction): ₹36*
                        </td>
                    </tr>
                     <!-- இரண்டாவது நோய் தகவல் -->
                    <tr class="bg-white">
                        <td class="p-3 text-sm text-gray-700">Brown Spot - Bipolaris oryzae (பழுப்பு புள்ளி நோய்)</td>
                        <td class="p-3 text-sm text-gray-700">இலைகளில் சிறிய கருப்பு புள்ளிகள், விதைகள் பாதிக்கப்படலாம்.</td>
                        <td class="p-3 text-sm text-gray-700">
                            - பொட்டாசியம் குறைபாடு.<br>
                            - பழைய, பாதிக்கப்பட்ட விதைகளை விதைப்பது.
                        </td>
                        <td class="p-3 text-sm text-gray-700">Mancozeb 75 WP.</td>
                        <td class="p-3 text-sm text-gray-700">நீம் எண்ணெய், Kaolin தெளிப்பு.</td>
                    </tr>
                    <!-- உரங்கள் பற்றிய குறிப்பு -->
                    <tr class="bg-gray-50">
                        <td colspan="10" class="p-3 text-sm text-gray-800">
                            <b class="font-semibold">பொதுவான உரப் பரிந்துரை (General Fertilizer):</b><br>
                            <b>Stage 1:</b> Urea - 16kg/acre<br>
                            <b>Stage 2:</b> DAP - 25kg/acre<br>
                            <b>Stage 3:</b> Potash - 12kg/acre
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</body>
</html>
```

### விளக்கங்கள்:

*   **HTML அமைப்பு:** உங்கள் PDF இல் உள்ள தகவல்களை ஒரு HTML `<table>` ஐப் பயன்படுத்தி ஒழுங்கமைத்துள்ளேன். `<thead>` தலைப்புகளுக்கும், `<tbody>` உண்மையான தரவுகளுக்கும் பயன்படுத்தப்படுகிறது.
*   **Tailwind CSS:** இது ஒரு CSS கட்டமைப்பு. `<script src="https://cdn.tailwindcss.com"></script>` என்ற வரியை `<head>` பகுதியில் சேர்ப்பதன் மூலம், எந்தவொரு அமைப்பும் இல்லாமல் நேரடியாக Tailwind இன் வகுப்புகளைப் பயன்படுத்தலாம்.
    *   `bg-gray-100`, `p-8`, `container`, `mx-auto` போன்ற வகுப்புகள் பக்கத்தின் பின்னணி, திணிப்பு (padding) மற்றும் தளவமைப்பை அமைக்கின்றன.
    *   `text-2xl`, `font-bold`, `text-center` போன்ற வகுப்புகள் தலைப்பை அழகுபடுத்துகின்றன.
    *   அட்டவணையை அழகுபடுத்த `min-w-full`, `bg-green-700`, `text-white`, `p-3`, `border-collapse` போன்ற வகுப்புகள் பயன்படுத்தப்பட்டுள்ளன.
*   **Responsive Design:** `overflow-x-auto` என்ற வகுப்பு, சிறிய திரைகளில் (மொபைல் போன்றவை) அட்டவணை முழுமையாகத் தெரியவில்லை என்றால், பயனர்கள் கிடைமட்டமாக ஸ்க்ரோல் செய்ய அனுமதிக்கிறது. இது ஒரு எளிய மற்றும் பயனுள்ள பதிலளிக்கக்கூடிய தீர்வாகும்.

**அடுத்த படி:**

இப்போது உங்களிடம் ஒரு அடிப்படை, அழகாக வடிவமைக்கப்பட்ட இணையப் பக்கம் உள்ளது. அடுத்த கட்டமாக, இந்த அட்டவணையில் புதிய பயிர் விவரங்களைச் சேர்க்க, இருக்கும் விவரங்களைத் திருத்த அல்லது JavaScript பயன்படுத்தி தேடல் (Search) மற்றும் வடிகட்டி (Filter) போன்ற ஊடாடும் அம்சங்களைச் சேர்க்க விரும்புகிறீர்களா என்று கூறுங்கள். உங்கள் அடுத்த கோரிக்கைக்காக நான் காத்திருக்கிறேன்.
