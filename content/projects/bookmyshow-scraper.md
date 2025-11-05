+++
title = "BookMyShow Scraper - I Got 60x Speedup by Running 4 Windows Instead of Threads"
date = 2024-04-05
tags = ["Web Scraping", "Selenium", "BeautifulSoup", "Python", "Multithreading"]
description = "How I scraped BookMyShow's highly secured platform and achieved 60x performance improvement using a clever Windows GUI trick"
+++

## The Challenge

BookMyShow is **notoriously difficult** to scrape. They have:
- Advanced bot detection
- Dynamic JavaScript content
- Rate limiting
- CAPTCHA challenges

I needed to scrape **all 1,898 cities** in India across 4 categories (plays, sports, events, activities).

## The Problem

First attempt: Single-threaded Python script
- **Result**: 3 hours per city
- **Total time**: ~6,000 hours (way too slow)

```python
# This was painfully slow
for city in cities:
    events = scrape_city(city)  # 3 hours per city!
```

Tried multithreading, but threads running in the background got **blocked by BookMyShow's detection**.

## The Innovation: Windows Multi-Window Layout

The breakthrough: instead of running threads in the background, I ran **4 windows simultaneously in the foreground**.

```python
# Instead of this (blocked):
threads = []
for url in urls:
    t = Thread(target=scrape, args=(url,))
    t.start()  # Background = blocked
    threads.append(t)
t.join()

# Do this (60x faster):
for i in range(4):
    driver = webdriver.Chrome()
    driver.set_window_position(i * 400, 0)  # Tile windows
    driver.get(urls[i])
    scrape_parallel(driver, urls[i:i+4])  # 4 cities at once!
```

Each window scraped a different set of cities **simultaneously**. No background thread blocking!

## The Results

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **Time per city** | 3 hours | 3 minutes | **60x faster** |
| **All 1,898 cities** | 6,000 hours | 100 hours | Feasible! |

## Anti-Detection Strategy

```python
def human_like_behavior(driver):
    # Human-like scrolling
    for _ in range(5):
        driver.execute_script("window.scrollBy(0, 500);")
        time.sleep(random.uniform(1, 3))
    
    # Random delays
    time.sleep(random.uniform(2, 5))
    
    # Mouse movements
    action = ActionChains(driver)
    action.move_by_offset(random.randint(0, 100), random.randint(0, 100))
    action.perform()
```

## Coverage

- **1,898 cities** across India
- **4 categories**: Plays, Sports, Events, Activities
- **Success rate**: 94%

## Key Takeaway

Sometimes the best optimization isn't faster code—it's a **different approach altogether**.

Instead of fighting the system, I worked with it. Windows GUI automation beat multithreading because it looked like a real user.

---

*This scraper collected comprehensive event data from India's largest entertainment platform.*
