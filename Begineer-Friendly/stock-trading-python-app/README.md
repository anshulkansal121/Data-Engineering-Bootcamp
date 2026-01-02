# Stock Trading Python App (Beginner-Friendly)

A small beginner-friendly Python project that fetches stock tickers from the Polygon API and loads them into Snowflake. Includes a simple scheduler example to run the job every minute (for demonstration).

This README explains the repository structure, required environment variables, how to run the script, and notes about Snowflake loading and scheduling.

---

## Repository contents

- `script.py` — Main script that:
  - Calls the Polygon API to fetch stock tickers
  - Adds a `ds` (date string) field to each record
  - Creates a Snowflake table (if not exists) and inserts the data
- `scheduler.py` — Example scheduler using `schedule` library to run the job repeatedly (demo only).
- `requirements.txt` — Python dependencies.
- `tickers.csv` — Sample/large CSV of tickers included in the folder (can be used for offline testing).
- `.gitignore` — Git ignore settings (includes `.env`).

---

## Requirements

- Python 3.8+
- A Snowflake account (if you want to load data into Snowflake)
- Polygon API key (for ticker metadata)
- The packages in `requirements.txt`

Install dependencies:

```bash
pip install -r requirements.txt
```

Note: `requirements.txt` in repo lists `requests`, `dotenv`, `openai`, `schedule`. If you prefer, replace `dotenv` with `python-dotenv` for the commonly used package name.

---

## Environment variables

Create a `.env` file (this repo already ignores `.env`) with the following variables:

Required:
- `POLYGON_API_KEY` — Your Polygon API key.

Snowflake connection variables (used by `script.py`):
- `SNOWFLAKE_USER`
- `SNOWFLAKE_PASSWORD`

Optional / recommended (will be used if set):
- `SNOWFLAKE_ACCOUNT`
- `SNOWFLAKE_WAREHOUSE`
- `SNOWFLAKE_DATABASE`
- `SNOWFLAKE_SCHEMA`
- `SNOWFLAKE_ROLE`
- `SNOWFLAKE_TABLE` — Defaults to `stock_tickers` when not set.

Example `.env`:
```
POLYGON_API_KEY=your_polygon_api_key
SNOWFLAKE_USER=YOUR_USER
SNOWFLAKE_PASSWORD=YOUR_PASSWORD
SNOWFLAKE_ACCOUNT=your_account
SNOWFLAKE_WAREHOUSE=COMPUTE_WH
SNOWFLAKE_DATABASE=MY_DB
SNOWFLAKE_SCHEMA=PUBLIC
SNOWFLAKE_ROLE=MY_ROLE
SNOWFLAKE_TABLE=stock_tickers
```

---

## Usage

1. Ensure dependencies installed and `.env` created.
2. Run the job once:

```bash
python script.py
```

This will:
- Fetch active stock tickers from Polygon (with pagination)
- Add `ds` with today's date
- Create `SNOWFLAKE_TABLE` if it doesn't exist and insert rows

3. Run the scheduler demo (runs every minute):

```bash
python scheduler.py
```

Note: `scheduler.py` demonstrates scheduling using the `schedule` package and runs in-process. It requires the host system to remain running and is not suited for production (see "Deployment / Production" below).

---

## Snowflake behavior

- The script constructs a table schema derived from an example ticker and creates the table if it doesn't exist.
- Type overrides include `VARCHAR`, `BOOLEAN`, and `TIMESTAMP_NTZ` for `last_updated_utc`.
- The script uses `cursor.executemany` to insert rows in bulk.
- Client telemetry is disabled via session parameter `"CLIENT_TELEMETRY_ENABLED": False`.

---

## Notes & caveats

- Pagination: the Polygon API may return multiple pages; the script follows `next_url` to paginate. Be mindful of API rate limits.
- `tickers.csv` is included for offline testing or reference. It may be large.
- `scheduler.py` is a simple in-process scheduler — consider using cron, a cloud scheduler, or an orchestration tool (Airflow, Prefect, etc.) for production.
- Secrets: `.env` is listed in `.gitignore` — keep credentials out of source control.

---

## Troubleshooting

- If Snowflake connection fails, confirm all Snowflake environment variables are correct and that network access (e.g., IP allowlist) is configured.
- If Polygon responses are empty or rate-limited, confirm `POLYGON_API_KEY` and check API quota.
- If using `dotenv`, ensure your environment loader (the repo uses `load_dotenv()` in `script.py`) loads variables before the connection.

---

## Next steps / Improvements (ideas)

- Add retries and backoff for API calls.
- Improve logging (instead of prints) and add error handling.
- Add unit tests and CI.
- Make table name and schema mapping configurable.
- Make scheduler production-ready using a managed scheduler or orchestration framework.

---

## License

Add a LICENSE file if you plan to open-source this project. (No license included here by default.)
