# 📚 Book-Crossing Analysis

> Exploratory data analysis of the Book-Crossing dataset — 278,858 users, 271,379 books, and 1,149,780 ratings.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Dataset Description](#dataset-description)
3. [Project Structure](#project-structure)
4. [Requirements](#requirements)
5. [Getting Started (Google Colab)](#getting-started-google-colab)
6. [Notebook Walkthrough](#notebook-walkthrough)
7. [Key Findings](#key-findings)
8. [Compatibility Notes](#compatibility-notes)
9. [Known Limitations](#known-limitations)

---

## Project Overview

This project performs a full exploratory data analysis (EDA) on the **Book-Crossing dataset**, a publicly available collection of book ratings gathered from the Book-Crossing community. The goal is to understand:

- Who the readers are (demographics, location, age distribution)
- Which books, authors, and publishers are most popular
- How users rate books (explicit vs. implicit ratings)
- Trends in publication years

The analysis is implemented as a **Google Colab-compatible Jupyter notebook** using Python and standard data science libraries.

---

## Dataset Description

The dataset consists of three CSV files, all semicolon-delimited and `latin-1` encoded:

### `BX-Books.csv`
| Column | Description |
|---|---|
| `ISBN` | International Standard Book Number — unique identifier for each book |
| `Book-Title` | Title of the book |
| `Book-Author` | Author's name |
| `Year-Of-Publication` | Year the book was published |
| `Publisher` | Publishing company |
| `Image-URL-S` | URL to a small cover image |
| `Image-URL-M` | URL to a medium cover image |
| `Image-URL-L` | URL to a large cover image |

### `BX-Users.csv`
| Column | Description |
|---|---|
| `User-ID` | Anonymised numeric user identifier |
| `Location` | Free-text location string (`city, state, country`) |
| `Age` | User's age (nullable; many values are missing or invalid) |

### `BX-Book-Ratings.csv`
| Column | Description |
|---|---|
| `User-ID` | Matches `User-ID` in `BX-Users.csv` |
| `ISBN` | Matches `ISBN` in `BX-Books.csv` |
| `Book-Rating` | Integer 0–10. `0` = implicit interaction (no explicit rating given); 1–10 = explicit rating |

**Scale of the data:**
- 278,858 users
- 271,379 books
- 1,149,780 ratings

---

## Project Structure

```
book-crossing-project/
│
├── book_crossing_big_data_project.ipynb   # Main analysis notebook
├── BX-Books.csv                           # Book metadata
├── BX-Users.csv                           # User demographics
├── BX-Book-Ratings.csv                    # Rating data
└── README.md                              # This file
```

---

## Requirements

### Python Packages

| Package | Purpose |
|---|---|
| `pandas` | Data loading, cleaning, and transformation |
| `numpy` | Numerical operations and NaN handling |
| `matplotlib` | Base plotting |
| `seaborn` | Statistical visualisations |
| `wordcloud` | Word cloud generation for book titles |
| `colorama` | Coloured terminal output for dtype printing |
| `Pillow` (PIL) | Image handling |
| `requests` | Fetching book cover images from URLs |
| `ydata-profiling` | Automated EDA profile report (optional but recommended) |

Install all optional dependencies in Colab by running the cell at the top of the notebook:

```python
!pip install ydata-profiling wordcloud colorama -q
```

> **Note:** `pandas`, `numpy`, `matplotlib`, `seaborn`, `Pillow`, and `requests` are pre-installed in Google Colab.

---

## Getting Started (Google Colab)

### Option A — Upload CSVs directly (simplest)

1. Open the notebook in [Google Colab](https://colab.research.google.com/)
2. In the left sidebar, click the **Files** icon (folder)
3. Upload `BX-Books.csv`, `BX-Users.csv`, and `BX-Book-Ratings.csv`
4. Leave `DATA_PATH = ''` in the first code cell (default)
5. Run all cells: **Runtime → Run all**

### Option B — Use Google Drive

1. Upload the three CSVs to a folder in your Google Drive (e.g. `My Drive/book-crossing/`)
2. In the first code cell, uncomment the Drive mount lines:
   ```python
   from google.colab import drive
   drive.mount('/content/drive')
   DATA_PATH = '/content/drive/MyDrive/book-crossing/'
   ```
3. Run all cells

---

## Notebook Walkthrough

### 1. Environment Setup
Checks that all three CSV files are accessible and prints their paths. Sets up the `DATA_PATH` variable used throughout.

### 2. Imports
Loads all required libraries. Installs `ydata-profiling`, `wordcloud`, and `colorama` if not already present. Defines colour shorthand variables using `colorama` for formatted console output.

### 3. Custom Colour Palette
Defines a teal-to-mint colour palette (`#48bfe3` → `#80ffdb`) used consistently across all charts.

### 4. Reading the CSV Files
Loads all three datasets with `pd.read_csv` using semicolon separators and `latin-1` encoding. Column names are normalised to lowercase with underscores (e.g. `Book-Title` → `book_title`).

### 5. Data Exploration
Prints `.head()`, `.describe()`, and `.dtypes` for each of the three dataframes so you can visually inspect the raw data before cleaning.

### 6. Type Casting & Null Handling
- `age`, `user_id`, `rating` are cast with `pd.to_numeric(..., errors='coerce')` to safely handle dirty string values
- Ages outside the range 5–99 are replaced with `NaN`, then filled with the column mean
- Two books with missing `publisher` values are manually filled using their ISBN
- One book with a missing `book_author` is manually filled
- Publication years of `0` or beyond 2008 are replaced with the column mean

### 7. Merging
The three dataframes are merged into a single `df`:
- `users` + `ratings` on `user_id`
- Result + `items` on `isbn`

This produces a flat table where each row represents one rating event with full user and book context.

### 8. Location Splitting
The `location` column (format: `"city, state, country"`) is split into three separate columns: `city`, `state`, `country`.

### 9. Book Cover Images
Demonstrates fetching and displaying book cover images at small, medium, and large sizes using the Amazon image URLs embedded in the dataset. Image display uses `IPython.display.Image`.

### 10. Column Cleanup
Drops `location`, `img_s`, `img_m`, `img_l` from the merged dataframe — they are no longer needed after splitting and image display.

### 11. Automated Profile Report (ydata-profiling)
Generates a comprehensive automated EDA report with correlations, distributions, and missing value summaries. Renders inline in the notebook. Skipped gracefully if `ydata-profiling` is not installed.

### 12. Rating Distribution
Bar chart of all rating values (0–10). Rating `0` dominates, representing implicit interactions (page views, borrows) with no explicit score.

### 13. Explicit Rating Distribution
Same chart filtered to ratings 1–10 only. Shows that users tend to give higher ratings — the distribution is left-skewed with a peak around 8.

### 14. Age Distribution
Histogram of user ages after outlier removal. Most active users are in the 25–45 range.

### 15. Top 25 Years of Publication
Horizontal bar chart showing the most common publication years among rated books.

### 16. Top 25 Books, Authors, Publishers
Three horizontal bar charts generated by a shared `barplot()` helper function, showing the most-rated books, most-rated authors, and most-rated publishers.

### 17. Word Cloud
A word cloud of book titles using a custom teal colour scheme. Common stop words are excluded. The top 100 most frequent words are rendered.

---

## Key Findings

- **Implicit ratings dominate.** The vast majority of `rating` values are `0`, meaning most user–book interactions recorded were passive (views/borrows) rather than deliberate scores.
- **Users rate generously.** Among explicit ratings (1–10), scores cluster at the high end — 8, 9, and 10 are the most common values.
- **Most users are adults aged 25–45**, with a sharp drop-off at both extremes. Many age entries were invalid (>99 or <5) and were cleaned prior to analysis.
- **Popular books skew toward well-known fiction.** The top-rated titles are dominated by bestselling novels.
- **Publication years cluster in the 1990s–early 2000s**, consistent with when the Book-Crossing community was most active.

---

## Compatibility Notes

This notebook was originally written in 2019 for Kaggle. The following breaking changes were addressed to ensure full compatibility with modern Google Colab (2024+):

| Issue | Fix Applied |
|---|---|
| `pandas_profiling` package removed | Replaced with `ydata_profiling` (the official successor) |
| `sns.distplot()` removed in seaborn ≥ 0.12 | Replaced with `sns.histplot()` |
| `sns.palplot(size=...)` removed in seaborn ≥ 0.13 | Removed the `size` argument |
| `value_counts().reset_index()` column naming changed in pandas ≥ 2.0 | Updated column assignments after `reset_index()` |
| Kaggle-specific file paths (`/kaggle/input/...`) | Replaced with a configurable `DATA_PATH` variable |
| CSV read assumed no header row | Fixed — CSVs do have headers; `names=` and `drop(index[0])` removed |
| `items['year_of_publication'].astype(int)` failed on dirty strings | Replaced with `pd.to_numeric(..., errors='coerce')` |
| `IPython.core.display.Image` (legacy path) | Updated to `IPython.display.Image` |
| Direct `df[col][i]` indexing raised pandas warnings | Replaced with `df[col].iloc[i]` |

---

## Known Limitations

- **Image URLs may be broken.** The Amazon image URLs in the dataset are several years old and some may no longer resolve. This affects only the image display cells; the rest of the analysis is unaffected.
- **Location data is noisy.** The free-text `location` field contains inconsistencies (missing states, non-standard country names). The city/state/country split is best-effort.
- **Age imputation is simple.** Invalid ages are replaced with the column mean, which may introduce bias in any age-based analysis.
- **Dataset is static.** The Book-Crossing dataset was collected in 2004. It does not reflect modern reading or rating behaviour.
