# Notes for DP-600 Exam

## Links

- [Microsoft Practice Assessment](https://learn.microsoft.com/en-us/credentials/certifications/exams/dp-600/practice/assessment?assessment-type=practice&assessmentId=90)  
- [DP-600 Exam Preparation on YouTube](https://www.youtube.com/playlist?list=PLug2zSFKZmV05ZJcmHemXxyJjPVXeQ2qS)

## Pyspark code

- Display top 100 rows from a dataframe: `display(df.limit(100))`
- `df.describe().show()` returns MEAN, COUNT, STD, MIN, MAX
- Return rows in dataframe with blank columns: `df_customers[df_customers.isnull().any(axis=1)]`
- To profile data in notebook: Display dataframe using `display(df)` then click Inspect
- Use matplotlib to plot a bar chart with x-axis SalesTerritory and y-axis CityCount:
  
    ```python
    plt.bar(x=data['SalesTerritory'], height=data['CityCount'])
    plt.xlabel('SalesTerritory')
    plt.ylabel('Cities') plt.show()
    ```  

- Directory for data is stored in `'Files/data/file_name'`
- File API path is in the format: `lakehouse/default/Files/Customers.parquet`
- read data and specify schema
  
    ```python
    productSchema = StructType([
        StructField("ProductID", IntegerType()),
        StructField("ProductName", StringType()),
        StructField("Category", StringType()),
        StructField("ListPrice", FloatType())
        ])

    df = spark.read.load('Files/data/product-data.csv',
        format='csv',
        schema=productSchema,
        header=False)
    display(df.limit(10))
    ```

- select columns and filter columns using where: `df.select("ProductName", "Category", "ListPrice").where((df["Category"]=="Mountain Bikes") | (df["Category"]=="Road Bikes"))`
- `df.withColumn('colName', formula)` to add new columns to dataframe
- filter columns: `filtered_df = df.filter(raw_df["storeAndFwdFlag"].isNotNull())`
- transform columns: `df.withColumn("age, col("age").cast("int")).show()` col = select column. cast = change column type.
- Write df to table: `df.write.mode("overwrite").format("delta").saveAsTable("files/sales")`
- Add new columns to existing table and append data: `df.write.format("delta").mode("append").option("overwriteSchema", "false").saveAsTable("table")`
- Write to a delta table: `df.write.mode("overwrite").format("delta").save(f"Tables/{table_name}")`
- Write to a parquet file: `df.write.mode("overwrite").parquet(file_path)`
- Create temp SQL table from delta table: `df.createOrReplaceTempView("yellow_taxi_opt")`

## SQL

- too simple queries to write notes
- ROW_NUMBER() returns sequential numbers, does not consider ties. Rows with same value will have different row number.
- DENSE_RANK() function returns the rank of each row within the result set partition, with no gaps in the ranking values. The RANK() function includes gaps in the ranking.
- When manually creating/updating statistics for optimizing query performance, you should focus on columns used in JOIN, ORDER BY, and GROUP BY clauses.
- Adding primary key:  
  
    ```sql
    ALTER TABLE dbo.Dim_Customer add CONSTRAINT PK_Dim_Customer PRIMARY KEY NONCLUSTERED (CustomerKey) NOT ENFORCED
    ```

    PRIMARY KEY is only supported when NONCLUSTERED and NOT ENFORCED are both used.
  
## Data engineering

- Data pipeline copy data activity = ADF copying data. Low-code. Use this for ingesting external data into lakehouse. Fastest way.
- Data pipeline can call dataflow, run stored procedure, copy data, run notebook. Use data pipeline to schedule activities.
- Dataflow Gen2 = use power query for data transformation. M language is used.
- Dataflows can only output into a data table or warehouse. JSON files is not supported.
- Maximum number of dataflow refreshes a day is 48.
- Cannot use notebooks to write data into a data warehouse table.
- In data pipeline (ADF), only notebooks and SQL stored procs can support parameterization (can pass parameters to it)
- For one time data load of small files, use Lakehouse explorer and manually upload
- Query Editor in Warehouse = visual way of writing SQL statements
- How to reduce the size of queries sent to Azure SQL in Fabric pipelines: Add a deployment parameter rule to filter the data.
- Combine data from 2 different lakehouses like Azure Databricks: use Fabric lakehouse
- Query data across 2 different fabric warehouses: Use cross-database querying
- Shortcuts in Lakehouse to table in another lakehouse: when setting up, you can select specific partition folders to enable access to. This is to limit access to specific data
- A shortcut to ADLS has a connection string like: `https://contoso.dfs.core.windows.net`
- Partitioning by column creates files for each value in that column. e.g. each customer ID will have its own file.
- Slowly changing dimension(SCD)
  - Type 1: update existing record and update the modified date column. Historical record not saved.
  - Type 2: there are columns with the valid date range: start date and end date, and a column to indicate the current record. Updates are added as new rows to table, existing rows are not modified.
  - Type 3: there is a column that contains the previous value of the dimension to track the history e.g. CurrentEmail and OriginalEmail. Can be used in combination with Type 1 or Type 2. Adds new columns to track changes.
- To improve query performance and reduce storage costs. Reduce having too many parquet files: Select Maintenance and run the OPTIMIZE command as well as the VACUUM command with a retention policy of seven days. Optimize consolidates multiple small parquet files into large file. Vaccum removes old files no longer referenced by Delta table log.
- You notice that access to one of the data sources is restricted to narrow time windows.:
Create a staging dataflow that will only copy the data from the source as-is. Extracct the data into staging first, then transform later.
- RANKX: The DAX measure ranks the product names by Sales, with the largest values getting the smallest (e.g. 1,2,3) ranks, and when product names have tied values, then the next rank value, after a tie, is the rank value of the tie plus the count of tied values
- Tools for execution: A schedule for Orchestration pipeline, pipeline copy activity for Bronze layer, pipeline Dataflow activity for Silver layer, pipeline Stored procedure for Gold layer

## Power BI

- Check valid records in Power Query: use column quality
- XMLA endpoint: a way to connect to Power BI workspaces and datasets from external apps. The main limitation using this is that PBIX files cannot be downloaded.
- Need to support near-real-time reporting for large datasets in PBI: use Direct Lake semantic model storage mode. Better performance than DirectQuery.
- Refreshing large datasets bigger than 10GB: use Large semantic model storage format to enable refresh. This will use the fabric capacity size
- Use Tabular Editor to add calculation groups to lakehouse semantic model.
- Allow users to change y-axis of bar chart using slicer: use field parameters
- To enable incremental refresh: configure RangeStart and RangeEnd
- User-defined aggregations can improve query performance for DirectQuery semantic models. To check whether queries are using in-memory cache or coming from data source by DirectQuery: use SQL Server Profiler or DAX Studio
- User-defined aggregations: the fact table must be in DirectQuery mode. It is recommended to set the storage mode to Import for aggregated tables because of the performance, while dimension tables should be set to Dual mode to avoid the limitations of limited relationships.
- Feature to capture queries and troubletshoot in DAX Studio: All Queries trace
- Use Best Practice Analyzer tool within Tabular Editor to assess your DAX performance. Level 3 and above severity code indicates an error.
- Fastest refresh is every 30 minutes
- To update measures without refresh in Power BI service: Use ALM Toolkit. is a schema diff tool for Power BI models and can be used to perform the deployment of metadata only. Deploying from Power BI Desktop will overwrite the model’s data in the service and will require a refresh by the Power BI service to load the data.
- Filtering on Dimension table will almost always perform faster than filtering directly on any fact table, as that requires more processing by both the DAX formula and the storage engine.
- Only the distinct values displayed under Column distribution will show the true number of rows of values that are distinct (one row per value). The count of unique only shows the number of distinct values that are in the first 1,000 rows, and the other two options do not review uniqueness.
- **Column quality**: check for % of valid, error, empty values.  (Top part green, red, grey)
- **Column profile**: analyse value distribution with empty or error values. Distribution chart and column statistics table. No. Of distinct, unique, empty rows.(underneath)
- **Column distribution**: shows distinct and unique values for all cols (no empty or nulls). Frequency and distribution on top. Distinct = no. Of different values. Unique = values that only appear once. Does not show no. Of empty rows. (top part, bars)
- Tabular Editor process modes:  
  - Process Data loads data to a table without rebuilding hierarchies or relationships or recalculating calculated columns and measures.
  - With Process Default, hierarchies, calculated columns, and relationships are built or rebuilt (recalculated).
  - Process Defrag defragments the auxiliary table indexes, while
  - Process Recalc recalculates hierarchies, relationships, and calculated columns on a database level.
- To configure object-level security: use Tabular Editor
- Configure dynamic RLS: Use USERPRINCIPALNAME() to get user's email.

### Dax

- Create virtual relationships between tables: `TREATAS(table 1 col, table 2 col)`
- Declaring the [Variance] measure as a VAR will cache the measure and only load it once. This reduces the amount of processing and data loading that the measure must do and increases its speed. 

## Fabric administration

- Where to find lakehouse SQL connection string to use in SSMS: in the Lakehouse settings under Copy SQL connection string
- Fabric capcity licensing: below F64 capacity, users need Pro or Premium license to view Power BI content. F64 and above users do not need license. Pro license still needed for developing PBI reports.
- To increase fabric capacity unit size: use Azure portal
- Good data governance practice: use shared semantic models for multiple reports. Place semantic models and reports in separate workspaces.
- High concurrency mode is enabled at workspace level. On and off in Workspace settings. Use it to allow more than 1 notebook to use the same Spark session.
- Require authentication for embedded reports.
Allow only read-only (live) connections against Fabric capacity cloud semantic models. 2 actions: Disabling Publish to web disables the ability to publish any unsecured (no login required) reports to any embedded location. Disabling XMLA Endpoints ensures that semantic models can be connected to, but not edited directly in, workspaces.
- Contributor role is needed to read lakehouse data in a workspace.
- User must have read-only access to lakehouse, no access to rest of the items, cannot use Spark: Share lakehouse directly with user and enable Read all SQL Endpoint data. Do not add as workspace member.
