+++
title = "I bypassed Google reCAPTCHA and LinkedIn's Anti-Scraping"
date = 2024-10-20
tags = ["Web Scraping", "Social Media", "LinkedIn", "Google Search", "Recaptcha Bypass"]

[cover]
image = "/images/ReCaptcha-blog-article-screen-2.png"
alt = "Description of the image"
+++

LinkedIn doesn't want you scraping their data. But I needed to, not just because I was asked to but I like the challenge! What I wanted to scrape? Posts about specific topics e.g., "LLMs","transformers",...., anything people posts about on linkedin.  

**The catch**: No login required, and it has to bypass anti-bot measures.

## The Approach: Google Search Strategy
But for that purpose I need a search engine first, so I used google's search with "site:linkedin.com/posts llms" and it gave me links to all the popular posts about llms on linkedin.

Instead of hitting LinkedIn directly, I go through Google. I used python libraries selenium and BeautifulSoup4 to build the scraper. 

```python
def google_worker(driver, keyword, entity, location, start_page, end_page, out_urls, lock):
    query = f"site:linkedin.com/ {keyword}"
    if location: query += f" {location}"

    for page in range(start_page, end_page+1):
        url = "https://www.google.com/search?q=" + urllib.parse.quote_plus(query) + f"&start={(page-1)*10}"
        driver.get(url); human_pause()

        if "recaptcha" in driver.page_source.lower():
            solve_recaptcha(driver)

        soup = BeautifulSoup(driver.page_source, "html.parser")
        for a in soup.select("a"):
            href = a.get("href")
            if not href or not href.startswith("http"): continue
            clean = href.split("&")[0]
            if entity == "company" and "linkedin.com/company" in href:
                with lock: out_urls.append(clean)
            elif entity == "posts" and "/posts/" in href:
                with lock: out_urls.append(clean)
            elif entity == "jobs" and "/jobs/" in href:
                with lock: out_urls.append(clean)
```

## The reCAPTCHA Breakthrough
This indirect approach avoids LinkedIn's anti-bot detection because Google is doing the "scraping." But there is just one big problem, How to avoid Google's anti-bot detection, with only two attempt on running the script google started with recaptcha! Now a bot can't solve the recaptcha byitself! Most of Web Scrappers back off after the recaptch comes. But what if I tell you I have found the way to **bypass recaptcha** 

```python
def solve_recaptcha(driver):
        frames = driver.find_elements(By.TAG_NAME, "iframe")
        for frame in frames:
            if "recaptcha" in (frame.get_attribute("src") or ""):
                driver.switch_to.frame(frame)
                break
        checkbox = WebDriverWait(driver, 5).until(
            EC.element_to_be_clickable((By.ID, "recaptcha-anchor"))
        )
        checkbox.click(); human_pause(2, 4)
        driver.switch_to.default_content()

        challenge_iframe = None
        for frame in driver.find_elements(By.TAG_NAME, "iframe"):
            src = frame.get_attribute("src") or ""
            title = frame.get_attribute("title") or ""
            if "recaptcha" in src and "anchor" not in src:
                challenge_iframe = frame; break
            if "recaptcha challenge" in title.lower():
                challenge_iframe = frame; break

        if not challenge_iframe:
            print("reCAPTCHA solved by checkbox only")
            return True

        print("Challenge detected → switching to audio")
        driver.switch_to.frame(challenge_iframe)
        audio_btn = WebDriverWait(driver, 5).until(
            EC.element_to_be_clickable((By.ID, "recaptcha-audio-button"))
        )
        audio_btn.click(); human_pause(2, 4)
        audio_src = WebDriverWait(driver, 5).until(
            EC.presence_of_element_located((By.ID, "audio-source"))
        ).get_attribute("src")

        audio_file, wav_file = "captcha.mp3", "captcha.wav"
        with open(audio_file, "wb") as f:
            f.write(requests.get(audio_src).content)
        sound = AudioSegment.from_mp3(audio_file)
        sound.export(wav_file, format="wav")
        recognizer = sr.Recognizer()
        with sr.AudioFile(wav_file) as source:
            audio_data = recognizer.record(source)
            text = recognizer.recognize_google(audio_data)
            print("🎤 Recognized:", text)

        input_box = driver.find_element(By.ID, "audio-response")
        input_box.send_keys(text)
        driver.find_element(By.ID, "recaptcha-verify-button").click()
        human_pause(3, 5)
        driver.switch_to.default_content()
        print(" reCAPTCHA solved with audio")
        return True
```

So how does bot bypasses recaptcha because the pattern and images always random in recaptcha? We don't solve recaptcha, we chose the option with icon of headphones bottom of the captcha, which is actually an accessibility for people where they can hear what the speech is and then write it in the text box. We download that speech file, use **Google's speech-to-text** to convert that speech to text and then make the bot input it.

Here's a demo of the LinkedIn scraper in action:

<video controls width="100%" style="margin: 20px 0;">
    <source src="/recaptcha.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

Yes **its exactly like I used google to defeat google**

<img src="/images/thanos2.jpg" alt="Thanos with custom text" width="600">

## Features
Not just that it can scrap posts, I add the functionality where we can scrape a company profile and jobs too. Where companies profile do require linkedin's login otherwise we just hit linkedin authwall!

In companies profile, with the help of beautifulsoup4 scraper can collect images in the posts made by company, I have used human-like behaviour to avoid linkedin security measures for bot detection.   
```python
def scrape_company(driver, urls, output_file="companies.csv", img_folder="images"):
    all_data = []
    for url in urls[:5]:
        driver.get(url); human_pause(0.5, 0.8)
        soup = BeautifulSoup(driver.page_source, "html.parser")

        # download company media images
        for img in soup.select("img"):
            src = img.get("src")
            if src and "media.licdn.com" in src:
                download_image(src, img_folder, driver)

        name = soup.select_one("h1.top-card-layout__title")
        headline = soup.select_one("h2.top-card-layout__headline")
        location = soup.select_one("h3.top-card-layout__first-subline")
        about = soup.select_one("section[data-test-id='about-us'] p")
        size = soup.select_one("div[data-test-id='about-us__size'] dd")
        industry = soup.select_one("div[data-test-id='about-us__industry'] dd")
        founded = soup.select_one("div[data-test-id='about-us__foundedOn'] dd")

        all_data.append({
            "url": url,
            "name": name.get_text(strip=True) if name else "",
            "headline": headline.get_text(strip=True) if headline else "",
            "location": location.get_text(strip=True) if location else "",
            "about": about.get_text(strip=True) if about else "",
            "size": size.get_text(strip=True) if size else "",
            "industry": industry.get_text(strip=True) if industry else "",
            "founded": founded.get_text(strip=True) if founded else "",
        })
    with open(output_file, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=["url","name","headline","location","about","size","industry","founded"])
        writer.writeheader(); writer.writerows(all_data)
    print(f" Saved {len(all_data)} companies → {output_file}")
```

```python 
def scrape_jobs(driver, urls, output_file="jobs.csv"):
    all_data = []
    for url in urls:
        driver.get(url); human_pause(0.5, 1)
        soup = BeautifulSoup(driver.page_source, "html.parser")
        for li in soup.select("ul.jobs-search__results-list li"):
            title = li.select_one("h3")
            company = li.select_one("h4 a")
            location = li.select_one(".job-search-card__location")
            date = li.select_one("time")
            all_data.append({
                "url": url,
                "title": title.get_text(strip=True) if title else "",
                "company": company.get_text(strip=True) if company else "",
                "location": location.get_text(strip=True) if location else "",
                "date": date.get_text(strip=True) if date else ""
            })
    with open(output_file, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=["url","title","company","location","date"])
        writer.writeheader(); writer.writerows(all_data)
    print(f" Saved {len(all_data)} jobs → {output_file}")

```


```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="LinkedIn Scraper with multiwindow + captcha solver")
    parser.add_argument("mode", choices=["posts","company","jobs"])
    parser.add_argument("keyword", type=str)
    parser.add_argument("--location", type=str, default=None)
    parser.add_argument("--pages", type=int, default=1)
    parser.add_argument("--output", type=str, default="out.csv")
    parser.add_argument("--img_folder", type=str, default="images", help="Folder to save downloaded images")
    args = parser.parse_args()
```
### 1. **Topic-Based Posts**
scrape first 50 pages with links of linkedin posts that google search query shows
```cli
python linkedin_scraper.py --mode posts llms --pages 50 --output llms.csv 
```
### 2. **User Profiles**
Extract profile info without login
```cli
python linkedin_scraper.py --mode company openai --pages 1 --output openai.csv 
```

### 3. **Job Listings**
Find Data Scientist jobs in Mumbai
```cli
python linkedin_scraper.py --mode jobs Data Scientist mumbai --pages 1 --output DS_jobs.csv 
```

## Key Takeaway

Sometimes the best way to scrape a protected site is to **not scrape it directly**. Go through Google, use CAPTCHA-solving services, and mimic human behavior.

---

*This scraper enables data extraction from LinkedIn without authentication or account requirements.*
