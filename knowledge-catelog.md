## Enable the Dataplex API


1. In Cloud Shell, run the following command to enable the Dataplex API.
```
gcloud services enable \
  dataplex.googleapis.com
```

2. Create a variable for project ID:
```
export PROJECT_ID=$(gcloud config get-value project)
```

3. Create a variable for region:
```
export REGION=us-west1
gcloud config set compute/region $REGION
```

## Task 1. Create a lake
1. create a new lake called Ecommerce:
```
gcloud dataplex lakes create <name>\
  --location=$REGION \
  --display-name="<Name of the lake>"
  --description="<Description>"
```
```
gcloud dataplex lakes create ecommerce \
   --location=$REGION \
   --display-name="Ecommerce" \
   --description="Ecommerce Domain"
```
- It can take up to 3 minutes for the lake to be created.
- If you receive an error of Status code: 403. Permission 'dataplex.lakes.create' denied, wait a few minutes and try again. The permissions may need more time to be granted after enabling the Dataplex API in the previous Setup section.

The final output should look similar to the following:

``` Created [ecommerce] Lake created in [projects/$PROJECT_ID/locations/$REGION].```

## Task 2. Add a zone to your lake
After you create a lake, you can add zones to the lake. Zones are subdomains within a lake that you can use to categorize data further. For example, you can categorize data by stage, usage, or restrictions.

There are two types of zones:

- Raw zones contain data in raw formats (such as files in Cloud Storage buckets) and are not subject to strict type-checking.
- Curated zones contain data that is cleaned, formatted, and ready for analytics such as BigQuery datasets.
**In this task, you use the command line to create a curated zone for working with BigQuery datasets.**

- Create a new curated zone called Orders Curated Zone with metadata discovery enabled:
```
gcloud dataplex zones create orders-curated-zone \
    --location=$REGION \
    --lake=ecommerce \
    --display-name="Orders Curated Zone" \
    --resource-location-type=SINGLE_REGION \
    --type=CURATED \
    --discovery-enabled \
    --discovery-schedule="0 * * * *"
```
- It can take up to 2 minutes for the zone to be created.
- The final output should look similar to the following:

```Created [orders-curated-zone] Zone created in [projects/$PROJECT_ID/locations/$REGION/lakes/ecommerce].```

## Task 3. Attach an asset to a zone

### 1. Create a BigQuery dataset
- In Cloud Shell, run the following command to create a new BigQuery dataset called **orders**:
```
bq mk --location=$REGION --dataset orders
```

### 2. Attach the BigQuery dataset to the zone
```
gcloud dataplex assets create orders-curated-dataset \
--location=$REGION \
--lake=ecommerce \
--zone=orders-curated-zone \
--display-name="Orders Curated Dataset" \
--resource-type=BIGQUERY_DATASET \
--resource-name=projects/$PROJECT_ID/datasets/orders \
--discovery-enabled
```
- It can take up to 2 minutes for the asset to be created.


## Task 4. Delete assets, zones, and lakes
`To delete a lake, you must first detach assets and then delete the zones.`
- Detach an asset
```
gcloud dataplex assets delete orders-curated-dataset
--location=$REGION --zone=orders-curated-zone --lake=ecommerce

Deleted asset [orders-curated-dataset].
```

- Delete a zone
```
gcloud dataplex zones delete orders-curated-zone \
  --location=$REGION \
  --lake=ecommerce
```

- Delete the lake
```
gcloud dataplex lakes delete ecommerce \
 --location=$REGION
```



