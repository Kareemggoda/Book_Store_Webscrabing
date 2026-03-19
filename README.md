# 📚 Book Store Scraper & Visualizer

A Python project that scrapes all **1,000 books** from [books.toscrape.com](https://books.toscrape.com/index.html), stores them in a structured format, and visualizes the data using **Matplotlib** and **Seaborn**.

---
## 📊 Key Insights

- Most books fall in the 3-star and 4-star categories
- Price distribution is right-skewed
- No strong correlation between rating and price
---  
## 🌐 Target Website

**URL:** [https://books.toscrape.com/index.html](https://books.toscrape.com/index.html)

A sandbox website built specifically for web scraping practice. It contains 1,000 books across 50 pages and 51 categories.

> ⚠️ Prices and ratings are randomly assigned and have no real meaning.

---

## 🛠️ Libraries Used

| Library | Purpose |
|---|---|
| `requests` | Send HTTP GET requests to the website |
| `beautifulsoup4` | Parse and navigate HTML content |
| `pandas` | Store and clean scraped data |
| `matplotlib` | Create charts and visualizations |
| `seaborn` | Style and enhance charts |
| `re` | Clean price strings with regex |
| `time` | Add delay between page requests |

### Install all dependencies

```bash
pip install requests beautifulsoup4 pandas matplotlib seaborn
```

---

## 📦 Data Collected

For every book across all 50 pages:

| Field | Example |
|---|---|
| `title` | A Light in the Attic |
| `price` | £51.77 |
| `rating` | Three |
| `page` | 1 |

---

## 🚀 How to Run

### Step 1 — Import libraries

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import seaborn as sns
import re
```

### Step 2 — Test a single page

```python
response = requests.get("https://books.toscrape.com/index.html")
soup = BeautifulSoup(response.content, 'html.parser')

book_store = soup.find_all('article', {'class': 'product_pod'})

for book in book_store:
    title  = book.select_one("h3 a")["title"]
    price  = book.select_one("p.price_color").text
    rating = book.select_one("p.star-rating")["class"][1]
    print(f'"{title} - {price} - {rating}"')
```

### Step 3 — Scrape all 50 pages (1,000 books)

```python
import time

all_books = []

for page in range(1, 51):
    try:
        url = f"http://books.toscrape.com/catalogue/page-{page}.html"
        response = requests.get(url, timeout=10)
        soup = BeautifulSoup(response.text, "html.parser")

        book_store = soup.select("li.col-xs-6.col-sm-4.col-md-3.col-lg-3")

        for book in book_store:
            title  = book.select_one("h3 a")["title"]
            price  = book.select_one("p.price_color").text
            rating = book.select_one("p.star-rating")["class"][1]

            all_books.append({
                "title":  title,
                "price":  price,
                "rating": rating,
                "page":   page
            })

        time.sleep(1)

    except requests.exceptions.RequestException as e:
        print(f"Page {page} failed: {e}")

print(len(all_books))  # → 1000
```

### Step 4 — Save to CSV

```python
pd.DataFrame(all_books).to_csv("books.csv", index=False)
```

### Step 5 — Clean and prepare for visualization

```python
df = pd.DataFrame(all_books)

# Remove £ symbol and any encoding artifacts (Â£), convert to float
df["price"] = df["price"].apply(lambda x: float(re.sub(r"[^\d.]", "", str(x))))

# Map star rating words to numbers
rating_map = {"One": 1, "Two": 2, "Three": 3, "Four": 4, "Five": 5}
df["rating_num"] = df["rating"].map(rating_map)
```

---

## 📊 Visualizations

### Chart 1 — Rating Distribution

```python
fig, ax = plt.subplots(figsize=(8, 5))
rating_counts = df["rating"].value_counts().reindex(["One","Two","Three","Four","Five"])
colors = ["#e74c3c", "#e67e22", "#f1c40f", "#2ecc71", "#27ae60"]

bars = ax.bar(rating_counts.index, rating_counts.values, color=colors, edgecolor="white")
for bar, val in zip(bars, rating_counts.values):
    ax.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 3,
            str(val), ha="center", va="bottom", fontweight="bold", fontsize=11)

ax.set_title("Number of Books per Star Rating", fontsize=14, fontweight="bold", pad=15)
ax.set_xlabel("Rating", fontsize=12)
ax.set_ylabel("Number of Books", fontsize=12)
sns.despine()
plt.tight_layout()
plt.show()
```

### Chart 2 — Price Distribution (Histogram + Box Plot)

```python
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

axes[0].hist(df["price"], bins=30, color="#3498db", edgecolor="white", linewidth=0.8)
axes[0].axvline(df["price"].mean(), color="red", linestyle="--", linewidth=1.5,
                label=f'Mean: £{df["price"].mean():.2f}')
axes[0].set_title("Price Distribution (Histogram)", fontsize=13, fontweight="bold")
axes[0].set_xlabel("Price (£)", fontsize=11)
axes[0].set_ylabel("Number of Books", fontsize=11)
axes[0].legend()
sns.despine(ax=axes[0])

axes[1].boxplot(df["price"], vert=True, patch_artist=True,
                boxprops=dict(facecolor="#3498db", color="navy"),
                medianprops=dict(color="red", linewidth=2),
                whiskerprops=dict(color="navy"),
                capprops=dict(color="navy"),
                flierprops=dict(marker="o", color="gray", alpha=0.5))
axes[1].set_title("Price Distribution (Box Plot)", fontsize=13, fontweight="bold")
axes[1].set_ylabel("Price (£)", fontsize=11)
axes[1].set_xticks([])
sns.despine(ax=axes[1])

plt.tight_layout()
plt.show()
print(f"Min: £{df['price'].min():.2f} | Max: £{df['price'].max():.2f} | Avg: £{df['price'].mean():.2f}")
```

### Chart 3 — Full Dashboard (2×3 grid, saved as PNG)

```python
fig, axes = plt.subplots(2, 3, figsize=(18, 11))
fig.suptitle("📚 Books to Scrape — Data Dashboard", fontsize=18, fontweight="bold", y=1.01)

colors = ["#e74c3c", "#e67e22", "#f1c40f", "#2ecc71", "#27ae60"]

# Rating Distribution
rating_counts = df["rating"].value_counts().reindex(["One","Two","Three","Four","Five"])
axes[0,0].bar(rating_counts.index, rating_counts.values, color=colors, edgecolor="white")
axes[0,0].set_title("Rating Distribution", fontweight="bold")
axes[0,0].set_xlabel("Rating")
axes[0,0].set_ylabel("Count")

# Price Histogram
axes[0,1].hist(df["price"], bins=30, color="#3498db", edgecolor="white")
axes[0,1].axvline(df["price"].mean(), color="red", linestyle="--",
                  label=f'Mean: £{df["price"].mean():.2f}')
axes[0,1].set_title("Price Distribution", fontweight="bold")
axes[0,1].set_xlabel("Price (£)")
axes[0,1].set_ylabel("Count")
axes[0,1].legend(fontsize=9)

# Avg Price per Rating
avg_price = df.groupby("rating")["price"].mean().reindex(["One","Two","Three","Four","Five"])
axes[0,2].bar(avg_price.index, avg_price.values, color=colors, edgecolor="white")
axes[0,2].set_title("Avg Price per Rating", fontweight="bold")
axes[0,2].set_xlabel("Rating")
axes[0,2].set_ylabel("Avg Price (£)")

# Price Box Plot
axes[1,0].boxplot(df["price"], patch_artist=True,
                  boxprops=dict(facecolor="#3498db"),
                  medianprops=dict(color="red", linewidth=2))
axes[1,0].set_title("Price Box Plot", fontweight="bold")
axes[1,0].set_ylabel("Price (£)")
axes[1,0].set_xticks([])

# Top 10 Most Expensive
top10 = df.nlargest(10, "price")
short = [t[:25]+"..." if len(t) > 25 else t for t in top10["title"]]
axes[1,1].barh(short[::-1], top10["price"].values[::-1], color="#8e44ad", edgecolor="white")
axes[1,1].set_title("Top 10 Most Expensive", fontweight="bold")
axes[1,1].set_xlabel("Price (£)")

# Top 10 Cheapest
bot10 = df.nsmallest(10, "price")
short2 = [t[:25]+"..." if len(t) > 25 else t for t in bot10["title"]]
axes[1,2].barh(short2[::-1], bot10["price"].values[::-1], color="#16a085", edgecolor="white")
axes[1,2].set_title("Top 10 Cheapest", fontweight="bold")
axes[1,2].set_xlabel("Price (£)")

plt.tight_layout()
plt.savefig("books_dashboard.png", dpi=150, bbox_inches="tight")
plt.show()
```

---

## 📁 Project Structure

```
Book_Store_Scraping/
│
├── Book_Store_Scrabing.ipynb   # Main Jupyter notebook
├── books.csv                   # Exported CSV (generated after running)
├── books_dashboard.png         # Saved dashboard image (generated after running)
└── README.md                   # This file
```

---

## ⬆️ How to Upload to GitHub

### 1. Create a new repository
Go to [https://github.com/new](https://github.com/new), name your repo (e.g. `book-store-scraper`), then click **Create repository**.

### 2. Push from your terminal

```bash
git init
git add .
git commit -m "Initial commit: Book Store scraper and visualizer"
git remote add origin https://github.com/YOUR_USERNAME/book-store-scraper.git
git branch -M main
git push -u origin main
```

> Replace `YOUR_USERNAME` with your actual GitHub username.

---

## 📝 Notes

- **50 pages × 20 books = 1,000 books** total
- Star ratings are stored as words: `One`, `Two`, `Three`, `Four`, `Five`
- Price values may contain encoding artifacts (`Â£`) — the `re.sub` cleaner handles this
- `time.sleep(1)` between requests prevents overloading the server

---

## 📄 License

Educational use only. The target website is a sandbox built specifically for scraping practice.
