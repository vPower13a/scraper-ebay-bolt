from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
import requests
from bs4 import BeautifulSoup

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/scrape")
def scrape(query: str):
    url = f"https://www.ebay.fr/sch/i.html?_nkw={query}"
    headers = {"User-Agent": "Mozilla/5.0"}
    r = requests.get(url, headers=headers)
    soup = BeautifulSoup(r.text, "html.parser")

    results = []
    for item in soup.select(".s-item")[:10]:
        title = item.select_one(".s-item__title")
        price = item.select_one(".s-item__price")
        link = item.select_one(".s-item__link")
        if title and price and link:
            results.append({
                "title": title.text,
                "price": price.text,
                "link": link["href"]
            })
    return results
