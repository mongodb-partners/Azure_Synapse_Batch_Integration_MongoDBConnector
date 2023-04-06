# MongoDB Atlas and Synapse Integration (Batch)
This repo gives instructions to achieve a Batch/ micro-batch integration between MongoDB Atlas and Synapse.

## Batch/ Micro-batch Integration
In Azure Synapse Analytics, you can integrate MongoDB on-premises instances and MongoDB Atlas as a Source or Sink resource. With historical data, you can retrieve all the data at once. You can also retrieve data incrementally for specific periods by using a filter in batch mode. Then you can use SQL pools and Apache Spark pools in Azure Synapse Analytics to transform and analyze the data. If you need to store the analytics or query results in an analytics data store, you can use the MongoDB sink resource in Azure Synapse Analytics.
![Fig1](https://user-images.githubusercontent.com/104025201/229103014-ca109b01-bdec-466b-8d14-8188c619769a.png)

For more information about how to set up and configure the connectors, see these resources for [MongoDB](https://learn.microsoft.com/en-us/azure/data-factory/connector-mongodb?tabs=data-factory) and [MongoDB Atlas](https://learn.microsoft.com/en-us/azure/data-factory/connector-mongodb-atlas?tabs=data-factory). 

### Pre-requisites:
You will need the below set up before starting the Lab:
1. MongoDB Atlas cluster setup:   
Register for a new Atlas Account [here](https://www.mongodb.com/docs/atlas/tutorial/create-atlas-account/#register-a-new-service-account).   
Follow steps from 1 to 4 (*Create an Atlas account*, *Deploy a Free cluster*, *Add your IP to the IP access list* and *Create a Database user*) to set up the Atlas environment.   
Also, follow step 7 “*Load Sample Data*” to load sample data to be used in the lab.

![Picture 2](https://user-images.githubusercontent.com/104025201/230300219-6f95d9be-616f-4267-8cce-e4d3af5d1411.png)

**Note: For this lab add “0.0.0.0/0” to the IP access list so that Synapse can connect to MongoDB Atlas. In Production scenarios, it is recommended to use Vnet Peering or Private Link options.**

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
- Open the newly created Synapse workspace and navigate to the Synapse Studio by selecting “*Open Synapse Studio*”
<img width="454" alt="fig2" src="https://user-images.githubusercontent.com/104025201/229139179-4132ef32-1264-4439-ae14-598a05186430.png">

- Navigate to the “*Integrate*” tab and select “+” sign and “*Pipeline*” to create a new Synapse Pipeline.
<img width="454" alt="fig3" src="https://user-images.githubusercontent.com/104025201/229139264-14eb20cf-f896-424f-a26a-b931c2b726e2.png">

- Drag and Drop “*Copy data*” activity from the “*Move & transform*” tile to the center plane to add the “*Copy data*" activity into the new Pipeline named “*Pipeline 1*” by default.
<img width="400" alt="fig4" src="https://user-images.githubusercontent.com/104025201/229139335-a9b1191d-015c-44b2-9499-5b45c97b9bd8.png">


#### Set up Atlas as Source
- Select the “*Source*” tab from the bottom plane and click on the “*+ New*” button against the “*Source dataset*” label.
<img width="416" alt="fig21" src="https://user-images.githubusercontent.com/104025201/229141328-725a990a-87e8-46cb-b03d-0a839ba29893.png">

- Search for “*mongodb*” in the box against the label “*Select a data store*” . Select “*MongoDB Atlas*” and press the “*Continue*” button.
<img width="416" alt="fig22" src="https://user-images.githubusercontent.com/104025201/229141365-0793915e-3f8d-45b8-893d-136c26bea59f.png">

- Select “*New*” against the “*Linked Service*” box to create a new Atlas cluster integration.
<img width="331" alt="fig23" src="https://user-images.githubusercontent.com/104025201/229141424-14c12968-2b67-411b-b72d-63ef8efe8024.png">

- In the MongoDB Atlas “*Database*” view (on left) under “*Data Services*” (on top) Select “*Connect*” button against your cluster. Here “*Sandbox*” is the new cluster created and the sample dataset loaded.
<img width="385" alt="fig24" src="https://user-images.githubusercontent.com/104025201/229141517-c8022be5-fa5e-4d90-8847-99385df49249.png">

- Select “*Connect you application*” to get the url for the MongoDB cluster to connect from Synapse. You will see the error as below saying that the current IP is not added, if you missed adding it as part of the prerequisite “*1. MongoDB Atlas cluster setup*”, step # 3 “*Add your IP to the IP access list*”. In our case, this doesn't matter as Synapse will use its set of Ip addresses and that is why we have asked to add “0.0.0.0/0” to the IP whitelist.
<img width="390" alt="fig25" src="https://user-images.githubusercontent.com/104025201/229141604-66b16475-f804-4f2b-b506-57a5d87fc537.png">

- Copy the connection url  and replace &lt; username &gt; with the Database user and &lt; password &gt; with the Database password, that was added as part of prerequisite “*1. MongoDB Atlas cluster setup*”, step # 4 “*Create a Database user*”. Make sure the database user has a ”*Read and write to any database*” built-in role attached.
<img width="400" alt="fig26" src="https://user-images.githubusercontent.com/104025201/229141741-bfaffa70-9a81-4cab-a19e-775dd47bda54.png">
  
- Paste the url in the Connection string and type the database name as “*sample_mflix*” and select “*Test Connection*” to test the connectivity. This database and its collections were created as part of “1. MongoDB Atlas cluster setup”, step # 7 “*Load Sample Data*”. You should see a “*Connection successful*” message indicating that Synapse is able to talk to MongoDB Atlas.
<img width="298" alt="fig27" src="https://user-images.githubusercontent.com/104025201/229141814-27ef6178-4bda-4e16-a3ab-dcfb78bb51e8.png">

- Select “Create” to create the connection as a new linked service in Synapse. After successful creation of the linked service, it will ask you to select the collection. Select the “*movies*” collection and select the “OK” button.
<img width="296" alt="fig29" src="https://user-images.githubusercontent.com/104025201/229143389-ca73e560-bcd9-47fe-a5b0-50951f4fc4d0.png">
  
- We can see the new linked service with the default name (MongoDbAtlasCollection1) added as the source dataset.
<img width="282" alt="fig210" src="https://user-images.githubusercontent.com/104025201/229142187-70dec889-0577-4963-a7e3-03af0595aff0.png">

Also select the “*Cursor method*” as “*limitv” and give a value of 10, to limit copying only 10 records from MongoDB Atlas to Synapse ADLS Gen2 storage.

<img width="281" alt="fig211" src="https://user-images.githubusercontent.com/104025201/229142252-ea7db1a3-33cc-4176-bb52-16d55e82ae27.png">

  
#### Set up ADLS Gen2 as Sink 
- Now that the source is set to MongoDB movies collection, let's set up the Sink to Azure Blob storage. Select the “*Sink*” tab and select “*+ New*” against the “*Sink datasetv” label. Select “*Azure Blob Storage*” from the list of the New integration datasets and click the “*Continue*” button.
<img width="191" alt="fig31" src="https://user-images.githubusercontent.com/104025201/229145382-124968d3-543e-430c-b730-e992268c592f.png">
  
- Select format as “*JSON*” for the format in which the blob will be written out. Click “*Continue*” button to select the JSON format.
<img width="258" alt="fig32" src="https://user-images.githubusercontent.com/104025201/229145465-2175f49e-84fc-4e73-bef4-cc9c164ddcde.png">

- Select “*+ New*” under “*Linked service*” for the new JSON based Azure blob storage linked service.

Select “*Azure subscription*” as the “Account selection method” and select the ADLS Gen2 we created when setting up the Synapse workspace. (labmdbsynapseadls in the example). Click “Test connection” and on “*Connection successful*” message , click the “*Create*” button to create the new linked service.

<img width="280" alt="fig33" src="https://user-images.githubusercontent.com/104025201/229145511-777d9d28-7775-413c-95d0-708d7c5784be.png">

- Select the **folder icon** in the “*File path*” to select the container created in Step. the blobs created by copying from MongoDB Atlas will be stored in this default container. We can create a specific folder in this container using the Data tab. For simplicity , we have kept it in the container itself. Click the “*Ok*” button and we can see the JSON linked service (Json1) against the “*Sink dataset*”.
<img width="313" alt="fig35" src="https://user-images.githubusercontent.com/104025201/229146084-d0822f3e-af68-4097-9a22-ae76f5089a55.png">


#### Publish Changes and Run the Pipeline
- After all settings are done, select the Publish all button to save all the changes. It will show all the changes, select “*Publish*” button. It will show that publishing is in progress on top right of the screen.
<img width="436" alt="fig41" src="https://user-images.githubusercontent.com/104025201/229148079-e3c71cff-6b05-453d-b2e3-dd1dd99b78dd.png">

- To confirm if Publishing is complete , check the notification bell icon on the top right and you can see the “*Publishing complete*” message.
<img width="234" alt="fig42" src="https://user-images.githubusercontent.com/104025201/229148121-589cc43a-ae97-4ff7-9c54-72b83d827332.png">

- We can run the Pipeline in Debug or trigger to see if the data is copied using the Pipeline from MongoDB movies collection to the default container in the default ADLS gen2. Click Debug or Add trigger -> Trigger now icons on the top of the Pipeline. You will see a warning below as we have not set up any parameters. We can ignore the message by clicking the “*Ok*” button.
<img width="253" alt="fig43" src="https://user-images.githubusercontent.com/104025201/229148217-d4b5d629-30f6-44f9-9dab-7146741cd88c.png">

- You will see a message that Pipeline is running. You can click that link in the message to see the Pipeline running, or always go to  the “*Monitor*” tab (on left) and see the Pipeline execution under the “*Integration -> Pipeline runs*”. Select “*Pipeline 1*” which should have Succeeded.
<img width="454" alt="fig44" src="https://user-images.githubusercontent.com/104025201/229148260-785d2069-e7a6-4d8a-bbec-84a3a2a754d7.png">

- You can see the “*Copy data1*” stage that was run and its details below in the “*Activity runs*” section. Select the spectacles icon against the “*Copy data1*” stage. Note: you will see the icon only when you hover over against “*Copy data1*” under the the “*Activity name*”
<img width="451" alt="fig45" src="https://user-images.githubusercontent.com/104025201/229148290-699ec1a9-49a5-45be-b303-cb754528014b.png">

- This will show the details of the stage, including size of data and rows read and written, along with the time taken and the DIUs consumed.
<img width="401" alt="fig46" src="https://user-images.githubusercontent.com/104025201/229148351-cf2bfb2e-067d-430b-9edd-44431d8b80b0.png">

- To verify that the 10 rows from the “*movies*” collection got transferred. Go to the “*Data*” tab (on left), select the “*Linked*" tab to see all linked services. We can see our default (Primary) Synapse ADLS Gen2 storage (labmdbsynapse in the example) which was created when Synapse workspace was set up. You can also see the default container (defaultprimary in the example). As we didnt create and select a folder or a filename, the Json blob was created directly inside the container and has a default name which is data-<random GUID>.json. Select the file and it will be downloaded to your local machine.
<img width="454" alt="fig47" src="https://user-images.githubusercontent.com/104025201/229148387-fd2d5220-a4fb-4d59-81c7-7fcc05f1ecd6.png">

- Open the downloaded file in VSCode and you can see the 10 records copied to Synapse -> ADLS Gen2 container from MongoDB sample_mflix/movies collection.
<img width="454" alt="fig48" src="https://user-images.githubusercontent.com/104025201/229148419-cb2f40f9-951d-4977-870a-21c846573389.png">


**Congratulations! You have successfully created a Synapse Pipeline and used teh MongoDB connector to fetch data from MongoDB collection to a blob storage in Synapse. Instead of ADLS Gen2, you can also move to dedicated SQL or other connected data stores in Synapse. You can repeat this exercise to create a new Pipeline and interchange the Source and sink to take the data from the json in Synapse to a MongoDB collection. In this case MongoDB Atlas acts as a sink.**









