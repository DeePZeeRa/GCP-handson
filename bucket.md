# Set the region first
```
gcloud config set compute/region us-central1
```

# Create a new bucket
```
gcloud storage buckets create "gs://$BUCKET"
```

# Task 2. Define a Retention Policy
You can define the Retention Policy using seconds, days, months, and years with the Cloud Storage gcloud storage tool. As an example, create a Retention Policy for 10 seconds:
```
gcloud storage buckets update "gs://$BUCKET" --retention-period=10s
```

- Verify the Retention Policy for a bucket:
```
gcloud storage buckets describe "gs://$BUCKET" --format="default(retention_policy)"
```
```
retention_policy:
  effectiveTime: '2026-09-16T08:13:06.882000+00:00'
  retentionPeriod: '10'
```

- Now that the bucket has a Retention Policy, add a transaction record object to test it:
```
gcloud storage cp gs://spls/gsp297/dummy_transactions "gs://$BUCKET/"
```

- Review the retention expiration:\
```
gcloud storage ls -L "gs://$BUCKET/dummy_transactions"
```

```
gs://qwiklabs-gcp-02-d9f709adde6c/dummy_transactions:
  Creation Time:               2026-10-24T17:49:11Z
  Update Time:                 2026-10-24T17:49:11Z
  Storage Class Update Time:   2026-10-24T17:49:11Z
  Storage Class:               STANDARD
  Retention Expiration:        2026-10-24T17:49:21Z
  Content-Language:            en
  Content-Length:              45
  Content-Type:                text/plain
  Hash (CRC32C):               4PdEyA==
  Hash (MD5):                  loAwRzgwWU+JCSjGGildQQ==
  ETag:                        CIKQ9Lab05cDEAE=
  Generation:                  1792864151209986
  Metageneration:              1
  ACL:
  [
    {
      "entity": "project-owners-1053740930681",
      "projectTeam": {
        "projectNumber": "1053740930681",
        "team": "owners"
      },
      "role": "OWNER"
    },
    {
      "entity": "project-editors-1053740930681",
      "projectTeam": {
        "projectNumber": "1053740930681",
        "team": "editors"
      },
      "role": "OWNER"
    },
    {
      "entity": "project-viewers-1053740930681",
      "projectTeam": {
        "projectNumber": "1053740930681",
        "team": "viewers"
      },
      "role": "READER"
    },
    {
      "email": "student-04-bca9a332901b@qwiklabs.net",
      "entity": "user-student-04-bca9a332901b@qwiklabs.net",
      "role": "OWNER"
    }
  ]
TOTAL: 1 objects, 45 bytes (45.00B)
```

# Task 3. Lock the Retention Policy
- Lock the Retention Policy:
```
gcloud storage buckets update "gs://$BUCKET" --lock-retention-period
```
```
This will PERMANENTLY set the Retention Policy on gs://YOUR-BUCKET-NAME to:

{'effectiveTime': '2026-09-15T15:34:08.028000+00:00', 'retentionPeriod': '10'}

This setting cannot be reverted!  Continue? [y|N]:
```


# Task 4. Temporary hold
Continuing the example above, consider the bucket already configured with a 10 second locked Retention Policy.

Financial regulators decide to perform an audit of one of the branch's customers, and require that those records are retained for the duration of the audit. Some of the Cloud Storage objects for this customer are close to their expiration date, and will soon be automatically deleted.

To handle this, when regulatory investigation begins, the Branch IT Administrator sets the temporary hold flag for each of the objects related to the audit. While that flag is set, the objects will continue to be protected from deletion, even if they are older than 10 seconds.

- Set a temporary hold on the transactions object:
```
gcloud storage objects update "gs://$BUCKET/dummy_transactions" --temporary-hold
```

By placing a temporary hold on the object, delete operations are not possible unless the object is released from the hold. As an example, attempt to delete the object:
```
gcloud storage rm "gs://$BUCKET/dummy_transactions"
```
**You should see the following error message:**
```ERROR: HTTPError 403: Object 'YOUR-BUCKET-NAME/dummy_transactions' is under active Temporary hold and cannot be deleted, overwritten or archived until hold is removed.```
Once regulators conclude their audit, the Branch IT Administrator removes the temporary hold. Use the following command to release the hold:
```
gcloud storage objects update "gs://$BUCKET/dummy_transactions" --no-temporary-hold
```
Now you can delete the file unless the Retention Policy for the file hasn't expired. Otherwise wait a few moments and try again.
```
gcloud storage rm "gs://$BUCKET/dummy_transactions"
```



| Task             | Console Location                    |
|------------------|-------------------------------------|
| Create Bucket    | Cloud Storage → Buckets → Create    |
| Set Retention    | Bucket → Protection Tab             |
| Lock Policy      | Bucket → Protection Tab → Lock      |
| Temporary Hold   | Object → Edit Metadata              |
| Event-Based Hold | Bucket Protection / Object Metadata |
| Delete Object    | Bucket → Objects                    |
| Delete Bucket    | Cloud Storage → Buckets             |


# Using Cloud Storage CLI to Perform Operations on Buckets and Objects
What you'll learn to do
- Create a bucket
- Copy files from a local folder to a bucket
- Synchronize the contents of the local folder with the contents of the bucket
- Change access control permissions on objects
- Delete a bucket.

### Task 0 : download the sample asset files
```
gcloud storage cp -r gs://qwiklabs-gcp-03-41b3cd18f779-assets/* .
```

### Task 1. Working with buckets and objects
```
PROJECT_ID=`gcloud config get-value project`
BUCKET=${PROJECT_ID}-bucket
```
```
Your active configuration is: [cloudshell-3825]
```

### Task 2. Create a bucket
- Create a bucket and multi-regional storage class:
```
gcloud storage buckets create --default-storage-class=MULTI_REGIONAL gs://${BUCKET}
```

### Task 3. Upload objects to your bucket
- Run the following to copy the `endpointslambda` object to your bucket:
```gcloud storage cp -r endpointslambda gs://${BUCKET}```
- In gcloud storage, multi-threaded and multi-processing parallel transfers are enabled automatically for high performance. The -r option allows recursion through directories.

### Task 4. List objects
```
gcloud storage ls gs://${BUCKET}/*
```

### Task 5. Sync changes with bucket
Use the following commands to rename and delete some files:
```
mv endpointslambda/Apache2_0License.txt endpointslambda/old.txt
```
```
rm endpointslambda/aeflex-endpoints/app.yaml
```
Now synchronize the local changes with the bucket:
```
gcloud storage rsync --delete-unmatched-destination-objects -r endpointslambda gs://${BUCKET}/endpointslambda
```
In this command, the `--delete-unmatched-destination-objects` option deletes files from the target if they're missing in the source (in this case, it deletes app.yaml from the bucket). The -r option runs the command recursively on directories.
- To verify that the bucket is now in sync with your local changes, list the files in the bucket again:
```gcloud storage ls gs://${BUCKET}/*```

### Task 6. Make objects public
- To allow public access to all files under the endpointslambda folder in your bucket, execute the following command:
```
gcloud storage objects update --predefined-acl=publicRead -r gs://${BUCKET}
```

### Task 7. Copy with different storage class
Next, copy a file with Nearline storage class instead of the bucket's default Multi-regional storage class:
```
gcloud storage cp --storage-class=NEARLINE ghcn/ghcn_on_bq.ipynb gs://${BUCKET}
```

### Task 8. Check storage classes
- Run the following to check the storage classes and view other detailed information about the objects in your bucket:
```
gcloud storage ls -L -r gs://${BUCKET} | more
```

### Task 9. Delete your bucket
- Before deleting a bucket, you must first delete all objects in the bucket. To delete all objects, execute the following command:
```gcloud storage rm -r gs://${BUCKET}/*```
Now delete the bucket:
```
gcloud storage buckets delete gs://${BUCKET}
```
