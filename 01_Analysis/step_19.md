**அடுத்து, இந்தப் பதிவேற்றிய படங்களை பயனர் பக்கத்தில் (`crop-details` page) ஒரு அழகான `Swiper.js` Carousel-ஐப் பயன்படுத்தி எப்படிக் காட்டுவது என்று பார்க்கலாமா?**

நிச்சயமாக, தொடர்வோம்.

நாம் பதிவேற்றிய படங்களைப் பயனர்களுக்கு ஒரு சிறந்த அனுபவத்துடன் காட்டுவதுதான் இந்தச் செயல்பாட்டின் இறுதி நோக்கம். இதற்கு `Swiper.js` என்ற இலகுவான, சக்திவாய்ந்த Carousel ലൈബ്രറി-ஐப் பயன்படுத்துவோம்.

### படி 5: பயனர் பக்கத்தில் Swiper.js Carousel-ஐச் செயல்படுத்துதல்

#### 1. Swiper.js லைப்ரரியை செயலியில் சேர்த்தல்

`views/layouts/main.hbs` கோப்பைத் திறந்து, Swiper.js-க்கான CSS மற்றும் JS கோப்புகளை CDN (Content Delivery Network) மூலம் இணைப்போம். இதை `<head>` மற்றும் `<body>` குறிச்சொற்களுக்குள் சேர்க்க வேண்டும்.

**`views/layouts/main.hbs`**
```html
<!DOCTYPE html>
<html lang="ta">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{title}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- படி 5.1: Swiper CSS-ஐ இணைக்கவும் -->
    <link rel="stylesheet" href="https://unpkg.com/swiper/swiper-bundle.min.css" />
    
    <!-- சில பிரத்யேக CSS (விரும்பினால்) -->
    <style>
        .swiper-button-next, .swiper-button-prev {
            color: #166534; /* Green color for arrows */
        }
    </style>
</head>
<body class="bg-gray-100">
    <div class="container mx-auto p-4">
        {{{body}}}
    </div>

    <!-- படி 5.2: Swiper JS-ஐ இணைக்கவும் -->
    <script src="https://unpkg.com/swiper/swiper-bundle.min.js"></script>
</body>
</html>
```

#### 2. விவரங்கள் பக்கத்தில் Carousel-க்கான HTML கட்டமைப்பை உருவாக்குதல் (`views/crop-details.hbs`)

`views/crop-details.hbs`-ல், நாம் தரவை ஏற்றிக் காண்பிக்கும் `#details-container`-க்குள்ளேயே Carousel-க்கான HTML σκελετός-ஐயும் JavaScript மூலம் உருவாக்குவோம். இது தரவு இல்லாதபோது Carousel காட்டப்படாமல் இருப்பதை உறுதி செய்யும்.

எனவே, இந்த `hbs` கோப்பில் நாம் நேரடியாக எந்த மாற்றமும் செய்யத் தேவையில்லை. முழு வேலையையும் `crop-details.js` பார்த்துக் கொள்ளும்.

#### 3. JavaScript மூலம் Carousel-ஐ உருவாக்குதல் (`public/js/crop-details.js`)

இதுதான் மிக முக்கியமான பகுதி. `loadCropDetails` செயல்பாட்டில், API-இடம் இருந்து படப் பாதைகளைப் பெற்று, Swiper-க்குத் தேவையான HTML கட்டமைப்பை உருவாக்கி, பின்னர் Swiper-ஐத் தொடங்குவோம்.

`public/js/crop-details.js` கோப்பில் உள்ள `loadCropDetails` செயல்பாட்டைப் பின்வருமாறு மாற்றவும்:

```javascript
// public/js/crop-details.js
// ...
const loadCropDetails = async () => {
    try {
        const response = await fetch(`/api/crops/${cropId}`);
        if (!response.ok) throw new Error('பயிர் விவரங்களைப் பெற முடியவில்லை.');
        const crop = await response.json();

        // ... ஏற்றுதல் செய்தியை நீக்குதல், தலைப்பை மாற்றுதல் ...

        // படி 5.3: Carousel-க்கான HTML-ஐ உருவாக்குதல்
        let carouselHTML = '';
        const images = [];
        if (crop.coverImage) images.push(crop.coverImage);
        if (crop.galleryImages) images.push(...crop.galleryImages);

        if (images.length > 0) {
            carouselHTML = `
                <div class="swiper mySwiper mb-8 rounded-lg shadow-lg">
                    <div class="swiper-wrapper">
                        ${images.map(imagePath => `
                            <div class="swiper-slide bg-gray-200">
                                <img src="${imagePath}" alt="${crop.name}" class="w-full h-64 md:h-96 object-cover">
                            </div>
                        `).join('')}
                    </div>
                    <!-- Navigation Buttons -->
                    <div class="swiper-button-next"></div>
                    <div class="swiper-button-prev"></div>
                    <!-- Pagination Dots -->
                    <div class="swiper-pagination"></div>
                </div>
            `;
        }

        // முழு HTML-ஐயும் உருவாக்குதல் (Carousel மற்றும் மற்ற விவரங்கள்)
        let contentHTML = `
            ${carouselHTML} <!-- Carousel-ஐ முதலில் சேர்க்கவும் -->
            
            <!-- 1. தலைப்புப் பகுதி -->
            <div class="border-b pb-4 mb-6">
                <!-- ... (ஏற்கனவே உள்ள தலைப்பு மற்றும் விவரங்கள்) ... -->
            </div>
            
            <!-- ... (மீதமுள்ள வளர்ச்சி நிலைகள், நோய்கள், உரங்கள் பகுதிகள்) ... -->
        `;
        
        // நோய் மேலாண்மைப் பகுதியில் படங்களைக் காட்டுதல்
        const diseasesHTML = crop.diseases.map(disease => `
            <div class="border border-red-200 rounded-lg p-4 flex flex-col sm:flex-row gap-4">
                ${disease.image ? `
                    <div class="w-full sm:w-1/4">
                        <img src="${disease.image}" alt="${disease.diseaseName}" class="rounded-md object-cover w-full h-32">
                    </div>
                ` : ''}
                <div class="w-full ${disease.image ? 'sm:w-3/4' : ''}">
                    <h3 class="text-lg font-bold text-red-700">${disease.diseaseName}</h3>
                    <p class="mt-2 text-sm"><b class="font-semibold">அறிகுறிகள்:</b> ${disease.symptoms}</p>
                    <p class="mt-1 text-sm"><b class="font-semibold">காரணங்கள்:</b> ${disease.causes.join(', ')}</p>
                    <!-- ... (ரசாயன மற்றும் இயற்கை முறைகள்) ... -->
                </div>
            </div>
        `).join('');

        // contentHTML-ல் நோய்கள் பகுதியை இந்த புதிய diseasesHTML உடன் மாற்றவும்
        // (இதைச் செயல்படுத்த, உங்கள் contentHTML உருவாக்கும் முறையைச் சற்று மாற்றியமைக்க வேண்டும்)


        // உருவாக்கப்பட்ட HTML-ஐ கொள்கலனில் செருகு
        detailsContainer.innerHTML = contentHTML; // இதைச் சற்று மாற்றியமைப்போம்

        // இதை இன்னும் சிறப்பாகச் செய்ய, ஒவ்வொரு பகுதியையும் தனித்தனியாக உருவாக்கி இணைப்போம்.
        detailsContainer.innerHTML = `
            ${carouselHTML}
            <div class="border-b pb-4 mb-6">
                <h1 class="text-3xl sm:text-4xl font-bold text-green-800">${crop.name}</h1>
                <p class="text-md text-gray-600 mt-2">விதைப்பு காலம்: ${crop.sowingSeason} | வளர்ச்சி காலம்: ${crop.growthDuration}</p>
            </div>
            <!-- ... (மற்ற விவரங்கள்) ... -->
            <div class="mt-8">
                <h2 class="text-2xl font-semibold text-gray-700 mb-4">நோய் மேலாண்மை</h2>
                <div class="space-y-4">${diseasesHTML}</div>
            </div>
        `;
        
        // படி 5.4: Swiper-ஐத் தொடங்குதல் (Initialize)
        if (images.length > 0) {
            new Swiper(".mySwiper", {
                loop: true,
                pagination: {
                    el: ".swiper-pagination",
                    clickable: true,
                },
                navigation: {
                    nextEl: ".swiper-button-next",
                    prevEl: ".swiper-button-prev",
                },
                autoplay: {
                    delay: 3000,
                    disableOnInteraction: false,
                },
            });
        }
    } catch (error) {
        // ... (பிழை கையாளுதல்) ...
    }
};

loadCropDetails();
```
**விளக்கம்:**
1.  **Carousel HTML:** `coverImage` மற்றும் `galleryImages`-ஐ ஒன்றாக இணைத்து, `images` என்ற வரிசையை உருவாக்குகிறோம். அந்த வரிசையில் படங்கள் இருந்தால் மட்டும், Swiper-க்குத் தேவையான HTML (`.swiper`, `.swiper-wrapper`, `.swiper-slide`) உருவாக்கப்படுகிறது.
2.  **நோய்ப் படங்கள்:** நோய் மேலாண்மைப் பகுதியை உருவாக்கும்போது, `disease.image` புலம் உள்ளதா என்று சரிபார்க்கிறோம். இருந்தால், அந்தப் படத்தைக் காண்பிக்க ஒரு `<img>` குறிச்சொல்லைச் சேர்க்கிறோம். Flexbox (`flex`, `flex-row`) படத்தையும் உரையையும் பக்கவாட்டில் அழகாக அடுக்க உதவுகிறது.
3.  **Swiper Initialization:** HTML பக்கத்தில் செருகப்பட்ட பிறகு, `new Swiper(".mySwiper", { ... })` என்ற வரியை இயக்குகிறோம். இது அந்த HTML கட்டமைப்பைக் கண்டுபிடித்து, அதை ஒரு ஊடாடும் Carousel-ஆக மாற்றுகிறது.
    *   `loop`: கடைசிப் படத்திற்குப் பிறகு மீண்டும் முதல் படத்திற்குச் செல்லும்.
    *   `pagination`: கீழே சிறிய புள்ளிகளைக் காட்டும்.
    *   `navigation`: அடுத்த மற்றும் முந்தைய அம்புக்குறிகளைக் காட்டும்.
    *   `autoplay`: படங்கள் தானாகவே மாறும்.

---

### சோதித்துப் பார்ப்போம்

1.  உங்கள் சேவையகத்தை மறுதொடக்கம் செய்யவும் (`npm start`).
2.  நீங்கள் ஏற்கெனவே படங்கள் பதிவேற்றிய ஒரு பயிரின் (எ.கா., "மாப்பிள்ளை சம்பா") விவரங்கள் பக்கத்திற்குச் செல்லவும்.
3.  பக்கம் ஏற்றப்பட்டவுடன், பக்கத்தின் மேலே நீங்கள் பதிவேற்றிய முக்கியப் புகைப்படம் மற்றும் காட்சியகப் புகைப்படங்களுடன் ஒரு அழகான, தானாக இயங்கும் Carousel-ஐக் காண்பீர்கள். அம்புக்குறிகள் மற்றும் புள்ளிகளைப் பயன்படுத்தி நீங்கள் படங்களை மாற்ற முடியும்.
4.  கீழே நோய் மேலாண்மைப் பகுதியில், நீங்கள் நோய்களுக்குப் பதிவேற்றிய படங்கள், அந்தந்த நோய்களின் விவரங்களுக்கு அருகில் காட்டப்படும்.
5.  படங்கள் இல்லாத பயிரின் பக்கத்திற்குச் சென்றால், Carousel மற்றும் நோய்ப் படங்கள் காட்டப்படாது, இது பிழை இல்லாமல் சீராக வேலை செய்யும்.

வாழ்த்துக்கள்! உங்கள் செயலியை இப்போது ஒரு முழுமையான, காட்சி சார்ந்த வழிகாட்டியாக வெற்றிகரமாக மாற்றியுள்ளீர்கள். இது பயனர்களுக்கு மிகச் சிறந்த மற்றும் பயனுள்ள அனுபவத்தை வழங்கும்.