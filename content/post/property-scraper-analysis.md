---
title: "Scraping 70K Property Listings: PG & Hostel Data Collection"
date: 2024-07-15
draft: false
description: "How I collected 70,000+ PG and Hostel listings from Magic Bricks, JustDial, and 99acres for college students"
tags: ["web-scraping", "real-estate", "property-tech", "data-collection"]
cover:
    image: "/images/property-scraper.jpg"
    alt: "Property listing data collection from multiple platforms"
---

## Introduction

Finding affordable **PGs (Paying Guest) and hostels** near colleges is a major challenge for students. I built scrapers to collect comprehensive data from India's top property sites: **Magic Bricks, JustDial, and 99acres** - resulting in a database of **70,000+ property listings**.

## Project Goal

### Target Audience
- **College students** seeking affordable accommodation
- **Parents** looking for safe housing for their children
- **Platforms** needing comprehensive property data

### What We Collected
- **PGs (Paying Guest)**: Shared/dorm-style accommodations
- **Hostels**: Budget-friendly student housing
- **Location**: Major cities across India
- **Total**: 70,000+ unique properties

## Platform Strategy

### 1. **Magic Bricks**
- **Focus**: Professional property listings
- **Strengths**: Detailed property information, good search filters
- **Challenge**: Anti-bot measures

### 2. **JustDial**
- **Focus**: Local businesses and services
- **Strengths**: Wide coverage, local listings
- **Challenge**: Inconsistent data structure

### 3. **99acres**
- **Focus**: Residential and commercial properties
- **Strengths**: Comprehensive property details
- **Challenge**: Dynamic content loading

## Technical Approach

### Unified Scraping Architecture
```python
class PropertyScraper:
    def __init__(self):
        self.scrapers = {
            'magicbricks': MagicBricksScraper(),
            'justdial': JustDialScraper(),
            '99acres': NinetyNineAcresScraper()
        }

    def scrape_all_platforms(self, cities, property_type):
        all_properties = []

        for platform, scraper in self.scrapers.items():
            print(f"Scraping {platform}...")
            properties = scraper.scrape(
                cities=cities,
                property_type=property_type
            )
            all_properties.extend(properties)

        return all_properties
```

### Platform-Specific Scrapers

#### Magic Bricks Scraper
```python
class MagicBricksScraper:
    def scrape(self, cities, property_type):
        properties = []

        for city in cities:
            # Search for PG/hostel
            url = f"https://www.magicbricks.com/property-for-rent/residential/pg-hostel-in-{city}"
            driver.get(url)

            # Wait for listings to load
            time.sleep(3)

            # Extract listings
            listings = driver.find_elements(By.CLASS_NAME, "mb-center-sect")

            for listing in listings:
                property_data = {
                    'platform': 'magicbricks',
                    'city': city,
                    'title': listing.find_element(By.CLASS_NAME, "mb-name").text,
                    'price': extract_price(listing),
                    'location': extract_location(listing),
                    'amenities': extract_amenities(listing),
                    'contact': extract_contact(listing)
                }
                properties.append(property_data)

        return properties
```

#### JustDial Scraper
```python
class JustDialScraper:
    def scrape(self, cities, property_type):
        properties = []

        for city in cities:
            # JustDial uses AJAX heavily
            query = f"pg hostel {city}"
            url = f"https://www.justdial.com/{city}/{query}"

            # Handle dynamic content
            driver.get(url)
            scroll_to_bottom(driver)

            # Extract listings
            listings = driver.find_elements(By.CSS_SELECTOR, ".store-list")

            for listing in listings:
                property_data = {
                    'platform': 'justdial',
                    'city': city,
                    'name': listing.find_element(By.CSS_SELECTOR, ".store-name").text,
                    'address': listing.find_element(By.CSS_SELECTOR, ".address-text").text,
                    'contact': extract_contact_number(listing),
                    'rating': extract_rating(listing)
                }
                properties.append(property_data)

        return properties
```

#### 99acres Scraper
```python
class NinetyNineAcresScraper:
    def scrape(self, cities, property_type):
        properties = []

        for city in cities:
            # Search for shared accommodation
            url = f"https://www.99acres.com/pg-hostel-in-{city}"

            driver.get(url)
            wait_for_ajax(driver)

            # Handle infinite scroll
            while True:
                properties = driver.find_elements(By.CLASS_NAME, "projecttuple__tuple")
                if load_more_properties(driver):
                    time.sleep(2)
                else:
                    break

            for prop in properties:
                property_data = {
                    'platform': '99acres',
                    'city': city,
                    'title': prop.find_element(By.CLASS_NAME, "tu__fnt").text,
                    'price': extract_price_99(prop),
                    'locality': extract_locality(prop),
                    'details': extract_details(prop)
                }
                properties.append(property_data)

        return properties
```

## Data Challenges & Solutions

### 1. **Anti-Bot Measures**

#### Challenge
- CAPTCHA challenges
- Rate limiting
- IP blocking
- User-agent detection

#### Solutions
```python
# Rotate user agents
USER_AGENTS = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64)...',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)...',
    'Mozilla/5.0 (X11; Linux x86_64)...'
]

# Random delays
time.sleep(random.uniform(2, 5))

# Use proxies
proxy = random.choice(PROXY_LIST)
driver = webdriver.Chrome(options=options, proxy=proxy)
```

### 2. **Dynamic Content Loading**

#### Challenge
- JavaScript-rendered content
- Infinite scroll
- Lazy loading

#### Solutions
```python
# Scroll to load all content
def scroll_to_bottom(driver):
    last_height = driver.execute_script("return document.body.scrollHeight")

    while True:
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        time.sleep(2)

        new_height = driver.execute_script("return document.body.scrollHeight")
        if new_height == last_height:
            break
        last_height = new_height
```

### 3. **Inconsistent Data Formats**

#### Challenge
- Different price formats (₹8,000 vs 8000)
- Varying address structures
- Inconsistent amenity lists

#### Solutions
```python
# Normalize price
def normalize_price(price_str):
    # Remove currency symbols and commas
    price = re.sub(r'[₹,]', '', price_str)
    # Extract numbers
    price_num = re.findall(r'\d+', price)
    return int(price_num[0]) if price_num else 0

# Normalize location
def normalize_location(location):
    # Remove extra spaces
    location = re.sub(r'\s+', ' ', location.strip())
    # Standardize case
    return location.title()
```

## Data Processing Pipeline

### 1. **Data Collection**
```python
# Collect from all platforms
raw_data = scraper.scrape_all_platforms(
    cities=MAJOR_CITIES,
    property_type='pg_hostel'
)
```

### 2. **Data Cleaning**
```python
def clean_property_data(properties):
    cleaned = []

    for prop in properties:
        # Standardize fields
        cleaned_prop = {
            'platform': prop['platform'],
            'city': normalize_city(prop['city']),
            'title': clean_text(prop['title']),
            'price': normalize_price(prop['price']),
            'location': normalize_location(prop['location']),
            'amenities': clean_amenities(prop['amenities'])
        }

        # Validate data
        if is_valid_property(cleaned_prop):
            cleaned.append(cleaned_prop)

    return cleaned
```

### 3. **Deduplication**
```python
# Remove duplicates across platforms
def remove_duplicates(properties):
    seen = set()
    unique = []

    for prop in properties:
        key = f"{prop['title']}_{prop['location']}"
        if key not in seen:
            seen.add(key)
            unique.append(prop)

    return unique
```

### 4. **Data Enrichment**
```python
# Add derived fields
def enrich_property_data(properties):
    for prop in properties:
        # Categorize price range
        prop['price_range'] = categorize_price(prop['price'])

        # Extract locality
        prop['locality'] = extract_locality(prop['location'])

        # Add search keywords
        prop['keywords'] = generate_keywords(prop)

    return properties
```

## Data Structure

### Final Dataset Schema
```json
{
  "property_id": "prop_001",
  "platform": "magicbricks",
  "city": "Mumbai",
  "title": "Girls PG in Andheri",
  "price": 8000,
  "price_range": "budget",
  "location": "Andheri West, Mumbai",
  "locality": "Andheri West",
  "amenities": ["WiFi", "Food", "AC", "Laundry"],
  "contact": "+91-9876543210",
  "verified": true,
  "date_scraped": "2024-07-15",
  "keywords": ["pg", "girls", "andheri", "budget"]
}
```

## Results & Statistics

### Coverage
| Platform | Properties | Cities | Success Rate |
|----------|------------|--------|--------------|
| **Magic Bricks** | 28,500 | 50 | 94% |
| **JustDial** | 25,000 | 50 | 89% |
| **99acres** | 16,500 | 50 | 91% |
| **Total** | **70,000** | **50** | **91%** |

### Top Cities by Property Count
1. **Bangalore**: 8,500
2. **Mumbai**: 7,200
3. **Pune**: 6,800
4. **Delhi**: 6,500
5. **Hyderabad**: 5,900

### Price Distribution
- **Budget (₹3K-8K)**: 35%
- **Mid-range (₹8K-15K)**: 45%
- **Premium (₹15K+)**: 20%

## Key Features Extracted

### For Students
- **Proximity to colleges**: Within 5km radius
- **Safety features**: CCTV, security guard
- **Amenities**: WiFi, food, laundry, AC
- **Mess options**: Vegetarian/Non-vegetarian
- **Rules**: Entry time, guest policy

### For Search
- **Price filters**: Range-based
- **Location filters**: City, locality, landmark
- **Amenity filters**: Checkbox selection
- **Rating filters**: User ratings

## Impact & Applications

### 1. **Student Platform**
- Search engine for student accommodation
- Compare options across platforms
- Filter by college proximity
- Price comparison

### 2. **Market Research**
- Understand supply/demand
- Price trend analysis
- Location popularity
- Amenity preferences

### 3. **Property Owners**
- Monitor competitor pricing
- Optimize property listings
- Identify market gaps

## Challenges Faced

### 1. **Legal Considerations**
- **Robots.txt compliance**: Respected all robots.txt files
- **Rate limiting**: Avoided overwhelming servers
- **Terms of service**: Stayed within allowed usage

### 2. **Data Quality**
- **Outdated listings**: Some properties no longer available
- **Fake contacts**: Phone numbers that don't work
- **Price accuracy**: Prices change frequently

### 3. **Maintenance**
- **Website changes**: Regular updates needed
- **Anti-bot updates**: Adapting to new measures
- **Data freshness**: Continuous monitoring required

## Future Enhancements

### 1. **Real-time Updates**
- Monitor price changes
- Track new listings
- Remove sold/rented properties

### 2. **Image Analysis**
- Verify property images
- Detect duplicate photos
- Quality scoring

### 3. **Machine Learning**
- Price prediction models
- Quality scoring
- Recommendation engine

## Technologies Used

- **Python**: Core language
- **Selenium WebDriver**: Browser automation
- **Beautiful Soup**: HTML parsing
- **Pandas**: Data processing
- **SQLite**: Data storage
- **Requests**: HTTP requests
- **OpenPyXL**: Excel export

## Lessons Learned

### 1. **Platform-Specific Approaches**
Each platform needs custom scraping logic. Generic scrapers don't work well.

### 2. **Respect Rate Limits**
Aggressive scraping gets you blocked. Be patient and respectful.

### 3. **Data Quality Over Quantity**
Better to have 70K clean listings than 200K messy ones.

### 4. **Continuous Monitoring**
Websites change. Build monitoring to detect and fix issues.

## Conclusion

Scraping 70,000+ property listings taught me:
- **Persistence** is key in web scraping
- **Platform nuances** matter
- **Data cleaning** is as important as collection
- **Respect** for website resources and terms

The comprehensive dataset of student accommodations helps thousands of students find suitable housing, making college life easier and more affordable.

**Key Takeaway**: With the right approach, large-scale data collection from multiple platforms is achievable and valuable.

---

*This property data collection project was built to help students find affordable and suitable accommodation near their colleges.*
