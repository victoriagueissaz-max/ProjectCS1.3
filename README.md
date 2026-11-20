# Wine & Dine 🍷

**Problem:** people want to organize a nice dinner but do not know which wine goes with which food.

**Solution:** Wine & Dine has three ways to help:

1. **I know what I'm cooking** – type your meal (the app suggests dishes while you type) and get the
   **3 best real wines** with a match bar from 1 to 100, where they come from, how they taste and
   why they match. You can also check a bottle you already have.
2. **I already have a wine** – type the name, year and producer of your wine, tell us what you
   cannot eat and what you feel like, and get a ranked list of up to 50 dishes. Click a dish to see
   why it matches, the recipe and the grocery list, and save it.
3. **Effortless evening** – answer four questions (occasion, guests, budget, food wishes) and get one
   dish with the best wine, the recipe and the number of bottles. Don't like it? *Generate again.*

Everything you save appears on **My profile**.

Group project for *Fundamentals and Methods of Computer Science*, HSG, autumn 2026.

---

## How to run the app

**On your laptop** (in the VS Code terminal, inside this folder):
1. Install the libraries (only once):
   ```
   pip3 install -r requirements.txt
   ```
2. Start the app:
   ```
   streamlit run streamlit_app.py
   ```
   The app opens in your browser at http://localhost:8501.
   The first start takes about 30 seconds, because the app builds its database
   (`wine_dine.db`) from the files in `data` (it needs internet for the dish photos).

**Online** (Streamlit Community Cloud): share.streamlit.io → Create app → choose this GitHub
repository, branch `main`, main file `streamlit_app.py` → Deploy.

---

## Files

| File | What it is |
|------|------------|
| `streamlit_app.py` | The whole web application (Streamlit) – it also builds the database the first time it starts |
| `wine_dine.db` | The SQLite database the app reads from (built automatically, not uploaded to GitHub) |
| `.streamlit/config.toml` | Colours and fonts of the app (the "theme") |
| `data/winemag-data-130k-v2.csv.zip` | Wine reviews (source: Kaggle), zipped so it fits on GitHub |
| `data/wine_food_pairings.csv` | Wine and food pairing scores (source: Kaggle) |
| `data/dishes.csv` | Our 60 dishes with allergens, calories, protein, recipe and grocery list (our own table) |
| `data/pairings.csv` | The 3 best grape varieties per dish with a score (1–100) and why (our own table) |
| `data/wine_styles.csv` | Colour, body and taste of the 26 grape varieties (our own table) |
| `AI_USE_LOG.md` | Where and how we used AI |
| `requirements.txt` | Python libraries the app needs |

---

## How the app meets the project requirements

| # | Requirement | How Wine & Dine does it |
|---|-------------|-------------------------|
| 1 | Clear problem | "Which wine goes with my dinner – and which dinner goes with my wine?" |
| 2 | Data from an API / database | **Database:** all data comes from the SQLite database `wine_dine.db` (≈70,000 real wines, 60 dishes, 180 pairings), read with SQL (`SELECT`, `JOIN`, `GROUP BY`, `LIKE`) and written with `INSERT` / `DELETE` (saved pairings). **API:** the app gets the dish photos from the **Wikipedia API** when it builds its database |
| 3 | Data visualization | The **match bars** (1–100, from dark green to orange), "Is a more expensive wine really better?" (% of top wines per price range) and "Where does the best wine come from?" (% of top wines per country) |
| 4 | User interaction | Search bar with suggestions, wine search by name, year and producer, allergy and preference buttons, pop-up window with recipe, four-question planner with "Generate again", saving and removing pairings, a nutrition profile (diet, allergies, foods to avoid, goals) that filters every page |
| 5 | Machine learning | **Placeholder** `predict_match_score()` in `streamlit_app.py` (simple rules for now) – the model is built by [name of ML teammate] |
| 6 | Documented code | Every function has a docstring and every block has comments |

---

## Database structure

```
dishes (dish_id, dish, course, diet, cooking_time, calories, protein, allergens,
        occasion, wiki_page, ingredients, recipe, image_url)
   │ 1
   │ n
pairings (dish_id, wine_type, score, why)
   │ n
   │            (wine_type = variety)
   │ 1          │
wine_styles (wine_type, color, body, taste)
                │ 1
                │ n
wines (title, variety, country, province, winery, points, price, description)

favorites (saved_id, dish_id, wine_type, wine_name, price, score, saved_on)  – filled by the users
profile (profile_id, diet, allergies, avoid, goals)                          – the user's nutrition requirements
kaggle_pairings (wine_type, food_item, ...)                                  – cleaned Kaggle data, for the ML model
```

---

## About the data

- **Wine reviews** – [Kaggle: zynicide/wine-reviews](https://www.kaggle.com/datasets/zynicide/wine-reviews).
  About 130,000 reviews from *Wine Enthusiast* magazine with price (in US dollars, converted to Swiss francs at 1 USD = 0.80 CHF), rating
  (80–100 points), region, producer and a tasting note. We keep only wines with a price and of the
  26 grape varieties that our app recommends (≈70,000 wines).
- **Wine and food pairings** – [Kaggle: wafaaelhusseini/wine-and-food-pairing-dataset](https://www.kaggle.com/datasets/wafaaelhusseini/wine-and-food-pairing-dataset).
  When we analysed this dataset we found that it is **mostly artificial**: many rows are random
  ("Heuristic pairing assessment.") or deliberately wrong examples, and averaged scores are
  almost the same for every wine. That is why we **do not use it for the recommendations**.
  We keep the ≈10,600 rows that give a real reason in the table `kaggle_pairings` for the
  machine learning part.
- **Dish photos** – from the **Wikipedia API** (Wikimedia Commons, free licences). The app shows
  "Photos: Wikipedia" as the source.
- **Our own tables** (`dishes.csv`, `pairings.csv`, `wine_styles.csv`) – drafted with AI based on
  common wine pairing rules and checked by our team (see `AI_USE_LOG.md`, entries 4 and 6).
  Calories and protein are rough estimates per portion.

---

## How the code works (in simple words)

Everything is in **one file, `streamlit_app.py`**. It runs from top to bottom **every time the
user clicks or changes something** (this is how Streamlit works). In order:

1. **Imports and page setup** – like in the week 4 demo.
   **Where are our files?** – `APP_FOLDER` is the folder of `streamlit_app.py`; all other paths
   are built from it, so the app finds its data no matter from where it is started.
   **Building the database (only the first time)** – if `wine_dine.db` does not exist yet,
   `build_database()` reads the five CSV files with `pd.read_csv`, cleans them (drops wines
   without price, keeps only our grape varieties, removes unreliable Kaggle rows), asks the
   Wikipedia API for a photo of every dish (with `requests.get`, 30 dishes per request) and saves
   each table into `wine_dine.db` with `to_sql`. It also creates an empty `favorites` table.
   Then the app connects to the database – like in the week 7 demo.
2. **Data for every page** – the small tables (dishes, pairings, wine styles) are read once.
   Lists and dictionaries hold the options (allergies, budgets, occasions) and famous wine names
   (e.g. `'barolo': 'Nebbiolo'`).
3. **Style** – a few lines of CSS for the golden labels and the match bar
   (colours and fonts are in `.streamlit/config.toml`).
4. **Functions** – each one does one small job:
   - `predict_match_score` – **ML placeholder**: how well a wine fits a dish (1–100)
   - `get_match_color` / `get_match_label` / `show_match_bar` – the coloured bar
   - `get_top_wines_for_dish` – the score of all 26 wine types, sorted
   - `find_best_bottle` – the best real wine of one type within the budget (SQL);
     sweet wines only for desserts
   - `search_wines` / `find_wine_type_in_text` – recognise the wine the user typed
   - `find_closest_dish` – the dish that fits a free text best (counts matching words)
   - `fits_allergies` / `fits_preferences` / `filter_dishes` – the filters of page 2 and 3
   - `save_favorite` – adds a row to the `favorites` table (`INSERT`)
   - `show_recipe`, `show_wine_details`, `ask_for_wine`, `open_dish_window` – pieces of the pages
   - `get_share_of_top_wines` / `get_top_wines_by_country` – the data for the two charts
5. **Sidebar** – the name of the app and the page selection.
6. **Pages** – an `if / elif` block shows the page the user selected.
7. **Close the database.**

**Why `?` in the SQL queries?** Instead of putting user input directly into the SQL text
(like the f-strings in the week 7 demo), we write `?` and pass the values with `params=[...]`.
pandas then inserts them safely. This prevents *SQL injection*, which the week 7 demo mentions
as an important topic.

**Three Streamlit features that go a bit beyond the course** (all explained in the code comments):
- `st.session_state` – a dictionary that Streamlit keeps between two clicks. The *Effortless
  evening* page needs it to remember the planned dinner when the user clicks "Generate again".
- `@st.dialog` – turns a function into a pop-up window (the recipe window on page 2).
- `st.markdown(..., unsafe_allow_html=True)` – lets us write a bit of HTML/CSS for the match bar
  and the photos.

---

## For the ML teammate

The app calls `predict_match_score(wine_type, dish)` in `streamlit_app.py` everywhere a match score is shown
(the bars on all pages and the ranking on page 2). `dish` is one row of the `dishes` table.
At the moment the function uses simple rules (our pairing table + similar wine colour and body).
When your model is ready, replace the body of this function so it returns a **whole number from
1 to 100**. Everything else in the app will then use your model automatically.
Training data: the table `kaggle_pairings` (cleaned), our `pairings` and `wine_styles` tables.

---

## Use of AI

See [`AI_USE_LOG.md`](AI_USE_LOG.md). Every AI-generated part of the code is marked with a
comment that points to the matching entry of the log.
