# Ngao-
Creating an innovative cybersecurity solution on Google Cloud Platform (GCP) can be an excellent way to address specific challenges or threats.

# Task 1. Preparation
You will be running a sensor simulator from the training VM. In Lab 1 you manually setup the Pub/Sub components. In this lab several of those processes are automated.

# Task 2. Create a BigQuery dataset and Cloud Storage bucket

The Dataflow pipeline will be created later and written into a table in the BigQuery dataset.

# Task 3. Simulate traffic sensor data into Pub/Sub

# Task 4. Launch Dataflow pipeline
Verify that Google Cloud Dataflow API is enabled for this project


#  Task 5. Explore the pipeline
This Dataflow pipeline reads messages from a Pub/Sub topic, parses the JSON of the input message, produces one main output and writes to BigQuery.

Return to the browser tab for Console. On the Navigation menu (Navigation menu icon), click Dataflow and click on your job to monitor progress.



Dataflow job monitoring window

Note: If Dataflow Job failed, run the command ./run_oncloud.sh $DEVSHELL_PROJECT_ID $BUCKET AverageSpeeds again.
After the pipeline is running, click on the Navigation menu (Navigation menu icon), click Pub/Sub > Topics.

Examine the line for Topic name for the topic sandiego.

Return to the Navigation menu (Navigation menu icon), click Dataflow and click on your job.

Compare the code in the Github browser tab, AverageSpeeds.java and the pipeline graph on the page for your Dataflow job.

Find the GetMessages pipeline step in the graph, and then find the corresponding code in the AverageSpeeds.java file. This is the pipeline step that reads from the Pub/Sub topic. It creates a collection of Strings - which corresponds to Pub/Sub messages that have been read.

Do you see a subscription created?
How does the code pull messages from Pub/Sub?
Find the Time Window pipeline step in the graph and in code. In this pipeline step we create a window of a duration specified in the pipeline parameters (sliding window in this case). This window will accumulate the traffic data from the previous step until end of window, and pass it to the next steps for further transforms.
What is the window interval?
How often is a new window created?
Find the BySensor and AvgBySensor pipeline steps in the graph, and then find the corresponding code snippet in the AverageSpeeds.java file. This BySensor does a grouping of all events in the window by sensor id, while AvgBySensor will then compute the mean speed for each grouping.

Find the ToBQRow pipeline step in the graph and in code. This step simply creates a "row" with the average computed from the previous step together with the lane information.

Note: In practice, other actions could be taken in the ToBQRow step. For example, it could compare the calculated mean against a predefined threshold and log the results of the comparison in Cloud Logging.
Find the BigQueryIO.Write in both the pipeline graph and in the source code. This step writes the row out of the pipeline into a BigQuery table. Because we chose the WriteDisposition.WRITE_APPEND write disposition, new records will be appended to the table.

Return to the BigQuery web UI tab. Refresh your browser.

Find your project name and the demos dataset you created. The small arrow to the left of the dataset name demos should now be active and clicking on it will reveal the average_speeds table.

It will take several minutes before the average_speeds table appears in BigQuery.


#  Task 6. Determine throughput rates
One common activity when monitoring and improving Dataflow pipelines is figuring out how many elements the pipeline processes per second, what the system lag is, and how many data elements have been processed so far. In this activity you will learn where in the Cloud Console one can find information about processed elements and time.

Return to the browser tab for Console. On the Navigation menu (Navigation menu icon), click Dataflow and click on your job to monitor progress (it will have your username in the pipeline name).

Select the GetMessages pipeline node in the graph and look at the step metrics on the right.

System Lag is an important metric for streaming pipelines. It represents the amount of time data elements are waiting to be processed since they "arrived" in the input of the transformation step.
Elements Added metric under output collections tells you how many data elements exited this step (for the Read PubSub Msg step of the pipeline it also represents the number of Pub/Sub messages read from the topic by the Pub/Sub IO connector).
Select the Time Window node in the graph. Observe how the Elements Added metric under the Input Collections of the Time Window step matches the Elements Added metric under the Output Collections of the previous step GetMessages.


#  Task 7. Review BigQuery output
Return to the BigQuery web UI.
Note: Streaming data and tables may not show up immediately, and the Preview feature may not be available for data that is still in the streaming buffer.
If you click on Preview you will see the message "This table has records in the streaming buffer that may not be visible in the preview." You can still run queries to view the data.
In the Query editor window, type (or copy-and-paste) the following query. Use the following query to observe the output from the Dataflow job. Click Run:
SELECT *
FROM `demos.average_speeds`
ORDER BY timestamp DESC
LIMIT 100
Copied!
Find the last update to the table by running the following SQL:
SELECT
MAX(timestamp)
FROM
`demos.average_speeds`
Copied!
Next use the time travel capability of BigQuery to reference the state of the table at a previous point in time.
The query below will return a subset of rows from the average_speeds table that existed at 10 minutes ago.

If your query requests rows but the table did not exist at the reference point in time, you will receive the following error message:

Invalid snapshot time 1633691170651 for Table PROJECT:DATASET.TABLE__

If you encounter this error please reduce the scope of your time travel by lowering the minute value:

SELECT *
FROM `demos.average_speeds`
FOR SYSTEM_TIME AS OF TIMESTAMP_SUB(CURRENT_TIMESTAMP, INTERVAL 10 MINUTE)
ORDER BY timestamp DESC
LIMIT 100

# Task 8. Observe and understand autoscaling
Observe how Dataflow scales the number of workers to process the backlog of incoming Pub/Sub messages.

Return to the browser tab for Console. On the Navigation menu (Navigation menu icon), click Dataflow and click on your pipeline job.

Examine the Job metrics panel on the right, and review the Autoscaling section. How many workers are currently being used to process messages in the Pub/Sub topic?

Click on More history and review how many workers were used at different points in time during pipeline execution.

The data from a traffic sensor simulator started at the beginning of the lab creates hundreds of messages per second in the Pub/Sub topic. This will cause Dataflow to increase the number of workers to keep the system lag of the pipeline at optimal levels.

Click on More history. In the Worker pool, you can see how Dataflow changed the number of workers. Notice the Status column that explains the reason for the change.



Click on More history. In the Worker pool, you can see how Dataflow changed the number of workers. Notice the Status column that explains the reason for the change.
#  Task 9. Refresh the sensor data simulation script
Task 10. Cloud Monitoring integration
Cloud Monitoring integration with Dataflow allows users to access Dataflow job metrics such as System Lag (for streaming jobs), Job Status (Failed, Successful), Element Counts, and User Counters from within Cloud Monitoring.
Task 11. Explore metrics
Cloud monitoring is a separate service in Google Cloud. So you will need to go through some setup steps to initialize the service for your lab account.


#  Task 12. Create alerts
If you want to be notified when a certain metric crosses a specified threshold (for example, when System Lag of our lab streaming pipeline increases above a predefined value), you could use the Alerting mechanisms of Monitoring to accomplish that.

#  Task 13. Set up dashboards
You can easily build dashboards with the most relevant Dataflow-related charts with Cloud Monitoring Dashboards.
