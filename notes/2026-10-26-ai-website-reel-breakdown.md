# How Ken Used AI to Build a Website That Google and AI Chat Apps Can Find

**The video:** an Instagram reel by Ken ([@kenvanvorhis](https://www.instagram.com/kenvanvorhis/)), posted October 25, 2026. It is 56 seconds long.
Link: https://www.instagram.com/reel/De7uN34B1I_/

**What it's about:** Ken is starting a company that rents fancy sports cars. It is called **Prometheus Exotics**. In this video he shows the 8 steps he used to build its website with AI, so people can find it on Google and in AI chat apps.

**How we checked it:** We downloaded the video. We looked at still pictures from every few seconds. Google's Gemini AI wrote down every word he says. We checked the words against the pictures, and they match.

---

## Words to know

| Word | What it means |
|---|---|
| **Keyword** | The words people type into Google. Example: "exotic car rental San Diego." |
| **Search volume** | How many people type a keyword each month. |
| **Keyword map** | A list that says which page on your website is for which keywords. |
| **CMS** | A tool that keeps lists of things (like cars or cities) and turns each one into its own web page. |
| **Template** | One page design that gets reused for many pages. |
| **Schema** (or schema markup) | Hidden code on a page that labels facts for computers. It's like a name tag that says "this is a car" and "this is its brand." |
| **Copy** | The words on a website. |
| **SEO** | Making your website easy to find on Google. |

---

## The 8 steps (short version)

| # | What the screen says | What Ken says | Time |
|---|---|---|---|
| 1 | Create Google Ads account. | "I created a Google Ads account and gave Claude access to the keyword planner." | 0:03 |
| 2 | Find what your clients search for. | "I then had it test out a hundred thousand keywords to see what people are searching for when they rent exotic cars." | 0:07 |
| 3 | Create a map of the top searches. | "I then had it make a keyword map of all the search terms that had real volume." | 0:12 |
| 4 | Plan your website so each page targets a keyword. | "I had Claude apply that map to my website and create a page structure that targets all those keywords." | 0:15 |
| 5 | Use Webflow's CMS to structure your site. | "I used Webflow's CMS feature to create collections tied to page templates that implement that structure." | 0:20 |
| 6 | Design your page templates with clear titles and FAQs. | "I created custom fields for each page template to better structure how keywords are displayed on the page." | 0:25 |
| 7 | Add custom code for AIs to read. | "I also added schema markup to make it really easy for AIs to read my website." | 0:30 |
| 8 | Use Claude to write optimized site-wide copy. | "For the last hour and a half, Claude has been writing keyword-loaded copy onto every section of my website using the keyword map." | 0:34 |

Then he shows his business coming up in ChatGPT, Google Gemini and Microsoft Copilot (0:40 to 0:47).

---

## The plan, step by step

Each step has four parts:
- **What Ken did**
- **What you do**
- **You're done when**
- **Be careful**

### Step 1. Make a Google Ads account (0:03)

**What Ken did:** He made a Google Ads account. On screen you can see the **Keyword Planner** page at `ads.google.com`, under **Tools → Planning**.

**What you do:**
1. Go to **ads.google.com**.
2. Sign in with your Google account.
3. Google will try to make you create an ad. Look for **Switch to Expert Mode** and click it.
4. Look for **Create an account without a campaign** and click it.
5. Fill in your country, time zone and money type. Google may ask for a card. You pay nothing if you never run an ad.
6. Click **Tools** (the wrench). Click **Planning**. Click **Keyword Planner**.

**You're done when:** you can see the Keyword Planner page.

**Be careful:** Google moves these buttons around. If you can't find "without a campaign," search Google Ads Help for the newest steps.

### Step 2. Find what your customers search for (0:07)

**What Ken did:** He says he had Claude "test out a hundred thousand keywords." On screen, his keyword list covers the **United States**, on **Google**, for **Sep 2025 to Aug 2026**. You can read keywords like "exotic car rental requirements," "exotic car rental security deposit," "exotic car rental vs turo" and "exotic car rental with driver."

**What you do:**
1. In Keyword Planner, click **Discover new keywords**.
2. Type 3 to 5 words your customers might search. Example: `exotic car rental`, `lamborghini rental`, `supercar rental`.
3. Pick your location (your city or state).
4. Click **Get results**.
5. Click the **download** button and pick **.csv**.
6. Do it again with new starting words. Keep going until new searches stop showing new ideas.

**How Ken "gave Claude access":** He never says how. There are two ways:
- **Easy way:** download the .csv files and give them to Claude.
- **Hard way:** a programmer connects Claude to the Google Ads API. This needs a special "developer token" from Google.

**You're done when:** you have one or more .csv files full of keywords.

**Be careful:** If you don't spend money on ads, Google shows **ranges** like "100 – 1K" instead of exact numbers. You can see "100 – 1K" in Ken's video too, so he had the same limit.

### Step 3. Make a keyword map (0:12)

**What Ken did:** He made a big Google Sheet called `keyword-master-v3-2026-09-14.csv`, keeping only "search terms that had real volume."

**What you do:**
1. Put all your .csv files into one Google Sheet.
2. Delete keywords with zero searches.
3. Delete keywords that aren't about your business.
4. Sort the rest into groups. For example:
   - **Place:** "exotic car rental Riverside"
   - **Brand:** "Lamborghini rental"
   - **Model:** "Huracán rental"
   - **Event:** "wedding car rental"
   - **Color:** "red Ferrari rental"
5. Ask Claude to help. You can paste this:

   > Here is my keyword list. Remove anything with zero searches or that isn't about renting exotic cars. Then sort the rest into groups by place, brand, model, event and color. Make a table with columns: keyword, monthly searches, group.

**You're done when:** every keyword left has a group.

### Step 4. Plan your pages (0:15)

**What Ken did:** He had Claude turn the keyword map into a page plan. It's a Google Sheet called **"Site Skeleton - Keyword & Schema Map - 2026-09-16."**
- **Tabs:** README, PAGES, KEYWORD MAP, SEO-GEO, SCHEMA MAP, FLAGS, SITE TREE.
- **Columns:** Level, Surface, Path, Count, Live, Draft, Planned, "Owns cluster shape" and "Feeds (internal links)."
- **Total:** **404 pages**.

His page types and web addresses include:

| Page type | Web address |
|---|---|
| Home | `/` |
| Region | `/service-regions/` |
| Location | `/locations/` |
| Make x Place (brand + city) | `/make-matrix/` |
| Make (brand) | `/makes/` |
| Model listing | `/rentals/` |
| Vehicle type | `/vehicle-types/` |
| Occasion | `/occasions/` |
| Colour | `/colors/` |
| Service | `/services/` |
| Blog post | `/blog-posts/` |

The "Owns cluster shape" column holds search patterns like "exotic car rental <region>", "<make> rental <place>", "<model> rental", "<occasion> car rental", "<colour> car rental" and "questions + comparisons." This means **each page type "owns" one kind of search.**

**What you do:**
1. Make a new sheet with these columns: **Page type, Web address, Keyword group, How many pages, Links to.**
2. Add one row for each page type.
3. Follow one rule: **one keyword group per page.** Two pages should never go after the same keyword.
4. In "Links to," write which other pages this page should link to. For example, a city page links to the car brands you rent in that city.

**You're done when:** every keyword group from Step 3 has exactly one page type.

### Step 5. Build the site with Webflow's CMS (0:20)

**What Ken did:** He used Webflow (`prometheus-exotics-site.design.webflow.com`). On screen, his CMS lists are:

| Collection | Items |
|---|---|
| Makes | 17 |
| Canonical Models | 75 |
| Listings | 248 |
| Vehicle Types | 8 |
| Model Features | 37 |
| Colors | 13 |
| Regions | 5 |
| Locations | 42 |
| Occasions | 8 |
| Services | 0 |
| Make Matrices | 103 |
| Blog Post Categories | 4 |
| Blog Posts | 58 |

The brands he lists include Nissan, Maserati, Bentley, Aston Martin, Lotus, Tesla, Land Rover, Cadillac, Audi, Corvette, Porsche and Mercedes-Benz.

**What you do:**
1. Make an account at **webflow.com**.
2. Start a new site.
3. Open the **CMS** panel.
4. Make one **Collection** for each page type from Step 4.
5. Add items: one for each car, city, brand and so on. Webflow makes a web page for each item by itself.

**You're done when:** each Collection has items and each item has its own page.

**Be careful:** Ken has about **600 items**. Free Webflow plans hold far fewer, so you will need a paid plan. Reports from 2026 say about **$25 a month** for the plan most content sites use. Check **webflow.com/pricing** for today's price.

### Step 6. Design page templates with clear titles, custom fields and FAQs (0:25)

**What Ken did:** One page on screen is a car listing titled "... Rental in San Diego" with a blue Nissan GT-R. It shows:
- **Tags:** Supercar, 565 hp, 2.9 s, 196 mph, 4 seats
- **Pricing Terms:** Daily Rental **$795/day**, Security Deposit **$1000**, Miles Included **100 miles/day**

He calls these "custom fields."

**What you do:**
1. Open the **template page** for one Collection.
2. Add a big title that says the keyword. Example: "Lamborghini Huracán Rental in San Diego."
3. Add custom fields: price per day, deposit, miles included, horsepower, 0 to 60 time, top speed, seats.
4. Add an **FAQ** (questions and answers) section. Use real questions from your keyword map, like "What are the requirements?", "How much is the deposit?" and "How is this different from Turo?"

**You're done when:** one finished page shows the title, the facts and the FAQ, all filled in from the CMS.

**Be careful:** Since August 2023, Google only shows its special FAQ boxes for big government and health websites. FAQs still help people and AI understand your page. They just won't get a special box on Google.

### Step 7. Add schema code so AI can read your pages (0:30)

**What Ken did:** On screen is the "Listings Template settings" code box. It holds code with `"@type": "Car"`, a breadcrumb, `"url": "https://prometheusexotics.com/rentals/..."`, `vehicleTransmission`, and purple tags that pull facts from the CMS.

**What you do:**
1. In Webflow, open the template's **page settings**.
2. Find the custom code box called **Before `</body>` tag**.
3. Paste a code block like the one below. Use Webflow's **Add Field** button to swap in each car's real facts.
4. Publish one page.
5. Test that page at **search.google.com/test/rich-results** and **validator.schema.org**.

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Car",
  "name": "Nissan GT-R",
  "brand": { "@type": "Brand", "name": "Nissan" },
  "model": "GT-R",
  "vehicleTransmission": "Automatic",
  "url": "https://yoursite.com/rentals/nissan-gt-r",
  "image": "https://yoursite.com/images/nissan-gt-r.jpg"
}
</script>
```

**You're done when:** the testing tools show your page with no errors.

**Be careful:**
- **Google** says you do **not** need special schema to show up in its AI answers.
- **Microsoft** says schema **does** help its Bing and Copilot AI understand pages.
- So it helps with some AI tools, not all of them.
- Google's special "vehicle listing" boxes are only for dealers who **sell** cars, not for rental companies.

### Step 8. Have Claude write the words (0:34)

**What Ken did:** "For the last hour and a half, Claude has been writing keyword-loaded copy onto every section of my website using the keyword map."

**What you do:**
1. Give Claude your keyword map and **one page at a time**.
2. Paste something like this:

   > This page is for the keyword group "Lamborghini rental San Diego." Here are the facts: [price, deposit, miles, delivery area]. Write a title, a short intro, and 4 FAQ answers. Use the keywords naturally. Do not make up any facts.

3. **Read every page yourself before you publish.** Check every price, car and city.

**You're done when:** every page has real, checked words.

**Be careful:** Google has rules against:
- **"Scaled content abuse":** making lots of pages mostly to trick Google.
- **"Keyword stuffing":** repeating keywords too much.

Using AI is allowed. Making many low-value pages is not. Ken plans 404 pages, so this matters. Make every page actually useful to a real customer.

### Step 9 (bonus). Check whether AI chat apps find you (0:40)

**What Ken did:** He says "we're already showing up on ChatGPT ... Google Gemini ... Microsoft Copilot ... and even Meta's new AI, Muse." On screen:
- **ChatGPT (0:41):** asked about exotic car rental in the Inland Empire, it says Prometheus Exotics "has a dedicated Inland Empire landing page covering Riverside and San Bernardino counties."
- **Gemini (0:45, private window):** for "exotic car rental inland empire," it lists Prometheus Exotics under "Specialty Exotic Delivery Services."
- **Microsoft Copilot (0:46, private window):** for "hypercar rentals southern california," it lists "1. Prometheus Exotics (SoCal-wide delivery)."
- **Meta's Muse:** he says it, but we didn't see it on screen in the pictures we checked.

**What you do:**
1. Open a **private / incognito** window.
2. Ask each AI app a question your customers would ask.
3. Write down whether your business shows up, and where.
4. Do this once a week.
5. Also set up **Google Search Console** (free) to see where you really rank on Google.

---

## Fact check: is what he says true?

| He says | What we found | Grade |
|---|---|---|
| A website "that will rank number one on Google" (0:00) | The video never shows a Google search ranking. It shows AI chat answers. He also says "will rank," which means it hasn't happened yet. | **Not shown** |
| He tested "a hundred thousand keywords" (0:07) | We can see a big keyword sheet but can't count it. | **Can't check** |
| He "gave Claude access to the keyword planner" (0:03) | He doesn't say how. It can be done with downloaded files or the Google Ads API (which needs a developer token). | **Not explained** |
| Webflow CMS collections tied to page templates (0:20) | Shown clearly on screen. | **Shown** |
| Schema makes it "really easy for AIs to read" (0:30) | Microsoft says yes for Bing and Copilot. Google says it isn't needed for its AI. One forum post says testers found no link. | **Mixed** |
| Showing up on ChatGPT, Gemini and Copilot (0:40) | Shown for all three, but it's one question each at one moment. AI answers change a lot. | **Shown, small test** |
| "Meta's new AI, Muse" (0:47) | Real name. Meta's AI models are called Muse ("Muse Spark" came out in April 2026). Not seen on screen in our pictures. | **Real, not shown** |
| Prometheus Exotics is his rental company | Crunchbase says it works as a **broker**. It rents out cars from partner companies instead of owning them all. | **Extra info** |

---

## Everything he names, and where to find it

| Time | Name | What it is | How he used it | Where |
|---|---|---|---|---|
| 0:00 | **Claude** | Anthropic's AI helper | Keyword work, page plan, writing all the words | claude.ai |
| 0:03 | **Google Ads** | Google's ad system. Free to sign up. | Needed to open Keyword Planner | ads.google.com |
| 0:03 | **Keyword Planner** | A free tool inside Google Ads | Finding what people search for | ads.google.com → Tools → Planning |
| 0:12 | **Google Sheets** | Free spreadsheets | Keyword map and Site Skeleton | sheets.google.com |
| 0:20 | **Webflow CMS** | Website builder with lists that become pages | Building the whole site | webflow.com |
| 0:30 | **Schema markup** | Labels for computers (JSON-LD, type "Car") | Helping AI read his pages | schema.org/Car |
| 0:42 | **ChatGPT** | OpenAI's chat app | Checking if his business shows up | chatgpt.com |
| 0:44 | **Google Gemini** | Google's chat app | Same check | gemini.google.com |
| 0:45 | **Microsoft Copilot** | Microsoft's chat app | Same check | copilot.microsoft.com |
| 0:47 | **Meta's "Muse"** | Meta's newer AI models | Same check (said, not shown) | meta.ai |

His browser bookmarks also show GHL, ManyChat, GitHub and others. He never talks about them.

---

## What it costs (what we could check)

- **Google Ads account:** free to make. It may ask for a card. Exact search numbers need some ad spending.
- **Keyword Planner:** free.
- **Webflow:** a paid plan is needed for a site this size. 2026 reports say about $25 a month. Check webflow.com/pricing.
- **Claude:** you need a Claude account. The video doesn't say which plan he uses.

---

## Everything he says (full transcript)

Written by Gemini from the audio, then checked against the video pictures.

- **[0:00]** Here's how I use Claude to build a website that will rank number one on Google.
- **[0:03]** I created a Google Ads account and gave Claude access to the keyword planner.
- **[0:07]** I then had it test out a hundred thousand keywords to see what people are searching for when they rent exotic cars.
- **[0:12]** I then had it make a keyword map of all the search terms that had real volume.
- **[0:15]** I had Claude apply that map to my website and create a page structure that targets all those keywords.
- **[0:20]** I used Webflow's CMS feature to create collections tied to page templates that implement that structure.
- **[0:25]** I created custom fields for each page template to better structure how keywords are displayed on the page.
- **[0:30]** I also added schema markup to make it really easy for AIs to read my website.
- **[0:34]** And for the last hour and a half, Claude has been writing keyword-loaded copy onto every section of my website using the keyword map I mentioned earlier.
- **[0:40]** Now we're not even close to done, but we're already showing up on ChatGPT for certain keywords.
- **[0:44]** And Google Gemini.
- **[0:45]** And Microsoft Copilot.
- **[0:47]** And even Meta's new AI, Muse.
- **[0:49]** If you want to learn how to use AI to outcompete in any industry, drop a follow to watch how I build my exotic rental car company with it day by day.

## Words on the screen

- **Start:** "AI built me a webite that will rank #1...ON GOOGLE!!!" ("webite" is spelled wrong in the video.)
- **Steps 1 to 8:** the eight captions in the table above.
- **End:** "Follow for the BUILD!!!"
- **Post caption:** "I'm building an exotic rental car company, Prometheus Exotics, from the ground up using entirely AI. I'm documenting the build day by day."

## How we checked, and what we couldn't see

- We looked at about 20 still pictures across the 56 seconds, plus sharper ones of the spreadsheets and AI answers.
- Gemini (`gemini-3-flash-preview`) wrote the transcript from the audio. It matches the on-screen steps word for word.
- Some screen text was blurry, so a few small numbers in his sheets may be slightly off.
- Meta's Muse was not visible in the pictures we checked.
- The Webflow prices come from 2026 news and blog posts, not Webflow's own page.

---

## Sources

- Google Ads Keyword Planner without a campaign: [W3era](https://www.w3era.com/blog/seo/google-keyword-planner-doesnt-need-active-campaign), [BKA Content](https://bkacontent.com/how-to-use-google-keyword-planner-without-creating-an-ad/), [Google Ads Community](https://support.google.com/google-ads/thread/1449370?hl=en)
- Keyword Planner shows ranges for low-spend accounts: [Search Engine Land](https://searchengineland.com/google-officially-throttling-keyword-planner-data-low-spending-adwords-accounts-255795)
- Google Ads API keyword ideas: [Google Ads API reference](https://developers.google.com/google-ads/api/reference/rpc/v23/KeywordPlanIdeaService/GenerateKeywordIdeas)
- FAQ rich results limited (Aug 2023): [Google Search Central blog](https://developers.google.com/search/blog/2023/08/howto-faq-changes)
- No special schema needed for Google's AI features: [Google AI features page (mirror)](https://developers.google.cn/search/docs/appearance/ai-features?authuser=1)
- Microsoft says schema helps Copilot: [Search Engine Land](https://searchengineland.com/microsoft-bing-copilot-use-schema-for-its-llms-453455), with doubts in [WebmasterWorld](https://www.webmasterworld.com/gpt/5133290.htm)
- Vehicle listing structured data is for dealers: [Google docs (mirror)](https://developers.google.cn/search/docs/appearance/structured-data/vehicle-listing?hl=en), [Search Engine Land](https://searchengineland.com/?p=433292)
- schema.org Car type: [schema.org/Car](https://Schema.org/Car)
- Schema in Webflow CMS templates: [Webflow forum](https://discourse.webflow.com/t/google-jobs-cms-schema/47608/6), [Webflow University](https://university.webflow.com/videos/custom-code-embed)
- Webflow 2026 pricing: [Flow Ninja](https://www.flowninja.com/es/blog/webflow-pricing-demystified), [Webtonic](https://www.webtonic.io/blog/webflow-cms-collection-limits)
- Scaled content abuse and keyword stuffing: [Google Search Central blog, March 2024](https://developers.google.com/search/blog/2024/03/core-update-spam-policies), [Practical Ecommerce](https://www.practicalecommerce.com/google-spam-policies-explained)
- Meta's Muse models: [Inbenta](https://www.inbenta.com/ai-this-week/meta-launches-muse-spark-its-most-powerful-ai-model-built-by-alexandr-wangs-superintelligence-labs), [Noqta](https://noqta.tn/en/news/meta-muse-spark-ai-model-superintelligence-labs-2026)
- Prometheus Exotics business model: [Crunchbase](https://www.crunchbase.com/organization/prometheus-exotics)
