This directory contains helper SELECT statements to query BigQuery audit logs \
More information regarding each is detailed below:


### [bigquery_audit_log_v2.sql](/views/audit/bigquery_audit_log_v2.sql)

BigQuery SELECT statement to help you extract and format BigQueryMetaData
events.

#### Prerequisites

[BigQueryAuditMetadata](https://cloud.google.com/bigquery/docs/reference/auditlogs/rest/Shared.Types/BigQueryAuditMetadata)

1.  Define a BigQuery log sink using any of the following methods:
    *   [gcloud command](https://cloud.google.com/bigquery/docs/reference/auditlogs#defining_a_bigquery_log_sink_using_gcloud)
        ```
        gcloud alpha logging sinks create my-example-sink \ 
        bigquery.googleapis.com/projects/my-project-id/datasets/auditlog_dataset \
        --log-filter='protoPayload.metadata."@type"="type.googleapis.com/google.cloud.audit.BigQueryAuditMetadata"' \ 
        --use-partitioned-tables
        ``` 
        Note: gcloud **alpha** is needed in order to use the parameter `--use-partitioned-tables` 
    *   [Cloud Console Logs Viewer](https://cloud.google.com/logging/docs/export/configure_export_v2#dest-create)
        *   Make sure to select
            [partitioning](https://cloud.google.com/logging/docs/export/bigquery#partition-tables)
            for your BigQuery destination
            
    Note: You can create a log sink at the folder, billing account, or organization level using an 
    [aggregated sink](https://cloud.google.com/logging/docs/export/aggregated_sinks#creating_an_aggregated_sink).
1.  The log sink will immediately create the BigQuery dataset but the table will
    be created once you run a BigQuery job post log sink creation.
1.  To use the SELECT statement in
    [bigquery_audit_log_v2.sql](/views/audit/bigquery_audit_log_v2.sql), change
    all occurrences of
    `project_id.dataset_id.cloudaudit_googleapis_com_data_access` to be the full
    table path you created in step 1.
    *   `sed
        's/project_id.dataset_id.cloudaudit_googleapis_com_data_access/YOUR_PROJECT.YOUR_DATASET.YOUR_TABLE/'
        bigquery_audit_log_v2.sql`
1.  From here, you can do further analysis in BigQuery by querying the view, or
    you can connect it to a BI tool such as DataStudio as a data source and
    build dashboards.
    
#### Usage Examples

* Retrieve All SELECT SQL Queries Executed 
  ```  
  SELECT * EXCEPT(
    modelDeletion,
    modelCreation,
    modelMetadataChange,
    routineDeletion,
    routineCreation,
    routineChange)
  FROM `project_id.dataset_id.cloudaudit_googleapis_com_data_access`
  WHERE 
    hasJobChangeEvent
    AND jobChange.jobConfig.queryConfig.statementType = 'SELECT'
  ``` 
  
* Retrieve All DML SQL Queries Executed 
  ```  
  SELECT * EXCEPT(
    modelDeletion,
    modelCreation,
    modelMetadataChange,
    routineDeletion,
    routineCreation,
    routineChange)
  FROM `project_id.dataset_id.cloudaudit_googleapis_com_data_access`
  WHERE 
    hasJobChangeEvent
    AND jobChange.jobConfig.queryConfig.statementType IN (
          'INSERT', 'DELETE', 'UPDATE', 'MERGE')
  ``` 
  
* Retrieve All BigQuery Load Jobs
  ```
  SELECT * EXCEPT(
    modelDeletion,
    modelCreation,
    modelMetadataChange,
    routineDeletion,
    routineCreation,
    routineChange)
  FROM `project_id.dataset_id.cloudaudit_googleapis_com_data_access`
  WHERE 
    hasJobChangeEvent AND 
    jobChange.jobConfig.loadConfig.destinationTable IS NOT NULL
  ```
  
* Retrieve All BigQuery Table Deletion Events
  ```
  SELECT * EXCEPT(
    modelDeletion,
    modelCreation,
    modelMetadataChange,
    routineDeletion,
    routineCreation,
    routineChange)
  FROM `project_id.dataset_id.cloudaudit_googleapis_com_data_access`
  WHERE 
    hasTableDeletionEvent
  ```

### [bigquery_audit_log_v1.sql](/views/audit/bigquery_audit_log_v1.sql) (old version of logs)

BigQuery SELECT statement to help you extract and format the legacy [(Audit Data)](https://cloud.google.com/bigquery/docs/reference/auditlogs/rest/Shared.Types/AuditData)
events. You'll want to use the [bigquery_audit_log_v2.sql](/views/audit/bigquery_audit_log_v2.sql) SELECT
statement as it reads the newer and more detailed [BigQueryAuditMetadata](https://cloud.google.com/bigquery/docs/reference/auditlogs/rest/Shared.Types/BigQueryAuditMetadata)
events (e.g. which tables were read/written by a given query job or which tables expired due to having an expiration time configured).
