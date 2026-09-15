# Book Recommendation System

A Flask web app that recommends books using **item-based collaborative filtering**. Type in a book you liked and it returns four similar titles — with covers and authors — based on how thousands of other readers rated them together.

It has two pages: a homepage showing the most popular, highly-rated books, and a search page for personalised recommendations.

---

## How it works

The engine is collaborative filtering, not content matching. It never looks at genre, blurb, or subject matter. Instead it works from a **user-book ratings pivot table** (`pt.pkl`), where each book is represented as a vector of ratings across users.

Two books end up close together in that space when the same people tend to rate both — so the recommendations reflect real reading patterns rather than surface similarity. A literary novel and a pop-science title could be linked simply because the same audience reads both.

**Cosine similarity** between every pair of book vectors is precomputed and stored in `similarity_scores.pkl`. At request time the app just looks up the matching row, sorts by score, and skips the top result — which is always the book you searched for — to return the next four.

The homepage is separate and simpler: a popularity ranking of well-rated books with enough ratings to be trustworthy, served straight from `popular.pkl`.

---

## Setup

**1. Clone the repo**

```bash
git clone https://github.com/Sanchitamaiti/book_reccomendation_system.git
cd book_reccomendation_system
```

**2. Unzip the books data — this step is required**

`app.py` loads `books.pkl`, but the repo ships it compressed as `books.zip`. The app will crash with a `FileNotFoundError` on startup if you skip this:

```bash
unzip books.zip
```

Make sure the extracted `books.pkl` sits in the project root, next to `app.py`.

**3. Install dependencies**

```bash
pip install flask pandas numpy
```

**4. Run**

```bash
python app.py
```

Then open `http://127.0.0.1:5000` in your browser.

---

## Project structure

```
book_reccomendation_system/
├── app.py                    # Flask routes and recommendation logic
├── templates/
│   ├── index.html            # Homepage — popular books grid
│   └── recommend.html        # Search page and results
├── books.zip                 # Book metadata (unzip to books.pkl before running)
├── popular.pkl               # Pre-ranked popular books
├── pt.pkl                    # User-book ratings pivot table
└── similarity_scores.pkl     # Precomputed cosine similarity matrix
```

---

## Routes

| Route | Method | Purpose |
|---|---|---|
| `/` | GET | Homepage with the popular books grid |
| `/recommend` | GET | The search form |
| `/recommend_books` | POST | Takes a book title, returns four recommendations |

If the title isn't in the dataset — or the field is submitted empty — the app returns the search page with a friendly "no results found" message rather than erroring out.

---

## Data

Built on the **Book-Crossing dataset**, which pairs user ratings with book metadata. The fields used are book title, author, cover image URL, rating count, and average rating.

Because recommendations come from co-rating patterns, a book needs a reasonable number of ratings before it has a meaningful similarity vector. Very obscure titles either won't appear at all or will produce weak matches — a normal limitation of collaborative filtering, known as the cold-start problem.

Book covers are loaded from external image URLs in the dataset. Some are dead links by now, so the occasional missing cover is expected.

---

## Notes and limitations

- **Exact title matching.** The lookup requires the title to match the dataset exactly, so "harry potter" won't find *Harry Potter and the Sorcerer's Stone*. Adding an autocomplete dropdown populated from `pt.index` would fix this and is the single highest-impact improvement available.
- **The model artifacts are pre-built.** The pickles are committed directly and the training notebook isn't in this repo, so the app runs instantly but the pipeline can't be retrained from here.
- **Debug mode is on.** `app.run(debug=True)` is fine locally but should be turned off before deploying anywhere public.
- **Pickle compatibility.** The `.pkl` files carry pandas objects, so a very different pandas version than the one used to create them may fail to unpickle.

---

## Possible extensions

Fuzzy or autocomplete title search, a hybrid model blending collaborative filtering with content features like genre and description, a configurable number of recommendations, and deployment to Render or Railway with a proper `requirements.txt` and WSGI server.

---

## Author

**Sanchita Maiti** — [GitHub](https://github.com/Sanchitamaiti)

Built with Flask, pandas, and NumPy.
