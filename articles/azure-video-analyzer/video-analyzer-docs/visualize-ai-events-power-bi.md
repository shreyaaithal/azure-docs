---
title: Real-time visualization of AI inference events with Power BI
description: You can use Azure Video Analyzer for continuous video recording or event-based recording. This tutorial walks through the steps for real-time to visualization AI inference events from IoT Hub in Microsoft Power BI.
ms.service: azure-video-analyzer
ms.topic: tutorial
ms.date: 09/08/2021
ms.custom: ignite-fall-2021
---

# Tutorial: Real-time visualization of AI inference events with Power BI

Azure Video Analyzer provides the capability to capture, record, and analyze live video along with publishing the results of video analysis in form of AI inference events to the [IoT Edge Hub](../../iot-edge/iot-edge-runtime.md?view=iotedge-2020-11&preserve-view=true#iot-edge-hub). These AI inference events can then be routed to other destinations including Visual Studio Code and Azure services such as Time Series Insights and Event Hubs.

Dashboards are an insightful way to monitor your business and visualize all your important metrics at a glance. You can visualize AI inference events generated by Video Analyzer using [Microsoft Power BI](https://powerbi.microsoft.com/) via [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/#overview) to quickly gain insights and share dashboards with peers in your organization.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/power-bi/tutorial-block-diagram.svg" alt-text="Block diagram to connect Azure Video Analyzer to Microsoft Power BI via Azure Stream Analytics.":::

In this tutorial, you will:

- Create and run a Stream Analytics job to retrieve necessary data from IoT Hub and send it to Power BI
- Run a live pipeline that generates inference events
- Create a Power BI dashboard to visualize the AI inferences

## Suggested pre-reading

- [Monitoring and logging](monitor-log-edge.md) in Video Analyzer
- Reading [device-to-cloud messages from IoT Hub](../../iot-hub/iot-hub-devguide-messages-read-builtin.md) built-in endpoints
- Introduction to [Power BI dashboards](/power-bi/create-reports/service-dashboards)

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) if you don't already have one.
- This tutorial is based on using [Line Crossing sample](use-line-crossing.md) to detect when objects cross a virtual line in a live video feed. You can choose to create visualization for other pipelines - **a pipeline with IoT Hub message sink is required**. Make sure to have the live pipeline created, but activate it only after creating a Stream Analytics job.

  > [!TIP]
  >
  > - The [Line Crossing sample](use-line-crossing.md) uses a 5-minute video recording. For best results in visualization, use the 60-minute recording of vehicles on a freeway available in [Other dataset](https://github.com/Azure/video-analyzer/tree/main/media#other-dataset).
  > - Refer Configuration and deployment section in [FAQs](faq-edge.yml) on how to add sample video files to rtsp simulator. Once added, edit `rtspUrl` value to point to the new video file.
  > - If you followed the Line Crossing sample and are using the [AVA C# sample repository](https://github.com/Azure-Samples/video-analyzer-iot-edge-csharp), then edit operations.json file at properties -> parameters -> value to `"rtsp://rtspsim:554/media/camera-3600s.mkv"` to change video source to 60-minute recording.

- A [Power BI](https://powerbi.microsoft.com/) account.

## Create and run a Stream Analytics Job

[Stream Analytics](https://azure.microsoft.com/services/stream-analytics/#overview) is a fully managed, real-time analytics service designed to help you analyze and process fast moving streams of data. In this section, you will create a Stream Analytics job, define the inputs, outputs and the query used to retrieve the required data.

### Create a Stream Analytics job

1. In the [Azure portal](https://portal.azure.com/), select **Create a resource**. Type _Stream Analytics Job_ in the search box and select it from the drop-down list. On the **Stream Analytics job** overview page, select **Create**
2. Enter the following information for the job:

   - **Job name**: The name of the job. The name must be globally unique.
   - **Subscription**: Choose the same subscription used for setting up the Line Crossing sample.
   - **Resource group**: Use the same resource group that your IoT Edge hub uses as part of setting up the Line Crossing sample.
   - **Location**: Use the same location as your resource group.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/power-bi/create-asa-job.png" alt-text="Screenshot to create a new Stream Analytics Job.":::

3. Select **Create**.

### Add an input to the Stream Analytics job

1. Open the Stream Analytics job.
2. Under Job topology, select **Inputs**.
3. In the Inputs pane, select **Add stream input**, then select **IoT Hub** from the drop-down list. On the new input pane, enter the following information:

   - **Input alias**: Enter a unique alias for the input, such as "iothubinput".
   - **Select IoT Hub from your subscription**: Select this radio button.
   - **Subscription**: Select the Azure subscription you're using for this article.
   - **IoT Hub**: Select the IoT Hub used in setting up the Line Crossing sample.
   - **Shared access policy name**: Select the name of the shared access policy you want the Stream Analytics job to use for your IoT hub. For this tutorial, you can select iothubowner. To learn more, see [Access control and permissions](../../iot-hub/iot-hub-dev-guide-sas.md#access-control-and-permissions).
   - **Shared access policy key**: This field is auto filled based on your selection for the shared access policy name.

   Leave all other fields set to their defaults.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/power-bi/add-iothub-input.png" alt-text="Screenshot to add IoT Hub input to Stream Analytics Job.":::

4. Select **Save**.

### Add an output to the Stream Analytics job

1. Under Job topology, select **Outputs**.
2. In the Outputs pane, select **Add**, and then select **Power BI** from the drop-down list.
3. On the Power BI - New output pane:

   - **Output alias**: Enter a unique alias for the output, such as "powerbioutput".
   - **Group workspace**: Select your target group workspace.
   - **Authentication mode**: Leave the default “User token” for testing purposes.

   > [!NOTE]
   > For production jobs, we recommend to [Use Managed Identity to authenticate your Stream Analytics job to Power BI](../../stream-analytics/powerbi-output-managed-identity.md).

   - **Dataset name**: Enter a dataset name.
   - **Table name**: Enter a table name.
   - **Authorize**: Select Authorize and follow the prompts to sign into your Power BI account (the token is valid for 90 days).

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/power-bi/add-pbi-output.png" alt-text="Screenshot to add Power BI output to Stream Analytics Job.":::

4. Select **Save**.

   > [!NOTE]
   > For more information on using Power BI as output for a Stream Analytics job, click [here](../../stream-analytics/power-bi-output.md). Learn more about [renewing authentication](../../stream-analytics/stream-analytics-power-bi-dashboard.md#renew-authorization) for your Power BI account.

### Configure the query for Stream Analytics job

1. Under Job topology, select **Query**.
2. Make the following changes to your query:

   ```SQL
   SELECT
       CAST(InferenceRecords.ArrayValue.event.properties.total AS bigint) as EventTotal,
       CAST(i.EventProcessedUtcTime AS datetime) as EventProcessedUtcTime
   INTO [YourOutputAlias]
   FROM [YourInputAlias] i
   CROSS APPLY GetArrayElements(inferences) AS InferenceRecords
   WHERE InferenceRecords.ArrayValue.subType = 'lineCrossing'
   ```

   > [!NOTE]
   >
   > - In the above query, the _i_ in FROM clause is syntactically required to fetch value of EventProcessedUtcTime that is not nested in the _inferences_ array.
   >   The above query is customized to get AI inferences for [Line Crossing tutorial](use-line-crossing.md).
   > - If you are running another pipeline, ensure to customize the query according to the pipeline's AI inference schema. Learn more about [Parsing JSON in Stream Analytics job](../../stream-analytics/stream-analytics-parsing-json.md).

3. Replace [YourOutputAlias] with the output alias used in the step to Add an output to the Stream Analytics job such as "powerbioutput". Note the right sequence of output and input aliases.
4. Replace [YourInputAlias] with the input alias used in the step to Add an input to the Stream Analytics job such as "iothubinput".
5. Your query should look similar to the following screenshot. Click on **Test query** to verify the Test results as a table of EventTotal and corresponding EventProcessedUtcTime values.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/power-bi/asa-query.png" alt-text="Screenshot to test and save query in Stream Analytics Job.":::

6. Select **Save query**.

### Run the Stream Analytics job

In the Stream Analytics job, select Overview, then select **Start** > **Now** > **Start**. Once the job successfully starts, the job status changes from Created to **Running**.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/power-bi/start-asa.png" alt-text="Screenshot to Start and Run a Stream Analytics Job.":::

> [!TIP]
> IoT Hub allows data retention in the built-in Event Hubs by default for 1 day and for a maximum of 7 days. You can set the retention time during creation of your IoT Hub. Data retention time in IoT Hub depends on your chosen tier and unit type. Click [here](../../iot-hub/iot-hub-devguide-messages-read-builtin.md) for more information. For longer data retention, use [Azure Storage as output](../../stream-analytics/blob-storage-azure-data-lake-gen2-output.md) and then connect Power BI to the files in storage account.

## Run a sample pipeline

When the Stream Analytics job created in above step is in **Running** state, go to [Run the sample program](use-line-crossing.md#run-the-sample-program) section of Line Crossing tutorial and activate the live pipeline. The live pipeline will start sending AI inference results to IoT Hub, which are then picked up by the Stream Analytics job.

## Create a Power BI dashboard to visualize AI events

In Power BI, you can visualize streaming data in 2 ways:

1. Create reports from table created for streaming dataset and pin to dashboard
2. A dashboard tile with custom streaming dataset

   > [!NOTE]
   > In this article, we will use the first method to create reports and then pin to dashboard. This method retains data on the visual for longer duration and aggregates automatically based on incoming data. To learn more about the second method, see [Setup your custom streaming dataset in Power BI](/power-bi/connect-data/service-real-time-streaming#set-up-your-real-time-streaming-dataset-in-power-bi).

### Create and publish a Power BI report

The following steps show you how to create and publish a report using the Power BI service.

1. Make sure the sample pipeline is activated on your device (started in previous step to run a pipeline).
2. Sign in to your [Power BI account](https://powerbi.microsoft.com/) and select **Power BI service** from the top menu.
3. Select the workspace you used from the side menu.
4. Under the **All** tab or the **Datasets + dataflows** tab, you should see the dataset that you specified in the step to _Add an output to the Stream Analytics job_.
5. Hover over the dataset name you provided while creating Stream Analytics output, select **More options** menu (the three dots to the right of the dataset name), and then select **Create report**.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/power-bi/create-report.png" alt-text="Screenshot to create report in Power BI.":::

6. Create a line chart to show line crossing events in real time.

   - On the **Visualizations** pane of the report creation page, select the **Line chart** icon to add a line chart.
   - On the **Fields** pane, expand the table that you specified when you created the output for the Stream Analytics job.
   - Drag **EventProcessedUtcTime** to **Axis** on the **Visualizations** pane.
   - Drag **EventTotal** to **Values**.
   - A line chart is created. The x-axis displays date and time in the UTC time zone. The y-axis displays auto-summarized values for **Sum** of EventTotal from the live pipeline. Change the Values to **Maximum** of EventTotal to get the most recent value of EventTotal. You can change the aggregation function to show Average, Count (Distinct) etc.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/power-bi/powerbi-report.png" alt-text="Screenshot of line crossing report in Power BI.":::

7. Save the report by clicking **Save** at the top-right.

### Pin visual to a dashboard

Click **Pin to a dashboard** at the top-right and select where you want to pin this visual – in an existing dashboard or a new dashboard, and then follow the prompts accordingly.

Optionally, you can also [embed a player widget](embed-player-in-power-bi.md) to view video playback side-by-side with the report.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/power-bi/pinned-dashboard.png" alt-text="Power BI Dashboard with report pinned and widget player added in a tile.":::

> [!NOTE]
> The video playback and report creation will not be in sync because of the method used to generate reports. For a more near real-time experience, see [Setup your custom streaming dataset in Power BI](/power-bi/connect-data/service-real-time-streaming#set-up-your-real-time-streaming-dataset-in-power-bi).

## Interpret the dashboard

The line crossing processor node detects when an object crosses a line specified in the topology using lineCoordinates parameter. When objects cross these coordinates, an event is triggered with:

- EventTotal: The total number of line crossings by any object in any direction (clockwise or counterclockwise) so far since beginning of the video. Learn more about [Line Crossing Event inferences](use-line-crossing.md#line-crossing-events).
- Event Processed UTC Time: The time at which each event was processed by Stream Analytics Job. Alternatively, you can customize the Stream Analytics query to fetch EventEnqueuedUtcTime which denotes the time when the event reached the IoT Hub ready for further processing.

In the above dashboard, you will see an increasing number of **EventTotal** with time as more and more objects cross the virtual line. This visualization enables you to quickly identify that vehicles passing on the freeway have high frequency between 3:52:00 and 3:53:30 PM for example. You can then use these insights to narrow down your analysis on reasons for traffic congestion at certain times on the freeway.

## Clean up resources

If you want to try other quickstarts or tutorials, keep the resources that you created. Otherwise, go to the Azure portal, go to your resource groups, select the resource group where you ran the pre-requisites for this article and created the stream analytics job, and then select **Delete resource group**.

You created a dataset **LineCrossingDataset** in Power BI. To remove it, sign in to your [Power BI](https://powerbi.microsoft.com/) account. On the left-hand menu under **Workspaces**, select the workspace you used. In the list of datasets under the **Datasets + dataflows** tab, hover over the dataset. Select the three vertical dots that appear to the right of the dataset name to open the **More options** menu, then select **Delete** and follow the prompts. When you remove the dataset, the report is removed as well.

## Next steps

- Combine video playback with telemetry by [embedding player widget into Power BI](embed-player-in-power-bi.md)
- Learn more about [Monitoring and Logging events](monitor-log-edge.md)
