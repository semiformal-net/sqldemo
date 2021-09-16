0. `export PROJECT_ID=$(gcloud config get-value project)`
0. `export ROOTPW=$(pwgen 12 1)`
0. `export USERPW=$(pwgen 12 1)`
1. ```gcloud sql instances create sqldemo \
--tier db-f1-micro \
--region=us-central1``` --- and note the connection string, database user, and database password that you create.
2. `gcloud sql databases create demo --instance=sqldemo`
2. ```gcloud sql users set-password root \
--host=% \
--instance sqldemo \
--password ${ROOTPW} && echo password ${ROOTPW}
```
2. ```
gcloud sql users create demo \
--host=% \
--instance=sqldemo \
--password=${USERPW} && echo password ${USERPW}
```
2. ```
gcloud sql databases create demo \
--instance=sqldemo
```
3. ```
gcloud iam service-accounts create sqldemosrvc \
    --description="sql demo service account" \
    --display-name="sqldemosrvc"
    ```
4. ```
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member="serviceAccount:sqldemosrvc@${PROJECT_ID}.iam.gserviceaccount.com" \
    --role="roles/cloudsql.client"
    ```
5. `gcloud iam service-accounts keys create /tmp/sqldemosrvc.json --iam-account=sqldemosrvc@${PROJECT_ID}.iam.gserviceaccount.com`
6. `export GOOGLE_APPLICATION_CREDENTIALS=/tmp/sqldemosrvc.json`
7. Install the [cloud sql proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy#install) to `/opt/google`
8.
```
export DB_HOST=127.0.0.1:3306
export DB_USER=demo
export DB_PASS=${USERPW}
export DB_NAME=demo
```
9. `/opt/google/cloud_sql_proxy -instances=${PROJECT_ID}:us-central1:sqldemo=tcp:3306 -credential_file=$GOOGLE_APPLICATION_CREDENTIALS `
10. Now try `mysql -h 127.0.0.1 -P 3306 -D demo -u demo -p`
11. run `python main.py`
