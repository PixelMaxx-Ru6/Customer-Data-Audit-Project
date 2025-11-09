# Customer-Data-Audit-Project
“Imagine a company onboarding 10,000 customer records from multiple regions. This script ensures the data is clean, identifies issues, and produces actionable summaries for stakeholders.”
Clone the repo

Place customers-10000.csv  in the root directory

Run main_script.py

Check the output files and console summary
"""

output_path = "/mnt/data/README.md"
with open(output_path, "w", encoding="utf-8") as f:
f.write(readme_content)

print("README.md file created for GitHub project 'Customer Data Audit & Regional Analytics'")

import pandas as pd
import numpy as np

# === Load dataset ===
file_path = "customers-10000.csv"
df = pd.read_csv(file_path)

# === 1. Data Cleaning ===
df['Country'] = df['Country'].astype(str).str.strip()
df['Subscription Date'] = pd.to_datetime(df['Subscription Date'], errors='coerce')

# === 2. Detect Data Quality Issues ===
data_issues = {}

# Missing values
missing = df.isnull().sum()
missing = missing[missing > 0]
if not missing.empty:
    data_issues['Missing Values'] = missing.to_dict()

# Duplicate rows
duplicates = df.duplicated(subset=['Customer Id']).sum()
if duplicates > 0:
    data_issues['Duplicate Customer IDs'] = int(duplicates)

# Invalid dates
invalid_dates = df['Subscription Date'].isna().sum()
if invalid_dates > 0:
    data_issues['Invalid or Missing Subscription Dates'] = int(invalid_dates)

# Invalid emails
invalid_emails = df[~df['Email'].str.contains(r'^[\w\.-]+@[\w\.-]+\.\w+$', na=False)]
if len(invalid_emails) > 0:
    data_issues['Invalid Email Format'] = len(invalid_emails)

# === 3. Sort & Group by Region ===
df_sorted = df.sort_values(by=['Country', 'Subscription Date'], ascending=[True, True])

# === 4. Regional Summary ===
region_summary = (
    df_sorted.groupby('Country')
    .agg(
        Total_Customers=('Customer Id', 'count'),
        Earliest_Subscription=('Subscription Date', 'min'),
        Latest_Subscription=('Subscription Date', 'max')
    )
    .reset_index()
)

# Add duration between earliest and latest subscription per region
region_summary['Subscription_Span_Days'] = (
    region_summary['Latest_Subscription'] - region_summary['Earliest_Subscription']
).dt.days

# === 5. Determine Update Frequency ===
# Calculate average number of days between updates
all_dates = df['Subscription Date'].dropna().sort_values()
if len(all_dates) > 1:
    avg_update_gap = np.mean(np.diff(all_dates)).days
else:
    avg_update_gap = np.nan

update_frequency = {
    "Total Records": len(df),
    "Earliest Record": all_dates.min(),
    "Latest Record": all_dates.max(),
    "Average Days Between Updates": round(avg_update_gap, 2) if not np.isnan(avg_update_gap) else "N/A"
}

# === 6. Save Outputs ===
df_sorted.to_csv("customers_by_region_sorted.csv", index=False)
region_summary.to_csv("region_summary.csv", index=False)

# Create a Data Health Report
with open("data_health_report.txt", "w") as f:
    f.write("=== DATA HEALTH REPORT ===\n\n")
    f.write(f"Total Records: {len(df)}\n")
    f.write(f"Earliest Subscription: {update_frequency['Earliest Record']}\n")
    f.write(f"Latest Subscription: {update_frequency['Latest Record']}\n")
    f.write(f"Average Days Between Updates: {update_frequency['Average Days Between Updates']}\n\n")
    f.write("=== DATA QUALITY ISSUES ===\n")
    if data_issues:
        for issue, details in data_issues.items():
            f.write(f"- {issue}: {details}\n")
    else:
        f.write("No major data quality issues detected.\n")

# === 7. Display Summary in Console ===
print("✅ Analytics complete! Data grouped by region and checked for issues.\n")
print("📊 Summary saved as: region_summary.csv")
print("📁 Sorted dataset: customers_by_region_sorted.csv")
print("🩺 Data Health Report: data_health_report.txt\n")
print("=== Average Update Info ===")
print(update_frequency)
print("\n=== Sample Region Summary ===")
print(region_summary.head(10))
print("\n=== Detected Issues ===")
print(data_issues if data_issues else "No major data issues detected ✅")
