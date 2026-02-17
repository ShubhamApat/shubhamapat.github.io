+++
title = "BookMyShow Scraper"
date = 2025-04-05
tags = ["Web Scraping", "Selenium", "BeautifulSoup", "Python", "Multithreading"]
description = "How I scraped BookMyShow's highly secured platform and achieved 6x performance improvement using a clever Windows GUI trick"
+++

I had never made a scraper and the first scraper i was assigned to build was BookMyShow. I was asked to scrape the every event listed on  **all 1,898 cities** in India across 4 categories (plays, sports, events, activities). 

BookMyShow is **notoriously difficult** to scrape. They have:
- Advanced bot detection
- Dynamic JavaScript content
- Rate limiting
- CAPTCHA challenges

So as a newbie I started research on how to build a scraper from scratch, went on reddit, stack.com, medium and documentations of beautifulsoup, playwright and selenium. So I built a simple two step scraper, first step which can scrape just events urls from their cards using the selenium chromedriver and find the div class of card using beautifulsoup.

<img src="images/bms.png" | absURL>

The second step takes this scraped urls of cards, opens them one by one and scrape information like date, time, venue, language, age-limit, price and even about the event into a csv file. All of this using **BeautifulSoup4**.

<img src="images/bms-url.png" | absURL>

Only problem with the first attempt was that the first step was not able to scroll much further because BMS uses [lazy loading](https://www.cloudflare.com/learning/performance/what-is-lazy-loading/). Then again I started research about how to scrape sites that uses lazy loading for scrolling down the page to trigger the loading of dynamic content. Addition to lazy-loading, bookmyshow has an advance bot detection, if you keep scrolling to the bottom of the page for all the event cards at same pace it assumes you're bot and stops loading events!!

To tackle this I came up with my own human-like scroll method for bot after a lot of trial and errors. This human scroll makes the bot scroll the page to bottom till footer and as pages load more event cards it again scroll up to get the urls from those cards and then again repeat the whole loop till no more new cards are loaded, and the pace of scrolling is random everytime so that avoids bot detection. 

```python
def human_scroll(driver):
    actions = ActionChains(driver)
    for _ in range(random.randint(5, 8)):
        actions.send_keys(Keys.ARROW_DOWN).perform()
        time.sleep(random.uniform(0.08, 0.2))
```

The scraper was ready was ready but there are still optimizations needed, cause I needed to 1898 cities in 4 different categories(sports, events, plays, and activities). To scrape a city's events, sports, activities and plays page one by one was taking 120 seconds overall, now 120x1898 = 64 hours or 2.63 days. Ofcourse it needs optimization!

If you have read my other blogs, the zeroth step I take before optimise a pipeline is by measuring time taken by each function using time.time(). I got to know each categories page was taking around 35s to load and get scraped. To reduce, I decreased the sleep() time for loading and scrolling of scraper, which reduced 30s to around 25s but its still 100s overall. I couldn't decrease the sleep() time for scrolling beyond that because then it won't wait for lazy loading to load more events anymore and exit first.

Another optimization I applied was multithreading where I can scrape all 4 categories(events, sports, plays and activities) parallel that reduced the time 100s/4 t= 25s. With that I applied some methods to avoid collecting duplicate data where it does not save the same card url if thats scraped before which reduce unnecessary reads/writes, skip the city if there are no new cards and made scraper resumable. Only problem was we had to keep every window running parallel in foreground if the window is running in background BMS somehow stops loading events IKR!!

so i used windows 4-screens layout for scraping in parallel
```python
    driver_events.set_window_position(0, 0)       # top-left
    driver_sports.set_window_position(960, 0)     # top-right
    driver_activities.set_window_position(0, 540) # bottom-left
    driver_plays.set_window_position(960, 540)    # bottom-right
```
The final time was 18s per city which means 18*1898= 9.5 hours so finaly we optimized data scraping from 64 hours to 9.5 hours!

Here's a snap of my bookmyshow scraper:

<video controls width="100%" style="margin: 20px 0;">
    <source src="/bms.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>


## Key Takeaway

Sometimes the best optimization isn't faster code—it's a **different approach altogether**. 


---

*This scraper collected comprehensive event data from India's largest entertainment platform.*
