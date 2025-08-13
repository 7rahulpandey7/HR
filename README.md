import pandas as pd
import numpy as np
from faker import Faker
import random

# Initialize Faker
fake = Faker()

# Number of records and columns
num_records = 500_000
num_columns = 100

# Create base employee IDs
employee_ids = list(range(1, num_records + 1))

# Generate Manager IDs (random employees as managers, but not self)
manager_ids = [random.choice(employee_ids) for _ in range(num_records)]
for i in range(num_records):
    if manager_ids[i] == employee_ids[i]:
        manager_ids[i] = random.choice(employee_ids)

# Pre-generate reference data for IDs
departments = [(i, fake.word().capitalize()) for i in range(1, 21)]
locations = [(i, fake.city()) for i in range(1, 51)]
job_titles = [(i, fake.job()) for i in range(1, 101)]
genders = [(1, "Male"), (2, "Female"), (3, "Other")]

# Create DataFrame
data = {
    "Employee_ID": employee_ids,
    "First_Name": [fake.first_name() for _ in range(num_records)],
    "Last_Name": [fake.last_name() for _ in range(num_records)],
    "DOB": [fake.date_of_birth(minimum_age=18, maximum_age=65) for _ in range(num_records)],
    "Hire_Date": [fake.date_between(start_date='-20y', end_date='-1y') for _ in range(num_records)],
    "Termination_Date": [fake.date_between(start_date='-1y', end_date='today') if random.random() < 0.2 else None for _ in range(num_records)],
    "Manager_ID": manager_ids,
    "Department_ID": [random.choice(departments)[0] for _ in range(num_records)],
    "Department": [dict(departments)[dept_id] for dept_id in [random.choice(departments)[0] for _ in range(num_records)]],
    "Location_ID": [random.choice(locations)[0] for _ in range(num_records)],
    "Location": [dict(locations)[loc_id] for loc_id in [random.choice(locations)[0] for _ in range(num_records)]],
    "Job_Title_ID": [random.choice(job_titles)[0] for _ in range(num_records)],
    "Job_Title": [dict(job_titles)[job_id] for job_id in [random.choice(job_titles)[0] for _ in range(num_records)]],
    "Gender_ID": [random.choice(genders)[0] for _ in range(num_records)],
    "Gender": [dict(genders)[gid] for gid in [random.choice(genders)[0] for _ in range(num_records)]],
    "Salary": [round(random.uniform(30000, 150000), 2) for _ in range(num_records)],
}

# Fill remaining columns up to 100
current_cols = len(data.keys())
extra_needed = num_columns - current_cols
for i in range(extra_needed):
    col_name_id = f"Extra_Field_{i+1}_ID"
    col_name_val = f"Extra_Field_{i+1}"
    unique_ids = list(range(1, 21))
    names = [fake.word().capitalize() for _ in range(20)]
    mapping = dict(zip(unique_ids, names))
    chosen_ids = [random.choice(unique_ids) for _ in range(num_records)]
    data[col_name_id] = chosen_ids
    data[col_name_val] = [mapping[x] for x in chosen_ids]

# Create DataFrame
df = pd.DataFrame(data)

# Save to CSV
df.to_csv("HR_Dataset.csv", index=False)

print("✅ HR_Dataset.csv created with", num_records, "records and", len(df.columns), "columns.")