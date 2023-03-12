---
title: Loading Amplitude Events Data into Pandas DataFrame
description: Amplitude allows you to export the events data from their platform using their Export API. Let’s see how we can get that exported data loaded into a Pandas dataframe.
date: 2023-03-12 21:20:01
keywords: Pandas, Amplitude
tags:
  - Python
  - Pandas
  - Analytics
  - Amplitude
---

Amplitude allows you to export the events data from their platform using their [Export API](https://www.docs.developers.amplitude.com/analytics/apis/export-api/). Let’s see how we can get that exported data loaded into a Pandas dataframe.

First, get the Amplitude API credentials for your project.

```python
API_KEY = "xxxx"
SECRET_KEY = "yyyy"
```

The data is exported based on a date range, but there are limitations on the size of the data which can be exported in a single API call. So depending on the size of the data, we might need to make multiple API calls.

For this post, I’m assuming a day’s worth of data falls within this limit. Let’s export the data for the past 7 days.

```python
from datetime import date, timedelta

NUM_DAYS = 7
end_date = date.today()
start_date = end_date - timedelta(days=NUM_DAYS)
```

Since the API response is a zip file, let’s create a temp directory to store them.

```python
from pathlib import Path

raw_files_path = Path.cwd().joinpath("./temp/amplitude")
Path.mkdir(raw_files_path, parents=True, exist_ok=True)
```

As mentioned before, we’re splitting the specified date range into multiple days, and making an API request for each day. The date range for the API request is specified using the `start` and `end` query parameters formatted in the `YYYYMMDDTHH` format. We use the `API_KEY` and `SECRET_KEY` from before, encode them and send them along as the `Authorization` header. The response is then written to a ZIP file in the temp directory.

```python
import requests
from base64 import b64encode
from datetime import timedelta

url = "https://amplitude.com/api/2/export"
token = b64encode(f"{API_KEY}:{SECRET_KEY}".encode("utf-8")).decode("utf-8")
headers = {"Authorization": f"Basic {token}"}

for idx in range((end_date - start_date).days):
    current_date = end_date - timedelta(days=idx)
    current_date = current_date.strftime("%Y%m%d")
    params = {"start": f"{current_date}T00", "end": f"{current_date}T23"}
    res = requests.get(url, params=params, headers=headers)
    open(raw_files_path.joinpath(f"{current_date}.zip"), "wb").write(res.content)
```

Once we’re done downloading the ZIP files, we iterate through them and extract the contents.

```python
import shutil

for path in list(raw_files_path.iterdir()):
    if path.suffix != ".zip":
        continue
    shutil.unpack_archive(path, raw_files_path)
```

Finally, we iterate through the compressed JSON files and load them into dataframe `df`.

```python
import pandas as pd

df = None
for path in list(raw_files_path.iterdir()):
    if not path.is_dir():
        continue
    for file_path in list(path.iterdir()):
        if not file_path.name.endswith(".json.gz"):
            continue
        if df is None:
            df = pd.read_json(file_path, lines=True)
        else:
            new_df = pd.read_json(file_path, lines=True)
            df = pd.concat([df, new_df])
```
