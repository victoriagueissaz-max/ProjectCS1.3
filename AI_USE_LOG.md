# AI use log – Wine & Dine

This log documents **exactly where and how generative AI was used** in our project,
as required by the HSG rules on AI use and the group project requirements.
The source code also points to these entries with comments like
`# AI-generated (Claude) - see AI_USE_LOG.md entry 7`.

**AI tool used:** Claude (Anthropic), model "Claude Opus 5.5", used through the
Claude Code desktop app.
**Used by:** [name of team member] – responsible for the web application.

> **To do for the team:** fill in the fields marked with `[...]`, especially
> "What we checked / changed". The tutors want to see that we understood and
> reviewed the AI output, not only that we used it.

---

## Overview

| # | Date | What | Files | Type of use |
|---|------|------|-------|-------------|
| 1 | 16.11.2026 | Understanding the requirements and planning the app | – | AI as an aid (no code) |
| 2 | 16.11.2026 | Script that builds the database | `create_database.py` | AI-generated code |
| 3 | 16.11.2026 | The Streamlit web app | `app.py` | AI-generated code |
| 4 | 16.11.2026 | Dish list and food–wine pairing table | `data/dishes.csv`, `data/pairings.csv` | AI-generated data content |
| 5 | 16.11.2026 | Checking the quality of the Kaggle pairing dataset | – (result described in README) | AI as an aid (data analysis) |
| 6 | 17.11.2026 | More dishes, scores, wine styles, dish photos (Wikipedia API), design theme | `data/dishes.csv`, `data/pairings.csv`, `data/wine_styles.csv`, `create_database.py`, `.streamlit/config.toml` | AI-generated data content and code |
| 7 | 17.11.2026 | Redesign of the web app after our own design brief | `app.py` | AI-generated code |
| 8 | 17.11.2026 | Preparing the app to run online (Streamlit Community Cloud) | `app.py`, `create_database.py` | AI-generated code |
| 9 | 17.11.2026 | Bug fix: app and database script merged into one file | `streamlit_app.py` (replaces `app.py` and `create_database.py`) | AI-generated code |

---

## Entry 1 – Understanding the requirements and planning (AI as an aid)

- **What we gave the AI:** the project requirements (FCS-BWL-GroupProject.pdf), the grading sheet,
  our website proposal, the lecture slides (weeks 2–5 and 7) and the course notebooks.
- **What we asked:** to explain what the website needs to fulfil the requirements, and to keep
  the code at the level of our course so that we can explain it.
- **What the AI did:** summarised the requirements, explained that the app must be built in
  Python with Streamlit, and proposed a plan (database + Streamlit app + ML placeholder).
- **Our decisions:** we decided which data sources to use, and that the machine learning part
  stays with [name of ML teammate].

## Entry 2 – `create_database.py` (AI-generated code)

- **Prompt (summary):** "Build the website for our Wine & Dine idea with simple code that fits our
  course (slides and notebooks provided). Use our data sources (wine reviews from Kaggle, wine and
  food pairing dataset from Kaggle) and put them into a database."
- **What the AI generated:** the complete script. It reads the CSV files with pandas, removes
  wines without a price and unused grape varieties, removes the unreliable rows of the Kaggle
  pairing data, and writes everything into the SQLite database `wine_dine.db`
  (in the style of the week 7 demo `streamlit_db_demo.py`).
- **What we checked / changed:** [...]

## Entry 3 – `app.py` (AI-generated code)

- **Prompt (summary):** same as entry 2, plus: "use a placeholder for the machine learning part,
  because a teammate is responsible for it."
- **What the AI generated:** the complete Streamlit app with four pages
  (I know what I am cooking / I already have a wine / Plan my entire dinner / My favorites),
  the sidebar filters (budget, diet, cooking time), the "Surprise me" button,
  two charts, the shopping list, and the function `predict_match_score`
  as a placeholder for our ML model.
- **Corrections the AI made while testing (16.11.2026):**
  - Bug fix: dish numbers from pandas (`numpy.int64`) were not understood by SQLite,
    so no wines were found. Fixed with `int(dish_id)` in `get_pairings_for_dish`.
  - The first charts (price vs. rating as dots, average rating per country) were hard to read,
    because all ratings are between 80 and 100. Replaced by "% of wines rated 90+ points"
    per price range and per country.
- **What we checked / changed:** [...]

## Entry 4 – `data/dishes.csv` and `data/pairings.csv` (AI-generated content)

- **Why:** none of our datasets says which wine goes with which dish in a reliable way
  (see entry 5). We decided together with the AI to create our own small pairing table.
- **What the AI generated:** 32 dishes (course, diet, cooking time, ingredients, short recipe idea)
  and 64 pairings (two grape varieties per dish with a short explanation).
  The pairings follow common wine pairing rules (e.g. tannic red with red meat,
  acidic white with seafood, sweet wine with dessert).
- **What we checked / changed:** [... e.g. "we compared the pairings with wine guides / asked a
  wine shop and changed X"]

## Entry 5 – Quality check of the Kaggle pairing dataset (AI as an aid)

- **What the AI did:** analysed `wine_food_pairings.csv` and found that most rows are random or
  artificial ("Heuristic pairing assessment.", "Deliberately bad pairing example for contrast."),
  and that averaged scores are almost the same (≈3/5) for every wine – for example, Cabernet
  Sauvignon and sweet Ice Wine got the same score with steak.
- **Our decision:** use our own pairing table for the recommendations, and keep the cleaned
  Kaggle data in the database (table `kaggle_pairings`) for the machine learning part.

## Entry 6 – More data, dish photos and design theme (AI-generated content and code)

- **Prompt (summary):** our own design brief for the three pages (see entry 7). To build it, the
  app needed more information per dish (allergens, calories, protein, photo, full recipe and
  grocery list) and a match score from 1 to 100.
- **What the AI generated:**
  - `data/dishes.csv` extended from 32 to **60 dishes**, with allergens (gluten, lactose, eggs,
    nuts, fish, shellfish, soy), estimated calories and protein per portion, "everyday" or
    "festive", a recipe in 4 steps and a grocery list for 4 people. Some dishes were renamed or
    replaced so that Wikipedia has a matching photo (e.g. "Fish tacos" → "Fish and chips").
  - `data/pairings.csv` extended to **3 wines per dish (180 rows)** with a **score from 1 to 100**
    instead of "Perfect match" / "Good match".
  - `data/wine_styles.csv` (new): colour, body and taste of the 26 grape varieties.
  - `create_database.py`: new functions `get_image_urls` and `add_dish_images` that ask the
    **Wikipedia API** for one photo per dish. First version: one request per dish – this was too
    slow (6 minutes) and Wikipedia blocked some requests ("429 too many requests"), so it was
    changed to 30 dishes per request (now a few seconds).
  - `.streamlit/config.toml` (new): colours and fonts inspired by the website of the Antinori
    wine estate (our design reference).
- **What we checked / changed:** [... e.g. "we checked the allergens of every dish", "we compared
  the scores with wine guides and changed X"]

## Entry 7 – Redesign of `app.py` (AI-generated code)

- **Prompt (summary):** our design brief:
  1. *I know what I'm cooking:* a search bar that suggests dishes while typing, photo and name of the
     dish on the left, the 3 best wines on the right with a bar from 1 to 100 (dark green = best
     match), region, year, producer, taste and why it matches; below: "Do you already have a
     wine in mind?" with name or photo and its own bar.
  2. *I already have a wine:* name, year and producer of the wine; allergies and
     preferences; a ranked, scrollable list of up to 50 dishes; clicking a dish opens a pop-up with
     why it matches, recipe, grocery list and a save button at the top right.
  3. *Plan my entire dinner* (renamed "Effortless evening"): occasion, guests, budget and food
     preferences, each with an option to write your own answer; then one proposal (dish + wine)
     with fold-out recipe and wine notes, a save button and "Generate again".
  4. Style: sleek, elegant, luxurious, romantic, not crowded (reference: antinori.it).
- **What the AI generated:** the new `app.py` with four pages (including "My profile" for the saved
  pairings) and new functions for the match bar, wine search, dish search, allergy and preference
  filters, the pop-up window and the evening planner. The ML placeholder `predict_match_score`
  now returns a score from 1 to 100 based on simple rules.
- **Our decision about photos of the wine:** our first brief also asked for a photo upload of the
  bottle. The AI tested a free text-recognition API (OCR.space) on real label photos: it read the
  grape and year well, but could not reliably find the exact bottle. We decided not to use label
  recognition and removed the photo upload - the user types the name, year and producer instead.
- **Corrections the AI made while testing (17.11.2026):**
  - The best-rated bottle was sometimes a sweet dessert wine (e.g. an *Eiswein* with salmon).
    Fixed: sweet wines (Eiswein, Late Harvest, Auslese, ...) are only chosen for desserts.
  - The save button in the pop-up window crashed, because Streamlit runs the pop-up separately
    from the page and the page's database connection cannot be used there. Fixed: `save_favorite`
    opens its own short connection.
  - Option buttons were cut off at the right edge – fixed with `wrap=True`.
- **What we checked / changed:** [...]

## Entry 8 – Running the app online (AI-generated code)

- **Prompt (summary):** "We uploaded the code to GitHub through the website. How do we put the website online?"
- **What the AI generated / changed:**
  - The wine reviews file (50 MB) was zipped to 17 MB, because the GitHub website accepts only
    files up to 25 MB. `create_database.py` now reads the zip file directly with `pd.read_csv`.
  - At the start of `app.py`: if the database file does not exist (which is the case online,
    because the database is not uploaded to GitHub), the app runs `create_database.py` once
    (`import create_database`) before it connects to the database.
- **What we checked / changed:** [...]

## Entry 9 – Bug fix: one file `streamlit_app.py` (AI-generated code)

- **Problem we reported:** the app stopped with an error at line 49 (`import create_database`),
  both in VS Code and online on Streamlit.
- **What the AI found:** two weaknesses in its own code from entry 8:
  1. The file names (`'wine_dine.db'`, `'data/...'`) were relative to the folder the app was
     *started from*. Started from another folder (e.g. the outer project folder in VS Code), the
     app could not find the data files.
  2. The app needed a second file (`create_database.py`) next to it. On GitHub this file was
     missing, so Python could not import it (`ModuleNotFoundError`).
- **What the AI changed:** everything is now in one file, `streamlit_app.py` (the standard name
  Streamlit Cloud looks for). The functions of `create_database.py` were moved into it
  (`build_database()`), all paths are built from the folder of the file itself (`__file__`),
  the database is first written under a temporary name and only renamed when it is complete,
  and the app shows a clear message if a data file is missing. `app.py` and
  `create_database.py` were deleted.
- **How it was tested:** started without a database from a different folder, started a second
  time, started with a missing data file, and clicked through all four pages.
- **What we checked / changed:** [...]

## Entry 10 – Home page, navigation at the top and closed sidebar (AI-generated code)

- **What we asked:** first a home page with a big picture, our logo and brand name, a short
  description and the three options as cards. We did not like the first version, so we asked
  for an elegant style like the website of a famous wine estate, and that the sidebar only opens
  when you click on it and shows other information.
- **What the AI changed in `streamlit_app.py`:**
  - a new **Home** page: a picture over the whole width with "Wine & Dine" on it, a short
    introduction, the three options (I, II, III) with a button each, and a dark band with our
    numbers (70,000 wines, 60 dishes, 26 grape varieties)
  - the pictures are read from the folder `images` (`images/hero.jpg` and `images/logo.png`) by
    the new function `get_own_image()`. As long as they are missing, a dark background is shown.
  - a **navigation bar at the top** of every page instead of the radio buttons in the sidebar.
    The page is stored in `st.session_state['page']`, and the buttons change it with
    `go_to_page()`.
  - the **sidebar is closed** at the start (`initial_sidebar_state='collapsed'`) and now shows
    information about the project, the match score and our data
  - CSS rules for the navigation, the picture and the home page (in the STYLE block)
- **How it was tested:** started the app without and with a test picture, took screenshots on a
  laptop and a phone size, opened the sidebar and clicked the buttons (they open the right page).
- **What we checked / changed:** [...]

## Entry 11 – Small fixes: dish photo and wine caption (AI-generated code)

- **Prompt (summary):** "Remove the line 'We read this as a ...'" and "For the rack of lamb a raw
  image is used - find another image of lamb rack in the Wikipedia API."
- **What the AI changed:** removed the caption in `ask_for_wine()`. For the photo: the main image of
  the Wikipedia article "Rack of lamb" shows raw meat, so the AI added the dictionary
  `CHOSEN_PHOTOS` and the function `get_file_url()`, which asks the Wikipedia API for another photo
  of the same article (a cooked rack of lamb) and uses it instead.
- **What we checked / changed:** [...]

## Entry 12 – "My nutrition requirements" in the profile (AI-generated code)

- **Prompt (summary):** "In the profile section add a 'my nutrition requirements' section where the
  user can say if they are allergic to something or vegan etc., and the whole website filters for
  this profile: some fields are always preselected, and for a vegetarian the meat buttons are not shown."
- **What the AI generated / changed:**
  - A new table `profile` in the database (one row: diet, allergies, avoided foods, goals),
    created with `CREATE TABLE IF NOT EXISTS`, and the functions `load_profile()`, `save_profile()`,
    `fits_diet()`, `get_profile_dishes()`, `get_visible_options()` and `describe_profile()`.
  - A form on the "My profile" page (diet, allergies, other foods, goals) and a short summary of the
    profile in the sidebar.
  - **The diet is always applied:** dishes that do not fit it are never shown, and buttons that make
    no sense are hidden (e.g. "Meat dish" and the fish allergy for a vegetarian).
  - **Allergies, avoided foods and goals** are preselected on the pages that ask for them ("I already
    have a wine", "Effortless evening") and can be switched off for one search. On the pages that do
    not ask for them ("I know what I'm cooking" and the dish of "Effortless evening") they are always
    applied.
- **How it was tested:** with Streamlit's test tool (`AppTest`): all four pages without a profile,
  then with a vegetarian / nut allergy / no mushrooms / low calorie profile (27 of 60 dishes left,
  no meat or fish buttons, values preselected), saving the form, and planning an evening with a
  vegan profile.
- **What we checked / changed:** [...]

## Entry 13 – Profile in three boxes, recommendations with machine learning, fewer texts (AI-generated code)

- **Prompt (summary):** "For the profile I want 3 boxes: my dietary preferences, my recommendations
  (machine learning, based on the wines and recipes the user saves) and my favorites. Remove
  unnecessary comments, the 'e.g. ...' in the typing boxes and 'Press Enter to submit form' - in
  general on the whole website."
- **What the AI generated / changed:**
  - "My profile" now has three boxes: *My dietary preferences* (the form from entry 12),
    *My recommendations* and *My favorites* (the saved pairings).
  - **Machine learning (content-based recommendation, scikit-learn):** `build_wine_taste_model()`
    turns the tasting notes of all wines into numbers with TF-IDF (cached, it runs once);
    `recommend_wines()` and `recommend_dishes()` take the average of what the user saved as their
    "taste" and use cosine similarity to find the most similar wines and dishes that the user has
    not saved yet. Dishes must fit the dietary preferences; wines stay within 1.5 times the most
    expensive saved bottle. `choose_wine_for_dish()` pairs a recommended dish with the user's
    favourite grape if it matches well (score 75 or more).
  - `scikit-learn` was added to `requirements.txt`.
  - Removed the explanatory captions, all "e.g. ..." placeholders and (with CSS) the
    "Press Enter to apply / submit form" hint below the text fields.
- **How it was tested:** with Streamlit's test tool (`AppTest`): all five pages; the profile page
  without favorites and with two saved pairings (mushroom risotto + Pinot Noir, cream of mushroom
  soup + Chardonnay -> recommended: French onion soup, Zürcher Geschnetzeltes, Ossobuco and three
  similar Pinot Noirs); the "Open" button of a recommended dish.
- **What we checked / changed:** [...]

## Entry 14 – Profile boxes next to each other, opened with a click (AI-generated code)

- **Prompt (summary):** "The three boxes should be next to one another, not below one another. On
  the box you only see the title; only when you click on it do the options appear."
- **What the AI changed:** the three boxes are now three buttons in a row (styled as cards with CSS).
  A click opens the content of the box in full width below the row, a second click closes it.
  The open box is remembered in `st.session_state['profile_box']` and gets a golden frame.
- **How it was tested:** with `AppTest`: at the start only the three titles are shown; every box
  opens its own content; saving the dietary preferences keeps the box open; the "Open" button of a
  recommendation still works; a second click closes the box.
- **What we checked / changed:** [...]

## Entry 15 – Bordeaux background, cards on the home page (AI-generated code)

- **What we asked:** a bordeaux red background for the whole website, how to change the fonts,
  remove the links "Find a wine", "Find a dish" and "Plan an evening" from the top bar, and show
  the three options on the home page as white cards with square corners that lift up a little
  when the mouse is over them.
- **What the AI changed:**
  - `.streamlit/config.toml`: dark theme with a bordeaux background (`#5C1A24`), light text and
    a lighter gold; comments explain where the fonts are set
  - `streamlit_app.py`: the colours in the STYLE block now fit the dark background (light text,
    light match bar text, see-through message boxes); the top bar only has "Wine & Dine" (back to
    Home) and "My profile"; the three options are white cards (CSS `transform: translateY` on
    `:hover` lifts them up)
- **How it was tested:** screenshots of the home page (with the mouse over a card), page 1 with a
  dish and the profile page.
- **What we checked / changed:** [...]

## Entry 16 – Cream pages again, dark red home page with a "pouring wine" effect (AI-generated code)

- **What we asked:** only the home page should be dark red like a good red wine (very dark), the
  other pages cream again. We also asked if the page could stay cream after the picture and fill
  up with red while scrolling down, as if wine was poured.
- **What the AI changed:**
  - `.streamlit/config.toml` and the STYLE block: back to the cream colours of entry 10
  - `streamlit_app.py`, HOME PAGE: a CSS block with a fixed layer `wine-layer` (colour `#2A0810`)
    whose height grows from 0 to the whole window while the user scrolls (CSS scroll-driven
    animation: `scroll-timeline` and `animation-timeline`), with a moving wave on top. The texts
    change from dark to light at the same time. Browsers without scroll animations show the
    dark red page directly.
- **How it was tested:** screenshots at six scroll positions, a short screen video, and page 1
  (cream) after clicking a card.
- **What we checked / changed:** [...]

## Entry 17 – A more realistic "pouring wine" effect (AI-generated code)

- **What we asked:** make the pouring more realistic, with bubbles or drops.
- **What the AI changed in `streamlit_app.py` (HOME PAGE):** a thin wine stream that falls from
  the top, trembles and gets shorter while the wine rises (it disappears when the page is full),
  drops falling next to it, rings and small splashes where it hits the wine, two waves moving at
  different speeds, a light line on the surface and bubbles rising in the wine. The bubbles,
  drops and splashes are made in Python with `for` loops from small lists (position, size,
  speed). People who switched off animations on their computer (`prefers-reduced-motion`) get
  the wine without movement.
- **How it was tested:** screenshots at four scroll positions and a short screen video.
- **What we checked / changed:** [...]

## Entry 18 – Back to the simple pouring, with realistic sloshing (AI-generated code)

- **What we asked:** we liked the first, simple version (entry 16) more, but the wine should
  move more realistically, like a glass that is being poured.
- **What the AI changed in `streamlit_app.py` (HOME PAGE):** the stream, drops, splashes and
  bubbles of entry 13 were removed. The wine now sloshes: while scrolling, the surface tilts left
  and right, less and less (keyframes `slosh`, linked to the scrolling), and afterwards it sways a
  little all the time (`sway`), with the wave bobbing up and down. The wine is wider and longer
  than the window, so no corner shows when it tilts.
- **How it was tested:** screenshots at several scroll positions and a short screen video.
- **What we checked / changed:** [...]

## Entry 19 – Irregular waves (AI-generated code)

- **What we asked:** the waves looked too symmetric, and the whole surface moved up and down
  in the same way.
- **What the AI changed in `streamlit_app.py` (HOME PAGE):** the surface is now made of four
  wave layers with irregular shapes (SVG paths) and different lengths. Each layer moves sideways
  at its own speed and direction (`drift` with its own `--drift` distance) and grows and shrinks
  in its own rhythm (`breathe`), so the surface never looks the same. While pouring, a big flat
  wave pushes from side to side (`swell-slosh`) instead of tilting the whole surface; the tilt is
  now much weaker. The title changes colour faster, so it is not grey for long.
- **How it was tested:** a screen video and frames at five moments.
- **What we checked / changed:** [...]

## Entry 20 – Waves in one colour and stronger (AI-generated code)

- **What we asked:** all wave layers in the same colour, and stronger waves.
- **What the AI changed in `streamlit_app.py` (HOME PAGE):** the lighter, see-through wave layer
  now has the colour of the wine like the others; all wave layers are about 1.5 times higher and
  they grow and shrink more (`breathe` from 65 % to 135 % instead of 75 % to 120 %).
- **How it was tested:** a screen video and frames at four moments.
- **What we checked / changed:** [...]

## Entry 21 – The wine only rises, more extreme waves (AI-generated code)

- **What we asked:** at the start the wine went up, then down and up again, which we did not
  like; and the waves should be more extreme.
- **What the AI changed in `streamlit_app.py` (HOME PAGE):** the sloshing linked to the scrolling
  (`slosh` and `swell-slosh`) was removed, so the wine level only rises. All wave layers are
  higher again (100 to 150px) and grow and shrink more (`breathe` from 50 % to 150 %). At the
  start the wine is pushed below the window, so the high waves do not show before the pouring.
- **How it was tested:** a screen video, frames at five moments and a screenshot on a tall window.
- **What we checked / changed:** [...]

## Entry 22 – Slower pouring, waves in sync, text colour follows the wine (AI-generated code)

- **What we asked:** keep everything, but the wine should rise a little slower and the waves
  should move more in sync, so it looks natural.
- **What the AI changed in `streamlit_app.py` (HOME PAGE):**
  - the wine now rises over a longer scroll distance (`animation-range: 40vh min(130vh, 100%)`);
    `min(..., 100%)` makes sure that the glass is also full on very tall screens
  - all wave layers move in the same direction at a similar speed and grow and shrink in the
    same 4-second rhythm (a little shifted with negative delays)
  - the texts are light exactly where the wine is: a colour gradient fixed to the window
    (`background-attachment: fixed`) shown only inside the letters (`background-clip: text`),
    split at `--level`, a registered CSS property (`@property`) that moves exactly like the wine
- **How it was tested:** measured on three window heights (800, 900, 1300 px) that the text is
  light on the wine and dark on cream, screenshots while the wine passes the title, a screen video.
- **What we checked / changed:** [...]

---

## Reflection for the video (to fill in at the end of the project)

- **What worked well with AI:** [...]
- **What did not work / where we had to correct the AI:** [...]
- **How we changed our way of using AI during the project:** [...]
- **What we would do differently next time:** [...]
