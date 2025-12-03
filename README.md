# Manchester Airbnb Recommendation System

An end-to-end **Airbnb recommendation system** for Manchester that combines:

- **K-Means clustering** of Airbnb listings  
- **K-Means clustering** of Manchester neighbourhoods by venue categories  
- **Geospatial enrichment** using ArcGIS and the Foursquare API  
- **Interactive map visualisation** using Folium  

Given a user’s preferences (price, room type, minimum nights, reviews, availability, neighbourhood amenities), the system recommends a shortlist of suitable Airbnb listings in Manchester.

---

## 1. Project Overview

Greater Manchester has **4,900+ Airbnb listings** across **41 neighbourhoods** and **10 boroughs**. Manually selecting an ideal place is difficult, especially when users care about both:

- Listing attributes (price, room type, reviews, availability), and  
- Neighbourhood characteristics (e.g. Chinese restaurants, pubs, supermarkets). 

This project builds a **two-level clustering system**:

1. **Listing Clusters** – K-Means on Airbnb listing features  
2. **Neighbourhood Clusters** – K-Means on venue-category distributions from Foursquare  

The recommendation engine then cross-filters listings and neighbourhoods to produce a **condensed, personalised shortlist** (e.g. ~50 listings out of 3,000+). 

---

## 2. Data Sources

- **Airbnb Listings for Greater Manchester**  
  - Source: [Inside Airbnb](http://insideairbnb.com/get-the-data.html)  
  - File: `data/raw/listings_greater_manchester.csv`  
  - Includes: `id`, `neighbourhood_group`, `neighbourhood`, `latitude`, `longitude`, `room_type`, `price`, `minimum_nights`, `number_of_reviews`, `availability_365`, etc.  

- **Geographical Coordinates of Manchester Neighbourhoods**  
  - Obtained via ArcGIS geocoding API and `geocoder` in Python  
  - Used to map neighbourhoods and request venue data from Foursquare. 

- **Venue Categories per Neighbourhood**  
  - Obtained through the Foursquare API (e.g. restaurants, supermarkets, pubs, etc.)  
  - Used to build a venue-category frequency matrix for neighbourhood clustering.   

> **Note:** This repository includes code hooks for the APIs, but you need to provide your own API keys and raw data (not included here for licensing reasons).

---

## 3. Architecture & Pipeline

### 3.1 High-Level Steps

1. **Load & Clean Data**
   - Load Greater Manchester listings
   - Filter to Manchester-only neighbourhoods
   - Handle missing values, convert data types, select relevant features

2. **Descriptive Analytics**
   - Summary statistics (price, room types, reviews, availability)  
   - Distribution of listings across neighbourhoods and boroughs 

3. **K-Means Clustering – Listings**
   - Features: `room_type`, `price`, `minimum_nights`, `number_of_reviews`, `availability_365`  
   - Preprocessing: scaling & one-hot encoding  
   - Output: 10 listing clusters (Cluster 0–9)   

4. **K-Means Clustering – Neighbourhoods**
   - Use ArcGIS to get lat/long of 32 Manchester neighbourhoods  
   - Use Foursquare to pull venues & categories for each neighbourhood  
   - Build venue-category frequency matrix  
   - Output: 5 neighbourhood clusters:
     - Cluster 1 – Chinese restaurants, pubs, convenience stores  
     - Cluster 2 – Variety of restaurants, grocery stores, bars  
     - Cluster 3 – Men’s store, warehouse, concert hall  
     - Cluster 4 – Food truck, construction, landscaping  
     - Cluster 5 – Chinese & fast-food restaurants, supermarkets/markets 

5. **Recommendation Engine**
   - Take user preferences (e.g. `room_type=private room`, `price=55`, `min_nights=2`, `reviews≈100`, `availability≈340`, neighbourhood type with Chinese & fast food restaurants).  
   - Predict the most suitable **listing cluster**  
   - Restrict to neighbourhoods in the desired **neighbourhood cluster** (e.g. Cluster 5: Higher Blackley, Gorton South, Gorton North, Harpurhey, Sharston) 
   - Return final recommended listings.

6. **Visualisation**
   - Folium maps for:
     - All listings and their clusters
     - Neighbourhood clusters
     - Final recommended listings on the Manchester map 

---

## 4. Repository Structure

```text
manchester-airbnb-recommender/
├── README.md
├── requirements.txt
├── .gitignore
├── config.example.yml
├── data/
│   ├── raw/
│   └── external/
├── models/
├── notebooks/
│   └── 01_manchester_airbnb_exploration.ipynb
├── src/
│   ├── config.py
│   ├── data.py
│   ├── features.py
│   ├── clustering.py
│   ├── recommend.py
│   └── viz.py
└── scripts/
    ├── run_full_pipeline.py
    └── demo_recommendation.py

```

---

## 5. Setup & Installation
# clone repo
git clone https://github.com/<your-username>/manchester-airbnb-recommender.git
cd manchester-airbnb-recommender

# create and activate virtual environment (optional but recommended)
python -m venv .venv
source .venv/bin/activate     # on Windows: .venv\Scripts\activate

# install dependencies
pip install -r requirements.txt

---

## 6. Configuration

Copy the example config and edit:

```bash
cp config.example.yml config.yml

```
config.yml:

```yaml
data:
  listings_csv: "data/raw/listings_greater_manchester.csv"
  neighbourhoods_csv: "data/external/manchester_neighbourhoods.csv"

foursquare:
  api_key: "YOUR_FSQ_API_KEY"

arcgis:
  api_key: "YOUR_ARCGIS_API_KEY"

clustering:
  listings_n_clusters: 10
  neighbourhoods_n_clusters: 5

```

---

## 7. Running the Pipeline
### 7.1 Train Clustering Models
python scripts/run_full_pipeline.py


This will:

Load and preprocess Airbnb listings

Retrieve or load neighbourhood & venue data

Train K-Means for listings and neighbourhoods

Save trained models to models/

Optionally generate Folium maps in outputs/

### 7.2 Demo Recommendation
python scripts/demo_recommendation.py


This script runs a demo scenario similar to the report:

Room type: private room

Price: £55

Minimum nights: 2

Number of reviews: 100

Availability: 340 days

Neighbourhood: cluster with Chinese & fast-food restaurants, supermarkets/markets (Cluster 5) 

It prints a small table of recommended listings and can save a map of recommended listings.

## 8. Extending the Project

You can extend this project by:

Adding more sophisticated recommendation logic (e.g. distance to city centre, stadiums, universities)

Integrating a simple web app (Flask/FastAPI) to expose an API

Deployed as a containerised service on Azure / AWS

## 9. License

MIT (or any other you prefer).

## 10. Acknowledgements

Inside Airbnb

for the listing data

ArcGIS and Foursquare for geocoding and venue APIs

Original project report and methodology documented in “Airbnb Recommendation System for Manchester Neighbourhoods Using K-Means Clustering and Foursquare API in Python”.