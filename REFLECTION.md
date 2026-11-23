# Reflection

Name:

Answer each question in 2–3 sentences.

**1. Which antipatterns from the lecture did `train_all.py` show? Point to the lines.**

> It is a monolith: load, clean, features, train, evaluate and save all sit in one top-to-bottom script, so no stage can be tested or reused alone. Settings are hard-coded inside the logic (the path on line 16, `max_depth=8` on line 41, the 10,000 price cutoff on line 21). Dead code stays in the file: the `USE_LOG_PRICE` experiment on lines 30–35, which the comment itself calls "Not sure. Keep for now". There is also no test and no evaluation gate. The script prints the MAE on line 46 but always saves the model on line 50, however bad it is.

**2. Suppose there were no test and no evaluation gate. What would have happened when the 2026 batch arrived? Think about both the crash and the missing `sqft` values.**

> The crash is loud. The 2026 dates look like `2026-02-23T10:30:00+07:00`, but line 20 forces `format="%Y-%m-%d"`, so `pd.to_datetime` raises a `ValueError` and the run dies with no model. The missing `sqft` values (12 of 150 rows) are the dangerous part. Once the date bug is fixed, nothing removes those rows. The model would train on bad input and still save, and with no gate, nobody would learn that its quality had dropped until wrong prices reached users.

**3. In the real marketplace, which non-ML parts of the system should trigger this pipeline or feed it data?**

> Data comes from the listings and transactions database, which records each completed sale. Data-entry forms and partner or MLS feeds also supply sales, and their validation rules decide how clean the data is. The trigger could be a scheduler such as Airflow or a cron job (for example weekly retraining) or a new-data event from the data warehouse, with CI such as GitHub Actions running tests on every code change. Afterwards, the model registry and the pricing service or app backend that serves predictions consume the output.
