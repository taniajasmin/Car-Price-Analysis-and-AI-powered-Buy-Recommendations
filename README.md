# Car Price Analysis and AI-powered Buy Recommendations

## Overview

Car Profit Analyser is an end-to-end AI system that:

- Collects and cleans car listings from online marketplaces
- Uses mathematical & AI-based logic to analyze profits, depreciation, and car value
- Compares two cars in real time
- Provides chat-based recommendations to users via a natural conversation flow

It combines:

- Machine learning–style numeric analysis
- Mathematical modeling for car profit/depreciation
- Conversational AI
- Web scraping


## Screenshots

| Light Mode | Dark Mode |
|------------|-----------|
| ![Light Mode Screenshot](https://github.com/user-attachments/assets/9daff068-6d48-4e9e-9dd8-c61b8b909e07) | ![Dark Mode Screenshot](https://github.com/user-attachments/assets/0e7a5369-232b-48cf-ac01-d7c95c62f9cd) |

*Light mode (left) and dark mode (right) views of the Car Profit Analyser frontend interface.*


## AI Logic and Mathematical Breakdown

All math is implemented in `ai_calculations.py`.

### 1. Price Normalization

Used to convert irregular price data into a standard scale:

$$
\text{price\_score} = \frac{1}{1 + e^{-(x - \mu)/\sigma}}
$$

**Why:** Smooth out price variations between premium and economy cars.  
**Best for:** Nonlinear price scaling with natural balance.

### 2. Depreciation Formula

$$
\text{depreciation\_rate} = \frac{\text{current\_price}}{\text{original\_price}} = e^{-k \times \text{age}}
$$

**Why:** Models real-world car value drop over time.  
**Used for:** Calculating expected resale or profit margins.

### 3. Mileage Impact

$$
\text{mileage\_factor} = 1 - \frac{\text{mileage}}{200000}
$$

**Why:** Reduces car’s profitability as mileage increases.  
**Best for:** Simple linear wear model with real data consistency.

### 4. Premium Brand Adjustment

```python
premium_bonus = 1.15 if is_premium else 1.0
```

### 5. Profit Calculation

$$
\text{expected\_profit} = (\text{base\_price} \times \text{premium\_bonus}) - (\text{depreciation\_rate} \times 1000)
$$

**Why:** Core AI decision metric for “BUY”, “HOLD”, or “SELL”.  
**Best for:** Real-time profit ranking in analyze-cars.


## Folder Structure
```txt
CAR_PRICE_COMPARE/
│
├── app/
│   ├── init.py
│   ├── ai_calculations.py      # All mathematical and AI logic
│   ├── main.py                 # FastAPI main entry point
│   ├── models.py               # Pydantic data models
│   └── routes.py               # Modular route definitions
│
├── cars_data.csv               # Scraped dataset
├── scrape_cars.py              # Scraping script (from AutoScout24, 2dehands etc.)
├── index.html                  # Frontend interface
├── requirements.txt            # Dependencies
├── .env                        # Environment variables
└── README.md                   # This documentation
```

## .env Configuration

Create a `.env` file in the project root:
```text
OPENAI_API_KEY=your_openai_api_key_here
SCRAPE_URL_AUTOSCOUT=https://www.autoscout24.be/lst
SCRAPE_URL_2DEHANDS=https://www.2dehands.be
DEBUG=True
```

```text
Never upload `.env` to GitHub — add it to `.gitignore`.
```


## AI Chat Suggestion Flow

The `/ai-suggest/` endpoint is state-based and acts as a conversational car advisor.

| Stage | Description              | Example Question                                      |
|-------|--------------------------|-------------------------------------------------------|
| 0     | Greet user               | “Hi there! What type of car are you looking for?”     |
| 1     | Ask car type             | “Got it — sedan, SUV, or EV?”                         |
| 2     | Ask budget               | “What’s your budget range?”                           |
| 3     | Ask special needs        | “Do you prefer electric, low-maintenance, or resale value?” |
| 4     | Generate recommendations | “Based on your inputs, here are 3 cars…”              |

- Maintains chat history for contextual conversation
- Returns natural, flowing text replies

## API Endpoints

| Method | Endpoint             | Description                              |
|--------|----------------------|------------------------------------------|
| GET    | `/`                  | Health check                             |
| POST   | `/analyze-cars/`     | Analyze car list for profit & value      |
| POST   | `/compare-cars/`     | Compare two cars                         |
| POST   | `/ai-suggest/`       | AI conversational car advisor            |
| GET    | `/car-specs/{model}` | Fetch specs for a specific model         |


## CURL Examples
1. Analyze Cars
```bash
curl -X POST http://127.0.0.1:8000/analyze-cars/ \
  -H "Content-Type: application/json" \
  -d '{"cars":[{"title":"Tesla Model 3","price_numeric":25000,"year_numeric":2025,"mileage_numeric":10000,"brand":"Tesla","is_premium":1,"age":0}]}'
```
2. Compare Cars
```bash
curl -X POST http://127.0.0.1:8000/compare-cars/ \
  -H "Content-Type: application/json" \
  -d '{"car1":"Tesla Model 3 2025","car2":"BMW Z4 2025"}'
```
3. AI Suggestion Chat
```bash
curl -X POST http://127.0.0.1:8000/ai-suggest/ \
  -H "Content-Type: application/json" \
  -d '{"stage":0}'
```


## Web Scraping (scrape_cars.py)
### How It Works
Uses requests to fetch HTML pages from:
- AutoScout24
- 2dehands

Parses car listings using BeautifulSoup:
```python
soup = BeautifulSoup(html, "html.parser")
titles = [item.text for item in soup.select(".ListItem_title__...")]
```

Extracts:
- Title
- Price
- Year
- Mileage
- URL
Saves data to cars_data.csv.

### Why This Approach

- Lightweight: No Selenium needed
- Fast: Ideal for nightly cron jobs
- Clean: Structured output → perfect for analysis


## Frontend Description (index.html)
Built using pure HTML + JS

Features:

- Three sections: Analyze, Compare, Chat
- Dark mode toggle
- Pretty JSON output viewer
- Continuous AI chat (simulated memory)

## The AI chat is displayed as a conversation:
```text
Hi there! I’m your AI Car Advisor. What type of car are you looking for?
SUV for family
Got it — what’s your budget range?
```


## Tech Stack

| Layer       | Technology                  |
|-------------|-----------------------------|
| Backend     | FastAPI                     |
| Frontend    | HTML, JS, CSS               |
| AI Logic    | Python math + custom logic  |
| Parsing     | BeautifulSoup               |
| Storage     | CSV / JSON                  |
| Security    | .env configuration          |


## Why This Architecture Works Best

- Modular: All routes isolated under app/routes.py
- Mathematical transparency: Explicit formulas for every decision
- Scalable: Easy to replace AI logic with LLMs later
- User-friendly: Interactive chat UI + clear API


## Installation
```bash
pip install -r requirements.txt
uvicorn app.main:app --reload
```

## Frontend:
Open index.html in your browser → it connects automatically to the local FastAPI server.
