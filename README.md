
# GCP Uber Data Pipeline Project

This project demonstrates an end-to-end Google Cloud Platform (GCP) data pipeline for analyzing Uber data and visualizing it in a Power BI dashboard.

---

## Resources and Tools Used

- **Raw Data:** Acquired from open source portals
- **Draw.io:** Used for dimensional modeling
- **Google Cloud Storage:** To store the raw data
- **Google Virtual Machine:** For compute efficiency
- **Mage AI:** To create and orchestrate the data pipeline
- **Google BigQuery:** For querying and managing the data warehouse
- **Power BI:** For visualization of insights from BigQuery

---

## Data Exploration

Sample columns from the Uber dataset:

| VendorID | tpep_pickup_datetime | tpep_dropoff_datetime | passenger_count | trip_distance | pickup_longitude | pickup_latitude | RatecodelD | store_and_fwd_flag |
|----------|---------------------|----------------------|-----------------|---------------|------------------|-----------------|------------|--------------------|
| 1        | 2016-03-01          | 2016-03-01 00:07:55  | 1               | 2.50          | -73.976746       | 40.765152       | 1          | N                  |
| 1        | 2016-03-01          | 2016-03-01 00:11:06  | 1               | 2.90          | -73.983482       | 40.767925       | 1          | N                  |
| 2        | 2016-03-01          | 2016-03-01 00:31:06  | 2               | 19.98         | -73.782021       | 40.644810       | 1          | N                  |
| 2        | 2016-03-01          | 2016-03-01 00:00:00  | 3               | 10.78         | -73.863419       | 40.769814       | 1          | N                  |
| 2        | 2016-03-01          | 2016-03-01 00:00:00  | 5               | 30.43         | -73.971741       | 40.792183       | 3          | N                  |

---

## Dimensional Model (Star Schema)

- **Fact Table**
- **Dimension Tables:**
  - Pickup location
  - Dropoff location
  - Passenger count
  - Trip distance
  - Date/time
  - Rate code type
  - Payment type

---

## Pipeline Steps

### 1. GCP Project and Storage Setup

- Create a new GCP project.
- Create a Google Cloud Storage bucket, enable public access, and upload the Uber CSV data.

### 2. Compute Engine Setup

- Create a Google Compute Engine VM instance.
- Allow HTTP/HTTPS inbound traffic.
- Open TCP port 6789 for Mage AI.

### 3. Install Mage AI and Dependencies

SSH into your VM and run:

```

sudo apt-get update
sudo apt-get install python3-distutils
sudo apt-get install python3-apt
sudo apt-get install wget
wget https://bootstrap.pypa.io/get-pip.py
sudo python3 get-pip.py
sudo pip3 install mage-ai
sudo pip3 install pandas
sudo pip3 install google-cloud
sudo pip3 install google-cloud-bigquery

```

Start Mage AI:

```

mage start <project_name>

```

Access Mage UI at `http://<VM_EXTERNAL_IP>:6789`.

### 4. Data Loading and Transformation in Mage AI

- Create a new batch process pipeline in Mage UI.
- Use the Data Loader (Python/API) to load data from your GCS bucket (public URL).
- Apply transformation code to create fact and dimension tables.
- Example transformation code snippet:

```

return {
"datetime_dim": datetime_dim.to_dict(orient="dict"),
"passenger_count_dim": passenger_count_dim.to_dict(orient="dict"),
"trip_distance_dim": trip_distance_dim.to_dict(orient="dict"),
"rate_code_dim": rate_code_dim.to_dict(orient="dict"),
"pickup_location_dim": pickup_location_dim.to_dict(orient="dict"),
"dropoff_location_dim": dropoff_location_dim.to_dict(orient="dict"),
"payment_type_dim": payment_type_dim.to_dict(orient="dict"),
"fact_table": fact_table.to_dict(orient="dict")
}

```

- For detailed transformation code, see the project's GitHub notebook.

### 5. Export to BigQuery

- Configure Mage AI to export tables to BigQuery.
- Set up GCP service account credentials in Mage AI (provide in YAML config).
- Example exporter code snippet:

```

from mage_ai.settings.repo import get_repo_path
from mage_ai.io.bigquery import BigQuery
from pandas import DataFrame
from os import path

if 'data_exporter' not in globals():
from mage_ai.data_preparation.decorators import data_exporter

@data_exporter
def export_data_to_big_query(data: DataFrame, **kwargs):
config_path = path.join(get_repo_path(), 'io_config.yaml')
config_profile = 'default'
for key, value in data.items():
table_id = f'project_id.dataset_id.{key}'
BigQuery.with_config(config_path, config_profile).export(
DataFrame(value),
table_id,
)

```

- Each table (dimension and fact) is exported to BigQuery.

---

## Analytics and Visualization

- Run SQL queries in BigQuery to analyze the data and generate insights.
- Example queries:
  - Total trips per day
  - Average trip distance by pickup location
  - Distribution of payment types
  - Peak hours for pickups and drop-offs

- Connect Power BI to BigQuery using the native connector.
- Build dashboards to visualize insights from the Uber data.

---

## References



---

**End of README**