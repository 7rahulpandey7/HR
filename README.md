import pandas as pd
import numpy as np
from faker import Faker
import random

# Record count and faker setup
num_records = 500_000
fake = Faker()

# Set seed for reproducibility
Faker.seed(42)
np.random.seed(42)
random.seed(42)

# Generate Employee IDs
employee_ids = np.arange(1, num_records + 1)

# Generate Departments
departments = [(i, fake.job()) for i in range(1, 21)]  # 20 departments
department_ids, department_names = zip(*departments)

# Generate Locations
locations = [(i, fake.city()) for i in range(1, 51)]  # 50 locations
location_ids, location_names = zip(*locations)

# Assign random department/location
dept_id_col = np.random.choice(department_ids, num_records)
dept_name_col = [department_names[d-1] for d in dept_id_col]
loc_id_col = np.random.choice(location_ids, num_records)
loc_name_col = [location_names[l-1] for l in loc_id_col]

# Manager IDs — random from employee list, but some will be NaN (top level)
manager_ids = np.random.choice(employee_ids, num_records)
# Ensure no one is their own manager
manager_ids = np.where(manager_ids == employee_ids, np.nan, manager_ids)

# Dates
hire_dates = [fake.date_between(start_date="-20y", end_date="today") for _ in range(num_records)]
termination_dates = [
    fake.date_between(start_date=hire, end_date="today") if random.random() < 0.2 else None
    for hire in hire_dates
]
birth_dates = [fake.date_of_birth(minimum_age=20, maximum_age=65) for _ in range(num_records)]

# Generate other random columns
data = {
    "EmployeeID": employee_ids,
    "FirstName": [fake.first_name() for _ in range(num_records)],
    "LastName": [fake.last_name() for _ in range(num_records)],
    "Email": [fake.email() for _ in range(num_records)],
    "PhoneNumber": [fake.phone_number() for _ in range(num_records)],
    "DepartmentID": dept_id_col,
    "DepartmentName": dept_name_col,
    "LocationID": loc_id_col,
    "LocationName": loc_name_col,
    "ManagerID": manager_ids,
    "HireDate": hire_dates,
    "TerminationDate": termination_dates,
    "BirthDate": birth_dates,
    "Salary": np.random.randint(30000, 200000, num_records),
    "Bonus": np.random.randint(0, 50000, num_records),
    "Gender": np.random.choice(["Male", "Female", "Other"], num_records),
    "MaritalStatus": np.random.choice(["Single", "Married", "Divorced"], num_records),
    "JobTitle": [fake.job() for _ in range(num_records)],
}

# Add dummy extra columns to reach 100 columns
for i in range(len(data), 100):
    data[f"CustomCol_{i}"] = [fake.word() for _ in range(num_records)]

# Create DataFrame
df = pd.DataFrame(data)

# Save to CSV
df.to_csv("HR_Dataset.csv", index=False)

print("Dataset generated: HR_Dataset.csv with", df.shape[0], "records and", df.shape[1], "columns")