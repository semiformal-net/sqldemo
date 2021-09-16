# Running

`export PROJECT_ID=$(gcloud config get-value project)`

`export ROOTPW=$(pwgen 12 1)`

`export USERPW=$(pwgen 12 1)`

```gcloud sql instances create sqldemo \
--tier db-f1-micro \
--region=us-central1``` --- and note the connection string, database user, and database password that you create.

`gcloud sql databases create demo --instance=sqldemo`

```gcloud sql users set-password root \
--host=% \
--instance sqldemo \
--password ${ROOTPW} && echo password ${ROOTPW}
```

```
gcloud sql users create demo \
--host=% \
--instance=sqldemo \
--password=${USERPW} && echo password ${USERPW}
```

```
gcloud sql databases create demo \
--instance=sqldemo
```

```
gcloud iam service-accounts create sqldemosrvc \
    --description="sql demo service account" \
    --display-name="sqldemosrvc"
```

```
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member="serviceAccount:sqldemosrvc@${PROJECT_ID}.iam.gserviceaccount.com" \
    --role="roles/cloudsql.client"
```

`gcloud iam service-accounts keys create /tmp/sqldemosrvc.json --iam-account=sqldemosrvc@${PROJECT_ID}.iam.gserviceaccount.com`

`export GOOGLE_APPLICATION_CREDENTIALS=/tmp/sqldemosrvc.json`

Install the [cloud sql proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy#install) to `/opt/google`

```
export DB_HOST=127.0.0.1:3306
export DB_USER=demo
export DB_PASS=${USERPW}
export DB_NAME=demo
```

`/opt/google/cloud_sql_proxy -instances=${PROJECT_ID}:us-central1:sqldemo=tcp:3306 -credential_file=$GOOGLE_APPLICATION_CREDENTIALS `

Now try `mysql -h 127.0.0.1 -P 3306 -D demo -u demo -p`

run `python main.py`
