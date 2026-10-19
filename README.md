<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SkyPulse - Weather App</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Inter', sans-serif;
        }

        body {
            background: linear-gradient(135deg, #1e3c72, #2a5298);
            color: #fff;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .container {
            width: 100%;
            max-width: 420px;
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(15px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            border-radius: 24px;
            padding: 30px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
        }

        .search-box {
            display: flex;
            gap: 10px;
            margin-bottom: 25px;
        }

        .search-box input {
            flex: 1;
            padding: 14px 18px;
            border: none;
            outline: none;
            border-radius: 12px;
            background: rgba(255, 255, 255, 0.2);
            color: #fff;
            font-size: 16px;
        }

        .search-box input::placeholder {
            color: rgba(255, 255, 255, 0.7);
        }

        .search-box button {
            padding: 0 18px;
            border: none;
            outline: none;
            background: #ffffff;
            color: #1e3c72;
            border-radius: 12px;
            cursor: pointer;
            font-size: 16px;
            font-weight: 600;
            transition: background 0.3s;
        }

        .search-box button:hover {
            background: #e0e0e0;
        }

        .weather-display {
            text-align: center;
            display: none;
        }

        .weather-display.active {
            display: block;
        }

        .city-name {
            font-size: 26px;
            font-weight: 600;
            margin-bottom: 5px;
        }

        .date {
            font-size: 14px;
            color: rgba(255, 255, 255, 0.7);
            margin-bottom: 20px;
        }

        .weather-icon {
            font-size: 80px;
            margin: 15px 0;
            color: #ffeb3b;
        }

        .temp {
            font-size: 54px;
            font-weight: 700;
            margin-bottom: 5px;
        }

        .condition {
            font-size: 18px;
            font-weight: 500;
            text-transform: capitalize;
            margin-bottom: 25px;
        }

        .details-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            background: rgba(0, 0, 0, 0.15);
            padding: 15px;
            border-radius: 16px;
        }

        .detail-item {
            display: flex;
            align-items: center;
            gap: 12px;
            text-align: left;
        }

        .detail-item i {
            font-size: 22px;
            color: #64b5f6;
        }

        .detail-item .label {
            font-size: 12px;
            color: rgba(255, 255, 255, 0.7);
        }

        .detail-item .value {
            font-size: 15px;
            font-weight: 600;
        }

        .error-message {
            text-align: center;
            color: #ff6b6b;
            font-weight: 500;
            margin-top: 15px;
            display: none;
        }

        .loading {
            text-align: center;
            font-size: 16px;
            color: rgba(255, 255, 255, 0.8);
            margin-top: 20px;
            display: none;
        }
    </style>
</head>
<body>

    <div class="container">
        <div class="search-box">
            <input type="text" id="cityInput" placeholder="Enter city name..." autocomplete="off">
            <button id="searchBtn"><i class="fa-solid fa-magnifying-glass"></i></button>
        </div>

        <div class="loading" id="loading">Fetching weather data...</div>
        <div class="error-message" id="errorMsg">City not found. Please try again.</div>

        <div class="weather-display" id="weatherDisplay">
            <h2 class="city-name" id="cityName">London, UK</h2>
            <p class="date" id="dateText">Monday, October 19, 2026</p>
            <div class="weather-icon" id="weatherIcon"><i class="fa-solid fa-cloud-sun"></i></div>
            <div class="temp" id="tempValue">22°C</div>
            <div class="condition" id="conditionValue">Partly Cloudy</div>

            <div class="details-grid">
                <div class="detail-item">
                    <i class="fa-solid fa-droplet"></i>
                    <div>
                        <div class="label">Humidity</div>
                        <div class="value" id="humidityValue">64%</div>
                    </div>
                </div>
                <div class="detail-item">
                    <i class="fa-solid fa-wind"></i>
                    <div>
                        <div class="label">Wind Speed</div>
                        <div class="value" id="windValue">12 km/h</div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        const cityInput = document.getElementById('cityInput');
        const searchBtn = document.getElementById('searchBtn');
        const weatherDisplay = document.getElementById('weatherDisplay');
        const loading = document.getElementById('loading');
        const errorMsg = document.getElementById('errorMsg');

        const cityName = document.getElementById('cityName');
        const dateText = document.getElementById('dateText');
        const weatherIcon = document.getElementById('weatherIcon');
        const tempValue = document.getElementById('tempValue');
        const conditionValue = document.getElementById('conditionValue');
        const humidityValue = document.getElementById('humidityValue');
        const windValue = document.getElementById('windValue');

        // Set current date
        const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
        dateText.innerText = new Date().toLocaleDateString('en-US', options);

        searchBtn.addEventListener('click', () => fetchWeather());
        cityInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') fetchWeather();
        });

        async function fetchWeather() {
            const query = cityInput.value.trim();
            if (!query) return;

            loading.style.display = 'block';
            weatherDisplay.classList.remove('active');
            errorMsg.style.display = 'none';

            try {
                // Step 1: Get Latitude & Longitude using Open-Meteo Geocoding API
                const geoRes = await fetch(`https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(query)}&count=1`);
                const geoData = await geoRes.json();

                if (!geoData.results || geoData.results.length === 0) {
                    throw new Error("City not found");
                }

                const { latitude, longitude, name, country } = geoData.results[0];

                // Step 2: Get Weather Data using Open-Meteo Forecast API
                const weatherRes = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current=temperature_2m,relative_humidity_2m,weather_code,wind_speed_10m`);
                const weatherData = await weatherRes.json();

                const current = weatherData.current;
                
                cityName.innerText = `${name}, ${country || ''}`;
                tempValue.innerText = `${Math.round(current.temperature_2m)}°C`;
                humidityValue.innerText = `${current.relative_humidity_2m}%`;
                windValue.innerText = `${current.wind_speed_10m} km/h`;

                // Map WMO weather code to text and icon
                const weatherInfo = getWeatherDescription(current.weather_code);
                conditionValue.innerText = weatherInfo.text;
                weatherIcon.innerHTML = `<i class="${weatherInfo.icon}"></i>`;

                loading.style.display = 'none';
                weatherDisplay.classList.add('active');

            } catch (error) {
                loading.style.display = 'none';
                errorMsg.style.display = 'block';
            }
        }

        function getWeatherDescription(code) {
            // WMO Weather interpretation codes (Open-Meteo standard)
            const codes = {
                0: { text: 'Clear Sky', icon: 'fa-solid fa-sun' },
                1: { text: 'Mainly Clear', icon: 'fa-solid fa-cloud-sun' },
                2: { text: 'Partly Cloudy', icon: 'fa-solid fa-cloud-sun' },
                3: { text: 'Overcast', icon: 'fa-solid fa-cloud' },
                45: { text: 'Foggy', icon: 'fa-solid fa-smog' },
                51: { text: 'Light Drizzle', icon: 'fa-solid fa-cloud-rain' },
                61: { text: 'Rain Showers', icon: 'fa-solid fa-cloud-showers-heavy' },
                71: { text: 'Snowfall', icon: 'fa-solid fa-snowflake' },
                95: { text: 'Thunderstorm', icon: 'fa-solid fa-bolt' }
            };
            return codes[code] || { text: 'Fair Weather', icon: 'fa-solid fa-cloud-sun' };
        }
    </script>
</body>
</html>
