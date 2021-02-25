---
title: Data pull
layout: redirect
weight: 35
---

Machine Learning Workbench (MLW) provides connectors to the various data sources such as Cumulocity IoT, DataHub, etc. from where the data could be downloaded to start the machine learning model-building process.

### Cumulocity IoT

The following steps illustrate how to ingest and transform data produced by devices connected to Cumulocity IoT.

1. Click the add icon <img src="/images/zementis/mlw-add-new-resource-icon.png" alt="Add" style="display:inline-block; margin:0"> and select **Import from Cumulocity**.

2. Select the device for which you want to pull the data and click the download icon <img src="/images/zementis/mlw-download-icon.png" alt="Download" style="display:inline-block; margin:0"> under **Fetch Data**.

3. As part of data pull, provide the parameters such as data file name, data interval, data aggregation, and sensor name for data extraction. Once these parameters are provided, click the submit icon <img src="/images/zementis/mlw-submit-icon.png" alt="Submit" style="display:inline-block; margin:0">.

    ![Cumulocity parameters](/images/zementis/mlw-app-c8y-datapull-param.png)

Click **Tasks** in the navigator and click the corresponding task name, to display the status of the Cumulocity IoT data pull in the **Task History** section at the centre.

Once the task has reached the status COMPLETED, the data is stored in the **Data** folder of the respective project in Machine Learning Workbench (MLW).

### DataHub

The following steps illustrate how to ingest and transform data offloaded by Cumulocity IoT DataHub.

1. Click the add icon <img src="/images/zementis/mlw-add-new-resource-icon.png" alt="Add" style="display:inline-block; margin:0"> and select **Import from DataHub**.

2. Input the query, and click the submit icon <img src="/images/zementis/mlw-submit-icon.png" alt="Submit" style="display:inline-block; margin:0">.

    ![DataHub Query](/images/zementis/mlw-app-dh-query.png)

3. Provide the resource name with which you want to save the pulled data, and click **Submit**.

    ![DataHub name](/images/zementis/mlw-app-dh-name.png)

Click **Tasks** in the navigator and click the corresponding task name, to display the status of the Cumulocity IoT Datahub data pull in the **Task History** section at the centre.

Once the task has reached the status COMPLETED, the data will be stored in the **Data** folder of the respective project in Machine Learning Workbench (MLW).

Once the data is ingested from Cumulocity IoT or DataHub, the corresponding CSV file is available in the **Data** folder of the respective project in Machine Learning Workbench (MLW). You can view the metadata for the newly created CSV file by clicking on the respective file name.

Data from the CSV file can be previewed by clicking the preview icon <img src="/images/zementis/mlw-preview-icon.png" alt="Preview" style="display:inline-block; margin:0"> at the right of the top menu bar.