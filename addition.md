# Lab 03: Data Engineering – Pipelines and Cross-Database Queries

### Estimated Duration: 60 Minutes

## 📘 Lab Scenario

Contoso Retail has established a Lakehouse and a Data Warehouse in Microsoft Fabric. Now, the data engineering team needs to automate the data movement between these assets. Instead of manually uploading files and running scripts, you will use **Data Pipelines** to ingest data into the Lakehouse, transform it into Delta tables, and load curated data from the Lakehouse into the Data Warehouse using **cross-database queries** — all orchestrated end-to-end through pipelines.

As a Data Engineer, you will build a production-style data pipeline that mirrors real-world ETL patterns: ingest raw data, transform it using notebooks, and load refined data into the warehouse for reporting.

## 📋 Overview

Microsoft Fabric Data Pipelines provide a visual orchestration engine to move and transform data across Fabric items. Combined with cross-database (cross-object) queries, you can read Lakehouse Delta tables directly from the Warehouse SQL endpoint — eliminating the need for redundant data copies.

In this lab, you will:

- Create a Data Pipeline to copy external data into the Lakehouse
- Use a Notebook activity to transform raw data into a curated Delta table
- Use cross-database queries to load data from the Lakehouse into the Data Warehouse
- Orchestrate all steps in a single pipeline with proper sequencing

## 🏗️ Architecture Diagram

The following diagram illustrates the end-to-end data flow you will build in this lab:

```
External Source (CSV)
        │
        ▼
 ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
 │  Copy Activity│     │  Notebook    │     │  Script      │
 │  (Ingest raw  │────▶│  Activity    │────▶│  Activity    │
 │   CSV to      │     │  (Transform  │     │  (Cross-DB   │
 │   Lakehouse   │     │   to Delta)  │     │   query to   │
 │   Files)      │     │              │     │   Warehouse) │
 └──────────────┘     └──────────────┘     └──────────────┘
        │                     │                     │
        └─────────── Data Pipeline ─────────────────┘
```

## 🎯 Lab objectives

In this lab, you will complete the following tasks:

- Task 1: Create a Notebook for data transformation
- Task 2: Create a Data Pipeline
- Task 3: Add a Copy Activity to ingest data into the Lakehouse
- Task 4: Add a Notebook Activity to transform data
- Task 5: Use cross-database query to load data into the Warehouse
- Task 6: Run and monitor the pipeline

## Task 1: Create a Notebook for data transformation

In this task, you will create a Spark notebook that transforms the raw CSV data (ingested into the Lakehouse Files area) into a cleansed Delta table. This notebook will later be called from the pipeline.

1. In the Power BI portal, make sure you are in workspace **Workspace-<inject key="DeploymentID" enableCopy="false"/>**, then click **Power BI** **(1)** on the left navigation bar, and click **+ New item** **(2)**.

   ![](<./Images/L2T1S1.png>)

   > **Reused from:** Lab 02, Task 1 — same "+ New item" action in workspace.

1. On the **New item** page, search for **Notebook** **(1)** in the search bar and select **Notebook** **(2)**.

   ![](<./Images/L3T1S2.png>)

   > **📸 NEW screenshot needed** — Search for "Notebook" on New item page.

   > **Note**: If prompted to select a Lakehouse, choose **Lakehouse_<inject key="DeploymentID" enableCopy="false"/>** and click **Add**.

1. Click on the notebook name at the top and rename it to **nb_transform_products** **(1)**.

   ![](<./Images/L3T1S3.png>)

   > **📸 NEW screenshot needed** — Notebook title bar showing rename to `nb_transform_products`.

1. If the notebook does not have a default Lakehouse attached, click **Add** in the **Lakehouse** section of the left **Explorer** pane, select **Existing lakehouse**, choose **Lakehouse_<inject key="DeploymentID" enableCopy="false"/>**, and click **Add**.

1. First, upload the sample data so you can test the notebook. In the **Lakehouse explorer** pane on the left, click the ellipses **(...)** next to the **Files** folder, then select **New subfolder**. Name it **raw** and click **Create**.

1. Click the ellipses **(...)** next to the **raw** folder, hover over **Upload**, then select **Upload files**. Download the file from [this link](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/products.csv), then upload the downloaded **products.csv** file and click **Upload**. Once the status shows **Completed**, close the upload dialog.

   > **Note**: Alternatively, you can paste and run the following code in a cell to download the file directly, then delete the cell afterwards:
   >> ```python
   > import urllib.request, os
   > dest = "/lakehouse/default/Files/raw/products.csv"
   > os.makedirs(os.path.dirname(dest), exist_ok=True)
   > urllib.request.urlretrieve("https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/products.csv", dest)
   > print("Download complete.")
   > ```

1. In the first cell, paste the following PySpark code and click the **&#9655; Run** button to execute it:

   ```python
   # Read the raw CSV file from the Lakehouse Files area
   df_raw = spark.read.format("csv") \
       .option("header", "true") \
       .option("inferSchema", "true") \
       .load("Files/raw/products.csv")

   # Display the raw data
   print(f"Raw row count: {df_raw.count()}")
   df_raw.show(5)
   ```

   > **Note**: You should see **295 rows** and 4 columns: `ProductID`, `ProductName`, `Category`, `ListPrice`.

1. Click **+ Code** below the first cell to add a new code cell. Paste the following transformation code and run it:

   ```python
   from pyspark.sql import functions as F

   # Clean and transform the data
   df_clean = (
       df_raw
       .dropna(subset=["ProductName", "Category", "ListPrice"])
       .withColumn("ListPrice", F.col("ListPrice").cast("decimal(10,2)"))
       .withColumn("ProductName", F.trim(F.col("ProductName")))
       .withColumn("Category", F.trim(F.col("Category")))
       .withColumn("LoadTimestamp", F.current_timestamp())
   )

   # Write as a managed Delta table in the Lakehouse
   df_clean.write.format("delta") \
       .mode("overwrite") \
       .saveAsTable("stg_products")

   print(f"Staged row count: {df_clean.count()}")
   df_clean.show(5)
   ```

1. In the **Lakehouse explorer** pane, click the **Refresh** icon. Expand **Tables** and verify that the **stg_products** table now appears.

   ![](<./Images/L3T1S7.png>)

   > **📸 NEW screenshot needed** — Lakehouse explorer showing `stg_products` under Tables.

   > **Congratulations** on completing the task! Now, it's time to validate it. Here are the steps:
   >> - If you receive a success message, you can proceed to the next task.
   > - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
   > - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help you out.

<validation step="a1b2c3d4-e5f6-7890-abcd-ef1234567890" />

## Task 2: Create a Data Pipeline

In this task, you will create a new Data Pipeline that will orchestrate the entire data flow — from ingestion to transformation to loading.

1. In the hub menu bar on the left, click on your workspace **Workspace-<inject key="DeploymentID" enableCopy="false"/> (1)**.

1. Click **+ New item** **(1)** at the top of the workspace.

   ![](<./Images/E1T2S1.png>)

   > **Reused from:** Lab 01, Task 2 — same "+ New item" button.

1. On the **New item** page, search for **Data pipeline** **(1)** in the search bar and select **Data pipeline** **(2)**.

   ![](<./Images/L3T2S3.png>)

   > **📸 NEW screenshot needed** — Search for "Data pipeline" on New item page.

1. In the **New pipeline** dialog, enter **pl_ingest_and_load** **(1)** as the pipeline name, then click **Create** **(2)**.

   ![](<./Images/L3T2S4.png>)

   > **📸 NEW screenshot needed** — New pipeline dialog with name `pl_ingest_and_load`.

1. The pipeline canvas opens with an empty design surface. You will add activities in the next tasks.

   ![](<./Images/L3T2S5.png>)

   > **📸 NEW screenshot needed** — Empty pipeline design canvas.

## Task 3: Add a Copy Activity to ingest data into the Lakehouse

In this task, you will add a **Copy Data** activity to the pipeline. This activity will download a sample products CSV file from an external HTTP source and land it in the Lakehouse **Files** folder.

1. On the pipeline canvas, click **Add pipeline activity** or select **Copy data** **(1)** from the **Activities** toolbar.

   ![](<./Images/L3T3S1.png>)

   > **📸 NEW screenshot needed** — Activities toolbar with "Copy data" highlighted.

1. Select the **Copy data** activity on the canvas and click the **General** tab at the bottom. Set the **Name** to **Copy Products CSV** **(1)**.

   ![](<./Images/L3T3S2.png>)

   > **📸 NEW screenshot needed** — Copy activity General tab, Name set to `Copy Products CSV`.

1. Click the **Source** **(1)** tab. Under **Connection**, click **+ New** **(2)** to create a new connection.

   ![](<./Images/L3T3S3.png>)

   > **📸 NEW screenshot needed** — Source tab showing "+ New" connection button.

1. In the **New connection** dialog:

   - Select **HTTP** **(1)** as the data source type.
   - In the **URL** field **(2)**, enter the following:

     ```
     https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/products.csv
     ```

   - Set **Authentication kind** to **Anonymous** **(3)**.
   - Click **Create** **(4)**.

   ![](<./Images/L3T3S4.png>)

   > **📸 NEW screenshot needed** — New HTTP connection dialog with URL and Anonymous auth.

1. Back on the **Source** tab, under **File format**, confirm the format is set to **DelimitedText** **(1)** and **First row as header** is checked **(2)**.

   ![](<./Images/L3T3S5.png>)

   > **📸 NEW screenshot needed** — Source tab file format showing DelimitedText + header checkbox.

1. Click the **Destination** **(1)** tab:

   - Under **Data store type**, select **Workspace** **(2)**.
   - Under **Workspace data store type**, select **Lakehouse** **(3)**.
   - Select **Lakehouse_<inject key="DeploymentID" enableCopy="false"/>** **(4)** as the Lakehouse.
   - Set **Root folder** to **Files** **(5)**.
   - In the **File path** field, enter the folder path **raw** **(6)** and file name **products.csv** **(7)**.

   ![](<./Images/L3T3S6.png>)

   > **📸 NEW screenshot needed** — Destination tab with Lakehouse, Files root, `raw/products.csv` path.

   > **Note**: This will place the file at `Files/raw/products.csv` in the Lakehouse, which is the path the notebook expects. If you already uploaded the file manually in Task 1, the Copy activity will overwrite it on each pipeline run — this is expected and ensures the pipeline is self-contained.

## Task 4: Add a Notebook Activity to transform data

In this task, you will add a **Notebook** activity after the Copy activity. This will execute the transformation notebook you created in Task 1 to convert the raw CSV into a Delta table.

1. On the pipeline canvas, from the **Activities** toolbar, select **Notebook** **(1)** to add a Notebook activity.

   ![](<./Images/L3T4S1.png>)

   > **📸 NEW screenshot needed** — Activities toolbar with "Notebook" highlighted.

1. Select the **Notebook** activity on the canvas and click the **General** tab. Set the **Name** to **Transform to Delta** **(1)**.

   ![](<./Images/L3T4S2.png>)

   > **📸 NEW screenshot needed** — Notebook activity General tab, Name set to `Transform to Delta`.

1. Click the **Settings** **(1)** tab:

   - Under **Notebook**, click the dropdown and select **nb_transform_products** **(2)**.

   ![](<./Images/L3T4S3.png>)

   > **📸 NEW screenshot needed** — Notebook activity Settings tab with `nb_transform_products` selected.

1. Now connect the two activities. Hover over the right edge of the **Copy Products CSV** activity until the green checkmark (&#10004;) connector appears. Drag it to the **Transform to Delta** activity.

   ![](<./Images/L3T4S4.png>)

   > **📸 NEW screenshot needed** — Green on-success connector between Copy and Notebook activities.

   > **Note**: The green connector means the Notebook activity will only run **on success** of the Copy activity. This ensures transformation only occurs after data is successfully ingested.

## Task 5: Use cross-database query to load data into the Warehouse

In this task, you will add a **Script** activity that uses a cross-database query to read the Delta table from the Lakehouse and insert it into a table in the Data Warehouse — all without moving files.

1. First, you need to create the target table in the Warehouse. Open a **new browser tab** and navigate to your workspace **Workspace-<inject key="DeploymentID" enableCopy="false"/>**.

1. Select the **myDataWarehouse** **(1)** warehouse to open it.

   ![](<./Images/L2T4S1.png>)

   > **Reused from:** Lab 02, Task 4 — same warehouse selection from workspace.

1. On the **Home** tab, click the dropdown next to **New SQL query** **(1)**, then select **New SQL query** **(2)**.

   ![](<./Images/L1T62.png>)

   > **Reused from:** Lab 01, Task 6 — same "New SQL query" dropdown.

1. Paste the following SQL and click **Run** to create the target table:

   ```sql
   CREATE TABLE dbo.DimProductStaging
   (
       ProductID INT,
       ProductName VARCHAR(100),
       Category VARCHAR(50),
       ListPrice DECIMAL(10,2),
       LoadTimestamp DATETIME2
   );
   GO
   ```

   ![](<./Images/L3T5S4.png>)

   > **📸 NEW screenshot needed** — Query results confirming `DimProductStaging` table created.

1. Verify that the table **DimProductStaging** appears in the **Explorer** pane under **dbo > Tables**. Click **Refresh** if needed.

1. Now, test the **cross-database query**. In a new SQL query, run the following to verify you can read Lakehouse data from the Warehouse:

   ```sql
   SELECT TOP 10 *
   FROM Lakehouse_<inject key="DeploymentID" enableCopy="false"/>.dbo.stg_products;
   GO
   ```

   ![](<./Images/L3T5S6.png>)

   > **📸 NEW screenshot needed** — Cross-database query results showing Lakehouse data in Warehouse.

   > **Note**: Cross-database queries use the **three-part naming** convention: `LakehouseName.SchemaName.TableName`. Both the Lakehouse and Warehouse must be in the **same workspace** for this to work.

1. Switch back to the browser tab with your **pl_ingest_and_load** pipeline.

1. From the **Activities** toolbar, select **Script** **(1)** to add a Script activity to the pipeline canvas.

   ![](<./Images/L3T5S8.png>)

   > **📸 NEW screenshot needed** — Activities toolbar with "Script" highlighted.

1. Select the **Script** activity on the canvas and click the **General** tab. Set the **Name** to **Load to Warehouse** **(1)**.

   ![](<./Images/L3T5S9.png>)

   > **📸 NEW screenshot needed** — Script activity General tab, Name set to `Load to Warehouse`.

1. Click the **Settings** **(1)** tab:

   - Under **Connection**, select your **myDataWarehouse** **(2)** warehouse connection.
   - Set **Script type** to **NonQuery** **(3)**.
   - In the **Script** field, paste the following SQL **(4)**:

   ```sql
   -- Truncate and reload the staging table using cross-database query
   TRUNCATE TABLE dbo.DimProductStaging;

   INSERT INTO dbo.DimProductStaging (ProductID, ProductName, Category, ListPrice, LoadTimestamp)
   SELECT
       ProductID,
       ProductName,
       Category,
       ListPrice,
       LoadTimestamp
   FROM Lakehouse_<inject key="DeploymentID" enableCopy="false"/>.dbo.stg_products;
   ```

   ![](<./Images/L3T5S10.png>)

   > **📸 NEW screenshot needed** — Script activity Settings tab: warehouse connection, NonQuery, SQL pasted.

   > **Note**: Replace `Lakehouse_<inject key="DeploymentID" enableCopy="false"/>` with your actual Lakehouse name if needed. The `TRUNCATE` ensures idempotent runs — the pipeline can be rerun without creating duplicates.

1. Connect the **Transform to Delta** activity to the **Load to Warehouse** activity using the green (on success) connector, just as you did in Task 4.

   ![](<./Images/L3T5S11.png>)

   > **📸 NEW screenshot needed** — Green connector between Notebook and Script activities.

1. Your completed pipeline should now look like this, with three activities connected in sequence:

   ```
   Copy Products CSV ──▶ Transform to Delta ──▶ Load to Warehouse
   ```

   ![](<./Images/L3T5S12.png>)

   > **📸 NEW screenshot needed** — Full pipeline canvas with all 3 activities connected in sequence.

## Task 6: Run and monitor the pipeline

In this task, you will validate, run, and monitor the pipeline to ensure all three activities complete successfully.

1. On the pipeline toolbar, click **Validate** **(1)** to check for configuration errors. Resolve any errors that are reported.

   ![](<./Images/L3T6S1.png>)

   > **📸 NEW screenshot needed** — Pipeline toolbar with "Validate" button highlighted.

1. Once validation passes, click **Run** **(1)** to execute the pipeline.

   ![](<./Images/L3T6S2.png>)

   > **📸 NEW screenshot needed** — Pipeline toolbar with "Run" button highlighted.

1. If prompted to save, click **Save and run**.

1. The **Output** tab opens at the bottom, showing the pipeline run progress. Monitor each activity status:

   - **Copy Products CSV** — should show **Succeeded** (&#10004;)
   - **Transform to Delta** — should show **Succeeded** (&#10004;)
   - **Load to Warehouse** — should show **Succeeded** (&#10004;)

   ![](<./Images/L3T6S4.png>)

   > **📸 NEW screenshot needed** — Output tab showing all 3 activities with "Succeeded" status.

   > **Note**: The pipeline run typically completes in 2-4 minutes. If an activity fails, click on it to see the error details and troubleshoot.

1. To verify the data landed in the Warehouse, switch to the browser tab with **myDataWarehouse**.

1. Open a **New SQL query** and run the following:

   ```sql
   SELECT COUNT(*) AS TotalProducts FROM dbo.DimProductStaging;
   GO

   SELECT TOP 10 * FROM dbo.DimProductStaging ORDER BY Category, ProductName;
   GO
   ```

   ![](<./Images/L3T6S6.png>)

   > **📸 NEW screenshot needed** — Warehouse query results showing product rows in `DimProductStaging`.

   You should see the product rows that were ingested from the CSV, transformed in the notebook, and loaded via the cross-database query.

1. **(Optional)** To schedule the pipeline for recurring runs:

   - Go back to the pipeline **pl_ingest_and_load**.
   - Click **Schedule** **(1)** on the toolbar.
   - Toggle the schedule **On** **(2)**.
   - Set the **Frequency** to **Daily** **(3)** and choose a time.
   - Click **Apply** **(4)**.

   ![](<./Images/L3T6S7.png>)

   > **📸 NEW screenshot needed** — Schedule dialog with daily frequency configured.

> **Congratulations** on completing the task! Now, it's time to validate it. Here are the steps:
>> - If you receive a success message, you can proceed to the next task.
> - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
> - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help you out.

<validation step="b2c3d4e5-f6a7-8901-bcde-f12345678901" /><validation step="c3d4e5f6-a7b8-9012-cdef-123456789012" />

## Summary

In this exercise, you have accomplished the following:

- Created a Spark Notebook to transform raw CSV data into a managed Delta table in the Lakehouse
- Built a Data Pipeline with three orchestrated activities: Copy, Notebook, and Script
- Used a Copy Data activity to ingest external data into the Lakehouse Files area
- Used a Notebook activity to run Spark transformations within the pipeline
- Leveraged cross-database queries (three-part naming) to load data from the Lakehouse into the Data Warehouse without data duplication
- Executed and monitored the end-to-end pipeline run
- Learned how to schedule the pipeline for recurring automated runs

### You have successfully completed the lab. Click on Next >>.

