# Exercise 5: Sending Device Data via Rest API

## Description
In this exercise we create a Rest API service that can be called by devices to save the performance data to a file. This could be the very beginning of the whole use case you just accomplished so far.  
1. Devices are sending data to SAP Data Intelligence
2. Data is written to the HANA database
3. Quality assurance test verifies if the data is within the expected boundaries.
4. All failed data are recorded to a `QMTICKET` table to rectify the data or resolve any issues with devices
5. Further analysis by a data scientist could develop more sophisticated quality checks and create and even better prediction on failed data


## Exercise Summary

Creating a Pipeline with a web-service that can be called externally and hand over the data to an operator for further processing before calling an operator for saving the data.

In order to test the Rest API service a client application is provided which can generate and send example data to web service. You may alternatively use the Postman application or the terminal command `curl`.


## Exercise 5.1
Each running pipeline consumes a node in the cluster. If you are using a shared clusters with many users it is highly encouraged to ensure that the web-service pipeline is only running as short as possible. For that reason we will design the pipeline to terminate as as soon as it has received a new record and appended it to a file.

Creating the basic RestAPI:

1. Create new Graph
2. Add the RestAPI operator **OpenAPI Servlow** and the **Wiretap** operator
3. Configure the **OpenAPI Servlow** operator
	1.  **Base Path**: `teched2020/<your_username>_performance` - This defines the path on which the web server can be accessed
	2. **One-Way**: `True` (Operator will not return a response to the client and instead sends immediately HTTP 204 back)
4. Add the **To File** converter and the **Write File** operator, then connect them
5. Configure the **Write File** operator for adding the data to your `performance.csv`:
	1. **Connection**: `DI_DATA_LAKE`
	2. **Path mode**: Static (from configuration)
	3. **Path**: `/shared/<your_username>/performance.csv`
	4. **Mode**: Append
6. Add the **Workflow Terminator** to the graph. Of course if this was pipeline running productively we would not use a terminator and the graph would run perpetually
7. Connect all operators  
8. Save the pipeline as `<your_username>.DeviceRestAPI`

![restapi](./images/restapi1.png)

#### Test

1. Now you can start the pipeline
2. When the pipeline is running send a sample HTTP POST request
3. The received request will be saved to the target file/directory and then the pipeline should gracefully terminate
4. Check the contents of file using Metadata Explorer.


### HTML Test Page

The easiest way to test the Rest API is using the small web application we have developed for this particular tutorial [https://sendcelldata.cfapps.eu10.hana.ondemand.com](https://sendcelldata.cfapps.eu10.hana.ondemand.com).

Be sure to use the tenant name as a prefix and your username e.g. `TAxx` or `system` as credentials e.g. `default\system`

Note, that the URL must end with a forward slash ( / ) otherwise you will get the error **`RestAPI not running!`**

![Test Page](./images/TestRestAPI.png)


### (Optional/Alternative) Using Postman to test REST API
#### Request URL


Substitute the URL and the user ID and send a POST request to: `https://<url pipeline modeler>/openapi/service/teched2020/<your_username>_performance/`

It is important to add a process-tag at the end, otherwise the request gets an error although the process tag is not used.

#### Request Header
In the "Headers"-tab add the following header: `X-Requested-With` :  `XMLHttpRequest`.

Without this parameter you get the error: `Forbidden cross-site request`

![postman1](./images/postman1.png)

#### Authorization
1. Change the authorization TYPE: Basic Auth
2. The username must be prefixed with the tenant ID followed by a backslash and then the username e.g. `default\taXX`

![postman3](./images/postman3.png)

#### Request Body

In the body you can add the actual data that should be posted. Here we can add already a JSON that contains the data of the device:

`{"TIMESTAMP":"2020-10-19 20:06:55","CELLID":1234512,"KEY1":111.1,"KEY2":222.2}`

![postman2](./images/postman2.png)


## Summary

In this final exercise you have learnt how to setup and send data -- using various HTTP clients -- to a Rest API web service hosted inside SAP Data Intelligence.
