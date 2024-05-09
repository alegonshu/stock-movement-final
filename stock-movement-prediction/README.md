# CSPC 452 Final Project: Enhancing Financial Forecasting Models with ChatGLM on Scraped Data

## Contents
1. `data_preparations`: contains the scraper for financial news content specifically targeted at the Chinese market. Running  ``


## Data Scraping and Preparation
The corresponding code is located in the `data_preparations` directory.
- **Environment dependencies**: You can install them directly by running `pip install -r requirements.txt`.

### Update and Download Data
- Update `hs_restricted.csv` with the stocks needed.
- Run `download_titles.py` to scrape news data. The data will be stored in `fin_data/scrape/results`. Then use `download_contents.py` to download all content and `fin_data/clean_data.py` to clean it. The cleaned data will be stored in `fin_data/results_with_content_clean`.
- You will need to use KuaiDaili's tunnel proxy. After registering and logging in, go to the "Tunnel Proxy" interface, add a tunnel proxy, and then modify the following content in the config variable:
  - `tunnel`: `<Tunnel host>:<HTTP port>`
  - `username`: `<username>`
  - `password`: `<password>`
  - Currently, the corresponding information in the code is from my KuaiDaili account, which still has some balance, so you can continue to use it if needed.

### Data Processing
- Run `read_clickhouse_data.py` to read relevant news and announcements from Clickhouse; the data will be stored in `fin_data/clickhouse_data`.
- Run `add_labels.py` to add labels and stock price information to each piece of news/announcement. The data is stored in `fin_data/content_with_labels`.
  - You can adjust `THRESH_VERY`, `THRESH`, `FORWARD_DAYS`, `END_DATE`.
  - The code uses akshare to read stock information from January 1, 2012, to `END_DATE`.
  - Use `UnivariateSpline` to fit the stock price curve. If you want to observe the fitted curve, you can run `fin_data/candle_figure.py` to view it.
  - Labels are defined as the slope of the stock price curve on the nearest stock trading day after the news/announcement is released:
    - `s < -THRESH_VERY`: very_negative (sell)
    - `-THRESH_VERY <= s < -THRESH`: negative (moderate sell)
    - `-THRESH <= s <= THRESH`: neutral (hold)
    - `THRESH < s <= THRESH_VERY`: positive (buy)
    - `s > THRESH_VERY`: very_positive (strong buy)
  - Add the slope of the stock price curve `FORWARD_DAYS` days before the news/announcement as judgment information.

### Dataset Arrangement
- Run `arrange_dataset.py` to bundle news information from the same day. The data is stored in `fin_data/content_with_labels`.
  - The current packaging method is: if a stock has more than 5 news + announcement items on the same day, they are randomly packaged into multiple groups of 5 each.
  - Announcement information is marked with "【公告】".

### Dataset Conversion
- Run `making_dataset.py` to convert the dataset into jsonl format. The dataset is stored in `fin_data/jsonl`.
  - Set the data between `train_start_date` and `test_start_date` as the training set, and data after `test_start_date` as the test set.
- Finally, the size of the training set, the test set, and the maximum data length will be printed. The `max_source_length` during p-tuning should not be less than this length.
