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
