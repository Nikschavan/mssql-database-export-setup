### Setup MSSQL database export from Umbraco on local

1. Start the database server locally using command
`docker compose up -d`

2. Drop the database dumps received in the directory `database_dumps`
These database dumps will be available inside the docker environment under `/var/opt/mssql/backup/`

3. Extract the databse.

```
docker exec -it sql-server-db /opt/mssql-tools/bin/sqlcmd -S localhost \
   -U SA -P 'Strong.Password.01' \
   -Q 'RESTORE FILELISTONLY FROM DISK = "/var/opt/mssql/backup/<.bak-file-name>"' \
   | tr -s ' ' | cut -d ' ' -f 1-2
```

In the above command replace the `<.bak-file-name>` with the name of the database dump file.
The output will be something like this

```
LogicalName PhysicalName
---------- ------------
data-name /var/opt/mssql/data/data-file.mdf
log-name /var/opt/mssql/data/log-file.mdf
---------- ------------
```

4. Restore the database

```
docker exec -it sql-server-db /opt/mssql-tools/bin/sqlcmd \
   -S localhost -U SA -P 'Strong.Password.01' \
   -Q 'RESTORE DATABASE <database-name> FROM DISK = "/var/opt/mssql/backup/<.bak-file-name>" WITH MOVE "<data-name>" TO "/var/opt/mssql/data/elle/<data-file.mdf>", MOVE "<log-name>" TO "/var/opt/mssql/data/elle/<log-file>.ldf"'
```

In the above command replace the `<database-name>` with the name of the database to be restored to.
Replace the `<.bak-file-name>` with the name of the database dump file.
Replace the `<data-name>` with the name of the data name, This is the output in place of <data-name> in the previous command.
Replace the `<log-name>` with the name of the log name, This is the output in place of <log-name> in the previous command.
Replace the `<data-file.mdf>` with the name of the data file, This is the output in place of <data-file.mdf> in the previous command.
Replace the `<log-file>.ldf` with the name of the log file, This is the output in place of <log-file>.ldf in the previous command.

5. Use MSSQL database client to connect to the database server and download required database tables as JSON.
Table Plus on Mac is an example client which has been tested in this approach to connect to the database and generate JSON files.

In tableplus for Mac, connect to the database using the following details

```
Host: localhost
Port: 1433
Username: SA
Password: Strong.Password.01
```

From databases select the database name which was selected when restoring the database.

#### Important tables

1. cmsContentXml

This databse table consists of all the content in the website in the XML format. Export this table as JSON

2. icUrlTracker

This table consists of all the redirects in the website. Export this table as JSON

3. cmsContentType

This defines the content type of the content in the website

4. cmsContent

This table contains relationship of all the content in cmsContentXML as well as cmsContentType.
This table can be helpful to only import certain type of content type at a time.

--------------------------

Note - Microsoft documentation for setting up and restoring the database backup is available here.
<https://learn.microsoft.com/en-us/sql/linux/tutorial-restore-backup-in-sql-server-container?view=sql-server-ver16>
