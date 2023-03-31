# Introduction
MongoDB Atlas is a great choice for the foundation of an Operational Data Layer (ODL) which can bring an end to the woes of the siloed data and can be that central Single view which can enable large scale limitless real-time analytics. Azure Synapse analytics is a unified platform that supports the multi-dimensional, infinite scale  analytics needs of the modern enterprise.Thus, the synergy would enable any enterprise to operationalise their siloed data and use it for powerful analytics to derive rich insights from their data.

# MongoDB Atlas and Synapse integration
This repo gives instructions to achieve a Batch/ micro-batch integration between MongoDB Atlas and Synapse.

## Batch/ Micro-batch Integration
In Azure Synapse Analytics, you can integrate MongoDB on-premises instances and MongoDB Atlas as a Source or Sink resource. With historical data, you can retrieve all the data at once. You can also retrieve data incrementally for specific periods by using a filter in batch mode. Then you can use SQL pools and Apache Spark pools in Azure Synapse Analytics to transform and analyze the data. If you need to store the analytics or query results in an analytics data store, you can use the MongoDB sink resource in Azure Synapse Analytics.
![Fig1](https://user-images.githubusercontent.com/104025201/229103014-ca109b01-bdec-466b-8d14-8188c619769a.png)

For more information about how to set up and configure the connectors, see these resources for [MongoDB](https://learn.microsoft.com/en-us/azure/data-factory/connector-mongodb?tabs=data-factory) and [MongoDB Atlas](https://learn.microsoft.com/en-us/azure/data-factory/connector-mongodb-atlas?tabs=data-factory). 

### Pre-requisites:
You will need the below set up before starting the Lab:
1. MongoDB Atlas cluster setup: 
Register for a new Atlas Account [here](https://www.mongodb.com/docs/atlas/tutorial/create-atlas-account/#register-a-new-service-account). Follow steps from 1 to 4 (Create an Atlas account, Deploy a Free cluster, Add your IP to the IP access list and Create a Database user) to set up the Atlas environment. Also, follow step 7 “Load Sample Data” to load sample data to be used in the lab.

**Note: For this lab add “0.0.0.0/0” to the IP access list so that Synapse can connect to MongoDB Atlas. In Production scenarios, its not recommended and you will use Vnet Peering or Private Link options.**

2. Azure account setup: 
Follow link [here](https://azure.microsoft.com/en-in/free/) to set up a free azure account
3. Azure Synapse Analytics Workspace setup:
Follow link [here](https://learn.microsoft.com/en-us/azure/synapse-analytics/get-started-create-workspace) to set up a Synapse workspace within you Azure account

### Integration Steps:
  1. [Pipeline Creation](#pipeline-creation)
  2. [Set up Atlas as Source](#set-up-atlas-as-source) 
  3. [Set up ADLS Gen2 as Sink](#set-up-adls-gen2-as-sink)
  4. [Publish Changes and Run the Pipeline](#publish-changes-and-run-the-pipeline)

#### Pipeline Creation
- Open the newly created Synapse workspace and navigate to the Synapse Studio by selecting “Open Synapse Studio”
<img width="454" alt="fig2" src="https://user-images.githubusercontent.com/104025201/229139179-4132ef32-1264-4439-ae14-598a05186430.png">

- Navigate to the “Integrate” tab and select “+” sign and “Pipeline” to create a new Synapse Pipeline.
<img width="454" alt="fig3" src="https://user-images.githubusercontent.com/104025201/229139264-14eb20cf-f896-424f-a26a-b931c2b726e2.png">

- Drag and Drop “Copy data”activity from the “Move & transform” tile to the center plane to add the “Copy data" activity into the new Pipeline named “Pipeline 1” by default.
<img width="400" alt="fig4" src="https://user-images.githubusercontent.com/104025201/229139335-a9b1191d-015c-44b2-9499-5b45c97b9bd8.png">


#### Set up Atlas as Source
- Select the “Source” tab from the bottom plane and click on the “+ New” button against the “Source dataset” label.
- Search for “mongodb” in the box against the label “Select a data store” . Select “MongoDB Atlas” and press the “Continue” button.
- Select “New” against the “Linked Service” box to create a new Atlas cluster integration.
- In the MongoDB Atlas “Database” view (on left) under “Data Services” (on top) Select “Connect” button against your cluster. Here “Sandbox” is the new cluster created and the sample dataset loaded.
- Select “Connect you application” to get the url for the MongoDB cluster to connect from Synapse. You will see the error as below saying that the current IP is not added, if you missed adding it as part of the prerequisite “1. MongoDB Atlas cluster setup”, step # 3 “Add your IP to the IP access list”. In our case, this doesn't matter as Synapse will use its set of Ip addresses and that is why we have asked to add “0.0.0.0/0” to the IP whitelist.
- Copy the connection url  and replace <username> with the Database user and <password> with the Database password, that was added as part of prerequisite “1. MongoDB Atlas cluster setup”, step # 4 “Create a Database user”. Make sure the database user has a ”Read and write to any database” built-in role attached.
- Paste the url in the Connection string and type the database name as “sample_mflix” and select “Test Connection” to test the connectivity. This database and its collections were created as part of “1. MongoDB Atlas cluster setup”, step # 7 “Load Sample Data”. You should see a “Connection successful” message indicating that Synapse is able to talk to MongoDB Atlas.
- Select “Create” to create the connection as a new linked service in Synapse. After successful creation of the linked service, it will ask you to select the collection. Select the “movies” collection and select the “OK” button.
- We can see the new linked service with the default name (MongoDbAtlasCollection1) added as the source dataset.
  Also select the “Cursor method” as “limit” and give a value of 10, to limit copying only 10 records from MongoDB Atlas to Synapse ADLS Gen2 storage.

#### Set up ADLS Gen2 as Sink 
- Now that the source is set to MongoDB movies collection, let's set up the Sink to Azure Blob storage. Select the “Sink” tab and select “+ New” against the “Sink dataset” label. Select “Azure Blob Storage” from the list of the New integration datasets and click the “Continue” button.
- Select format as “JSON” for the format in which the blob will be written out. Click “Continue” button to select the JSON format.
- Select “+ New” under “Linked service” for the new JSON based Azure blob storage linked service.
- Select “Azure subscription” as the “Account selection method” and select the ADLS Gen2 we created when setting up the Synapse workspace. (labmdbsynapseadls in the example). Click “Test connection” and on “Connection successful” message , click the “Create” button to create the new linked service.
- Select the folder icon in the “File path” to select the container created in Step. the blobs created by copying from MongoDB Atlas will be stored in this default container. We can create a specific folder in this container using the Data tab. For simplicity , we have kept it in the container itself. Click the “Ok” button and we can see the JSON linked service (Json1) against the “Sink dataset”.

#### Publish Changes and Run the Pipeline
- After all settings are done, select the Publish all button to save all the changes. It will show all the changes, select “Publish” button. It will show that publishing is in progress on top right of the screen.
- To confirm if Publishing is complete , check the notification bell icon on the top right and you can see the “Publishing complete” message.
- We can run the Pipeline in Debug or trigger to see if the data is copied using the Pipeline from MongoDB movies collection to the default container in the default ADLS gen2. Click Debug or Add trigger -> Trigger now icons on the top of the Pipeline. You will see a warning below as we have not set up any parameters. We can ignore the message by clicking the “Ok” button.
- You will see a message that Pipeline is running. You can click that link in the message to see the Pipeline running, or always go to  the “Monitor” tab (on left) and see the Pipeline execution under the “Integration -> Pipeline runs”. Select “Pipeline 1” which should have Succeeded.
- You can see the “Copy data1” stage that was run and its details below in the “Activity runs” section. Select the spectacles icon against the “Copy data1” stage. Note: you will see the icon only when you hover over against “Copy data1” under the the “Activity name”
- This will show the details of the stage, including size of data and rows read and written, along with the time taken and the DIUs consumed.
- To verify that the 10 rows from the “movies” collection got transferred. Go to the “Data” tab (on left), select the “Linked" tab to see all linked services. We can see our default (Primary) Synapse ADLS Gen2 storage (labmdbsynapse in the example) which was created when Synapse workspace was set up. You can also see the default container (defaultprimary in the example). As we didnt create and select a folder or a filename, the Json blob was created directly inside the container and has a default name which is data-<random GUID>.json. Select the file and it will be downloaded to your local machine.
- Open the downloaded file in VSCode and you can see the 10 records copied to Synapse -> ADLS Gen2 container from MongoDB sample_mflix/movies collection.

**Congratulations! You have successfully created a Synapse Pipeline and used teh MongoDB connector to fetch data from MongoDB collection to a blob storage in Synapse. Instead of ADLS Gen2, you can also move to dedicated SQL or other connected data stores in Synapse. You can repeat this exercise to create a new Pipeline and interchange the Source and sink to take the data from the json in Synapse to a MongoDB collection. In this case MongoDB Atlas acts as a sink.**









