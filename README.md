# 🎵 Spotify ETL Pipeline

A production-style Python ETL pipeline that extracts a user's recently played tracks from the **Spotify Web API**, applies data quality checks and transformations, and loads the cleaned data into a local database — fully automated for scheduled execution.

---

## 📌 Project Overview

This project implements a complete **Extract → Transform → Load** pipeline on real Spotify listening data. It is designed to be modular, reproducible, and easy to extend for analytics or machine learning use cases.

**Pipeline stages:**

```
Spotify API  ──►  Extract  ──►  Validate & Transform  ──►  Load (SQLite / PostgreSQL)
                                      │
                              • Duplicate detection
                              • Null value checks
                              • Schema enforcement
                              • Feature engineering
```

---

## 🗂️ Project Structure

```
spotify_etl/
│
├── spotify_etl.py        # Core ETL logic (extract, validate, transform, load)
├── extract.py            # Spotify API connection and data extraction
├── transform.py          # Data cleaning, validation, and transformation
├── load.py               # Database loading (SQLite / PostgreSQL)
├── dag.py                # Apache Airflow DAG for pipeline scheduling
├── requirements.txt      # Python dependencies
├── .env.example          # Environment variable template
└── README.md
```

---

## ⚙️ Features

- **Extract** — Pulls recently played tracks (up to 50) from Spotify's `user-read-recently-played` endpoint using the Spotipy library
- **Validate** — Enforces data quality rules: empty DataFrame check, null value detection, primary key uniqueness
- **Transform** — Structures raw JSON into a clean Pandas DataFrame with typed columns; engineers artist-level and time-based aggregations
- **Load** — Inserts validated records into a local SQLite database (or PostgreSQL) using SQLAlchemy, with upsert logic to avoid duplicates
- **Schedule** — Apache Airflow DAG triggers the pipeline on a configurable interval (default: daily)

---

## 🛠️ Tech Stack

| Layer | Tool |
|---|---|
| Language | Python 3.8+ |
| API Client | Spotipy |
| Data Processing | Pandas |
| Database ORM | SQLAlchemy |
| Database | SQLite · PostgreSQL |
| Orchestration | Apache Airflow |
| Containerization | Docker / Docker Compose |
| Environment | python-dotenv |

---

## 🚀 Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/JasserHedhli/spotify_etl.git
cd spotify_etl
```

### 2. Set up a virtual environment

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Configure environment variables

Copy the example file and fill in your credentials:

```bash
cp .env.example .env
```

```env
SPOTIPY_CLIENT_ID=your_spotify_client_id
SPOTIPY_CLIENT_SECRET=your_spotify_client_secret
SPOTIPY_REDIRECT_URI=http://localhost:8888/callback
DATABASE_URL=sqlite:///my_played_tracks.sqlite
```

> **Getting Spotify credentials:** Go to [Spotify for Developers](https://developer.spotify.com/dashboard), create an app, and copy your **Client ID** and **Client Secret**. Add your redirect URI in the app settings.

### 4. Run the pipeline manually

```bash
python spotify_etl.py
```

This will extract, validate, transform, and load your recently played tracks into the local database.

---

## 📅 Automated Scheduling with Airflow

### Option A — Local Airflow

```bash
export AIRFLOW_HOME=$(pwd)/airflow
airflow db init
airflow scheduler &
airflow webserver --port 8080
```

Place `dag.py` inside `airflow/dags/` and enable the DAG from the Airflow UI at `http://localhost:8080`.

### Option B — Docker Compose

```bash
docker-compose up --build
```

Access the Airflow UI at `http://localhost:8080` (default credentials: `airflow` / `airflow`).

---

## 🗃️ Database Schema

The pipeline loads data into a `played_tracks` table with the following structure:

| Column | Type | Description |
|---|---|---|
| `played_at` | TIMESTAMP (PK) | Timestamp of when the track was played |
| `track_name` | VARCHAR | Name of the track |
| `artist_name` | VARCHAR | Primary artist |
| `album_name` | VARCHAR | Album the track belongs to |
| `track_duration_ms` | INTEGER | Track duration in milliseconds |
| `popularity` | INTEGER | Spotify popularity score (0–100) |

---

## 📊 Sample Output

```
played_at             | track_name          | artist_name      | album_name         | popularity
----------------------|---------------------|------------------|--------------------|----------
2024-07-15 22:14:00   | Blinding Lights     | The Weeknd       | After Hours        | 95
2024-07-15 21:50:00   | Levitating          | Dua Lipa         | Future Nostalgia   | 89
2024-07-15 21:30:00   | Save Your Tears     | The Weeknd       | After Hours        | 91
```

---

## ✅ Data Quality Checks

Before loading, the pipeline enforces:

- **Empty DataFrame** — aborts if no data was returned from the API
- **Null values** — flags and drops records with missing critical fields
- **Duplicate primary keys** — deduplicates on `played_at` to ensure idempotent loads
- **Schema validation** — enforces expected column types and names

---

## 🔧 Configuration

| Parameter | Default | Description |
|---|---|---|
| `LOOKBACK_DAYS` | `1` | Number of past days to extract data for |
| `API_LIMIT` | `50` | Max tracks per API call (Spotify cap) |
| `DAG_SCHEDULE` | `@daily` | Airflow schedule interval |
| `DATABASE_URL` | SQLite local | Target database connection string |

---

## 📈 Potential Extensions

- Add **listening trend dashboards** using Power BI or Streamlit
- Implement **refresh token** rotation to eliminate manual re-authentication
- Extend schema with **audio features** (tempo, energy, danceability) via the `/audio-features` endpoint
- Deploy Airflow to **cloud** (AWS MWAA, GCP Composer) for 24/7 automation
- Build a **music recommendation engine** on top of accumulated listening history

---

## 👤 Author

**Jasser Hedhli** — Data Scientist & PhD Researcher  
[LinkedIn](https://linkedin.com/in/jasser-hedhli) · [GitHub](https://github.com/JasserHedhli) · jesserhedhli@gmail.com

---

## 📄 License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
