# 📚 Bookstore Intelligence Mission — Competitor Web Scraper

A Python web scraper that turns a competitor's online catalogue into a clean business dataset and uses it to make purchasing and product decisions.

Built for **Business Data and Text Mining — Assignment 2 (Web Scraping)** using the sandbox practice site [books.toscrape.com](https://books.toscrape.com/).

> The webpage is not the dataset. The scraper turns it into one.

---

## What it does

1. **Scrapes** every catalogue page (50 pages, 1,000 books) by following the site's "next" button.
2. **Extracts** title, price, star rating, availability and product URL from each book.
3. **Cleans** the data: strips the £ sign, converts word ratings (One–Five) to numbers 1–5, removes whitespace and duplicates.
4. **Saves** the result as `competitor_books.csv`.
5. **Analyses** the data to answer seven business questions.
6. **Solves** the CEO's £100 challenge with an exact optimisation algorithm.
7. **Visualises** how many books fall into each star-rating level.

## Repository contents

| File | Description |
|---|---|
| `bookstore_scraper.ipynb` | The complete scraper and analysis (runs on Google Colab) |
| `competitor_books.csv` | Dataset of 1,000 books produced by the scraper |
| `rating_counts.png` | Bar chart of books per star rating |
| `README.md` | Project overview, results and recommendation |

## How to run

1. Open `bookstore_scraper.ipynb` in [Google Colab](https://colab.research.google.com/) (File → Upload notebook).
2. Click **Runtime → Run all**.
3. The notebook scrapes the site, prints every answer, and downloads the CSV and chart.

To scrape fewer pages, change `MAX_PAGES = None` to `MAX_PAGES = 3` (or 5) in the setup cell.

**Libraries:** `requests`, `beautifulsoup4`, `pandas`, `numpy`, `matplotlib`

---

## Mission 1 — Reconnaissance

Each book is inside an `<article class="product_pod">` card.

| Data Needed | HTML Tag / Class | Notes |
|---|---|---|
| Book Title | `<h3>` → `<a title="...">` | Link text is truncated, so the full title comes from the `title` attribute |
| Price | `<p class="price_color">` | Text like `£51.77`; the £ sign is removed and the value converted to a float |
| Rating | `<p class="star-rating Three">` | The rating is the second CSS class word, mapped to 1–5 |
| Availability | `<p class="instock availability">` | Text "In stock" with surrounding whitespace, which is stripped |
| Product URL | `<h3>` → `<a href="...">` | Relative link converted to a full URL with `urljoin` |
| Next page | `<li class="next">` → `<a href>` | The loop follows this until it no longer exists |

## How the scraper works

All logic lives in small reusable functions, so the same code handles one page, three pages or the whole catalogue:

- `get_soup(url)` downloads a page and parses it with BeautifulSoup.
- `parse_book(card, page_url)` pulls the five fields out of one product card.
- `scrape_page(url)` parses every card on a page and finds the "next" link.
- `scrape_catalogue(start_url, max_pages)` loops through pages until there is no next page (or the page limit is reached), with a short polite delay between requests.

## Dataset

**1,000 books · 5 columns · 0 missing values**

| Title | Price | Rating | Availability | URL |
|---|---|---|---|---|
| A Light in the Attic | 51.77 | 3 | In stock | https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html |
| Tipping the Velvet | 53.74 | 1 | In stock | https://books.toscrape.com/catalogue/tipping-the-velvet_999/index.html |
| Soumission | 50.10 | 1 | In stock | https://books.toscrape.com/catalogue/soumission_998/index.html |

---

## Mission 4 — Intelligence Analysis

| # | Question | Answer |
|---|---|---|
| 1 | Most expensive book | **The Perfect Play (Play by Play #1)** at £59.99 (3 stars) |
| 2 | Cheapest book | **An Abundance of Katherines** at £10.00 (5 stars) |
| 3 | Average book price | **£35.07** (median £35.98) |
| 4 | Books with a 5-star rating | **196** (19.6% of the catalogue) |
| 5 | Percentage in stock | **100%** (all 1,000 books) |
| 6 | Top 5 highest-rated books | See below |
| 7 | Cheapest 5-star book | **An Abundance of Katherines** at £10.00 |

**Q6 — Top 5 highest-rated books.** 196 books share the top rating of 5 stars, so ties are broken by lower price (best value for customers):

| Rank | Title | Price | Rating |
|---|---|---|---|
| 1 | An Abundance of Katherines | £10.00 | 5 |
| 2 | Greek Mythic History | £10.23 | 5 |
| 3 | The Power Greens Cookbook: 140 Delicious Superfood Recipes | £11.05 | 5 |
| 4 | Dear Mr. Knightley | £11.21 | 5 |
| 5 | The Darkest Corners | £11.33 | 5 |

**Note on Q5:** the catalogue listing pages only show "In stock" or not, so every scraped book is available. Exact stock quantities are only shown on individual product pages.

### Supporting evidence

| Star rating | Books | Average price | Cheapest | Most expensive |
|---|---|---|---|---|
| 1 | 226 | £34.56 | £10.40 | £59.64 |
| 2 | 196 | £34.81 | £10.02 | £59.95 |
| 3 | 203 | £34.69 | £10.16 | £59.99 |
| 4 | 179 | £36.09 | £10.01 | £59.45 |
| 5 | 196 | £35.37 | £10.00 | £59.92 |

- **Price does not follow quality.** 5-star books average £35.37, almost the same as 1-star books at £34.56. The correlation between price and rating is just 0.03.
- **Prices are spread evenly** from £10 to £60, with roughly 200 books in each £10 band.
- **42 five-star books cost under £20**, and 90 five-star books are priced below the catalogue average.
- **1-star books are the largest group** (226 books, 22.6% of the catalogue).

![Books by star rating](rating_counts.png)

---

## Final Challenge — The CEO Has £100

**Goal:** buy books that maximise total star rating without spending more than £100.

This is a **0/1 knapsack problem** (each book can be bought once, under a fixed budget). The notebook solves it exactly with dynamic programming, working in pence to avoid rounding errors. If two baskets tie on stars, the cheaper one is chosen.

| # | Title | Price | Rating |
|---|---|---|---|
| 1 | An Abundance of Katherines | £10.00 | 5 |
| 2 | The Origin of Species | £10.01 | 4 |
| 3 | Greek Mythic History | £10.23 | 5 |
| 4 | The Power Greens Cookbook: 140 Delicious Superfood Recipes | £11.05 | 5 |
| 5 | Dear Mr. Knightley | £11.21 | 5 |
| 6 | The Darkest Corners | £11.33 | 5 |
| 7 | Naturally Lean: 125 Nourishing Gluten-Free, Plant-Based Recipes | £11.38 | 5 |
| 8 | Fruits Basket, Vol. 2 (Fruits Basket #2) | £11.64 | 5 |
| 9 | Old School (Diary of a Wimpy Kid #10) | £11.83 | 5 |
| | **Total** | **£98.68** | **44 stars** |

**Why 44 is the best possible score:**

- No basket can hold 10 books, because the 10 cheapest books in the catalogue cost more than £100 together.
- A perfect 45 (nine 5-star books) is also impossible: the nine cheapest 5-star books cost £100.56, which is 56p over budget.
- So the best option is eight cheap 5-star books plus the cheapest remaining book that fits, *The Origin of Species* (4 stars, £10.01).

For comparison, simply buying the cheapest books first also gets 9 books but scores only **30 stars**. This shows why rating must be part of the decision, not just price.

---

## Business Recommendation

If launching a competing bookstore, prioritise the competitor's **5-star books priced under £20**: 42 titles such as *An Abundance of Katherines* (£10.00) and *Greek Mythic History* (£10.23). The data shows the competitor does not price by quality. Its 5-star books average £35.37, almost the same as its 1-star books at £34.56, so we can win customers by offering top-rated books at clearly lower prices. We should also avoid stocking heavily from the 1-star segment, which is the competitor's largest group (226 books), and since all 1,000 competitor books are in stock, we should compete on curation and value rather than availability.

---

## Bonus missions completed

- [x] **Bonus 1:** scraped more than 5 pages
- [x] **Bonus 2:** product page URL added to the dataset
- [x] **Bonus 3:** books counted per star rating (1★ 226 · 2★ 196 · 3★ 203 · 4★ 179 · 5★ 196)
- [x] **Bonus 4:** bar chart of rating counts
- [x] **Boss Level:** all 50 catalogue pages scraped with the same reusable logic

## Responsible scraping

This project only scrapes [books.toscrape.com](https://books.toscrape.com/), a sandbox website built for web-scraping practice. The scraper sends a User-Agent header, uses timeouts, and pauses between requests to avoid overloading the server.
