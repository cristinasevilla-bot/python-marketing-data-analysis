# Python Marketing Data Cheatsheet

A practical reference for analyzing marketing and funnel data with Python, pandas and matplotlib.

---

## 1. Import tools

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```

| Tool | Purpose |
|---|---|
| `pandas` | Work with tables and datasets |
| `numpy` | Handle calculations and missing values |
| `matplotlib` | Create charts and visual reports |

---

## 2. Load a CSV file

```python
df = pd.read_csv("marketing_funnel_sample.csv")
```

Show the first rows:

```python
df.head()
```

Show the column names:

```python
df.columns
```

---

## 3. Clean column names

```python
df.columns = df.columns.str.lower().str.strip().str.replace(" ", "_")
```

Example:

```text
Lead Source -> lead_source
```

---

## 4. Create a summary table

### Group by channel

```python
summary = df.groupby("channel")[["leads", "attendees", "sales", "cost", "revenue"]].sum()
```

### Group by campaign

```python
summary = df.groupby("campaign")[["leads", "attendees", "sales", "cost", "revenue"]].sum()
```

### Group by country

```python
summary = df.groupby("country")[["leads", "attendees", "sales", "cost", "revenue"]].sum()
```

### Group by device

```python
summary = df.groupby("device")[["leads", "attendees", "sales", "cost", "revenue"]].sum()
```

### Group by country and device

```python
summary = df.groupby(["country", "device"])[["leads", "attendees", "sales", "cost", "revenue"]].sum()
```

---

## 5. Calculate marketing KPIs

```python
summary["lead_to_sale"] = summary["sales"] / summary["leads"] * 100
summary["show_up_rate"] = summary["attendees"] / summary["leads"] * 100
summary["attendee_to_sale"] = summary["sales"] / summary["attendees"] * 100
summary["cpl"] = summary["cost"] / summary["leads"]
summary["cac"] = summary["cost"] / summary["sales"]
summary["roas"] = summary["revenue"] / summary["cost"]
```

| KPI | Formula | Meaning |
|---|---|---|
| `lead_to_sale` | sales / leads x 100 | Percentage of leads that become customers |
| `show_up_rate` | attendees / leads x 100 | Percentage of registered leads who attend |
| `attendee_to_sale` | sales / attendees x 100 | Percentage of attendees who buy |
| `cpl` | cost / leads | Cost per lead |
| `cac` | cost / sales | Customer acquisition cost |
| `roas` | revenue / cost | Revenue generated for every euro spent |

---

## 6. Round numbers

```python
summary = summary.round(2)
```

---

## 7. Sort from best to worst

### Sort by ROAS

```python
summary.sort_values(by="roas", ascending=False)
```

### Sort by lead-to-sale conversion

```python
summary.sort_values(by="lead_to_sale", ascending=False)
```

### Sort by show-up rate

```python
summary.sort_values(by="show_up_rate", ascending=False)
```

```text
ascending=False = highest to lowest
ascending=True = lowest to highest
```

---

## 8. Filter data

### Only Email

```python
email_df = df[df["channel"] == "Email"]
```

### Only Meta Ads

```python
meta_df = df[df["channel"] == "Meta Ads"]
```

### Only Spain

```python
spain_df = df[df["country"] == "ES"]
```

### Spain + Email

```python
spain_email_df = df[(df["country"] == "ES") & (df["channel"] == "Email")]
```

| Symbol | Meaning |
|---|---|
| `&` | AND |
| `|` | OR |

Example:

```python
df[(df["country"] == "ES") & (df["channel"] == "Email")]
```

Means:

```text
Show only rows where country is Spain AND channel is Email.
```

---

## 9. Reusable summary function

```python
def create_summary(data, group_by):
    summary = data.groupby(group_by)[["leads", "attendees", "sales", "cost", "revenue"]].sum()

    summary["lead_to_sale"] = summary["sales"] / summary["leads"] * 100
    summary["show_up_rate"] = summary["attendees"] / summary["leads"] * 100
    summary["attendee_to_sale"] = summary["sales"] / summary["attendees"] * 100
    summary["cpl"] = summary["cost"] / summary["leads"]
    summary["cac"] = np.where(summary["sales"] > 0, summary["cost"] / summary["sales"], np.nan)
    summary["roas"] = summary["revenue"] / summary["cost"]

    summary = summary.round(2)

    return summary.sort_values(by="roas", ascending=False)
```

### How to use it

By channel:

```python
create_summary(df, "channel")
```

By campaign:

```python
create_summary(df, "campaign")
```

By country:

```python
create_summary(df, "country")
```

By device:

```python
create_summary(df, "device")
```

By country + device:

```python
create_summary(df, ["country", "device"])
```

With a filter:

```python
email_df = df[df["channel"] == "Email"]

create_summary(email_df, "campaign")
```

---

## 10. Create charts

### ROAS by channel

```python
channel_summary = create_summary(df, "channel")

channel_summary["roas"].plot(
    kind="bar",
    title="ROAS by Channel"
)

plt.xlabel("Channel")
plt.ylabel("ROAS")
plt.xticks(rotation=30, ha="right")
plt.show()
```

### Lead-to-sale rate by channel

```python
channel_summary["lead_to_sale"].plot(
    kind="bar",
    title="Lead to Sale Rate by Channel"
)

plt.xlabel("Channel")
plt.ylabel("Lead to Sale Rate (%)")
plt.xticks(rotation=30, ha="right")
plt.show()
```

### Show-up rate by channel

```python
channel_summary["show_up_rate"].plot(
    kind="bar",
    title="Show Up Rate by Channel"
)

plt.xlabel("Channel")
plt.ylabel("Show Up Rate (%)")
plt.xticks(rotation=30, ha="right")
plt.show()
```

---

## 11. Export results

Save a summary table as CSV:

```python
summary.to_csv("summary_results.csv")
```

Download it in Google Colab:

```python
from google.colab import files

files.download("summary_results.csv")
```

---

## 12. Common business questions and code

### Which channel is most profitable?

```python
create_summary(df, "channel")
```

Look at:

```text
roas
```

### Which campaign works best?

```python
create_summary(df, "campaign")
```

Look at:

```text
roas, lead_to_sale, show_up_rate
```

### Does Spain or France perform better?

```python
create_summary(df, "country")
```

### Does Mobile or Desktop convert better?

```python
create_summary(df, "device")
```

### Which Email campaign performs best?

```python
email_df = df[df["channel"] == "Email"]

create_summary(email_df, "campaign")
```

### Which channel performs best in Spain?

```python
spain_df = df[df["country"] == "ES"]

create_summary(spain_df, "channel")
```

### Which Spain + Email campaign performs best?

```python
spain_email_df = df[(df["country"] == "ES") & (df["channel"] == "Email")]

create_summary(spain_email_df, "campaign")
```

---

## 13. Common errors

### Error: missing comma

Wrong:

```python
df.plot(
    x="channel"
    y="roas"
)
```

Correct:

```python
df.plot(
    x="channel",
    y="roas"
)
```

### Error: column name misspelled

Wrong:

```python
summary["atendee_to_sale"]
```

Correct:

```python
summary["attendee_to_sale"]
```

### Error: `df is not defined`

This means the CSV was not loaded or Colab restarted.

Fix:

```python
df = pd.read_csv("marketing_funnel_sample.csv")
```

### Error: `plt is not defined`

This means matplotlib was not imported.

Fix:

```python
import matplotlib.pyplot as plt
```

---

## 14. Practice method

Every exercise follows this sequence:

```text
1. What do I want to know?
2. Which column should I group by?
3. Do I need a filter?
4. Which KPI should I check?
5. What business conclusion can I make?
```

Example:

```text
I want to know which Google Ads campaign works best.
```

Code:

```python
google_df = df[df["channel"] == "Google Ads"]

create_summary(google_df, "campaign")
```

Conclusion:

```text
The campaign with the highest ROAS is the most profitable. If it also has a strong lead_to_sale rate, it is bringing better-quality leads.
```

---

## 15. Professional interpretation phrases

### If Email wins

```text
Email is the strongest channel by ROAS, suggesting that warm audience nurturing is more profitable than cold acquisition.
```

### If Meta brings many leads but few sales

```text
Meta Ads generates high lead volume, but its low conversion rate suggests a lead quality or targeting issue.
```

### If LinkedIn converts well

```text
LinkedIn Organic shows strong lead quality and purchase intent, even with lower volume.
```

### If Google Ads underperforms

```text
Google Ads requires further keyword and intent analysis before scaling budget.
```

### If Spain performs better

```text
Spain is the strongest-performing market, mainly driven by higher conversion and stronger ROAS.
```

---

## 16. Minimum code to remember

```python
df = pd.read_csv("file.csv")
df.head()

create_summary(df, "channel")

email_df = df[df["channel"] == "Email"]

summary.sort_values(by="roas", ascending=False)

summary["roas"].plot(kind="bar")
```

The goal is not to memorize every line. The goal is to use Python to answer marketing questions with data.
