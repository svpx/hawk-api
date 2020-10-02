:warning: This is just a public repo for sharing documentation. Original API is under development and is in a private repo. This is a work in progress. 
If you have any suggestions, feedback etc. Please see the [Contact](#11-get-in-touch) section and feel free to write me an email. 
<br>
<br>
<br>
<img src="https://www.flaticon.com/svg/static/icons/svg/47/47294.svg" width="75" height="75">

[![version](https://img.shields.io/badge/v1.0.0-hawk-red?style=plastic)](https://semver.org)
# Hawk - API 
REST API for EU Horizon project Easy Crowd Go.


## 1. Description

The API is written to manage auto-registration, data logging and de-registration of IoT sensors for monitoring crowd movement using computer vision-based sensors and WiFi Kismet Library based sensors. The sensor goes through a self-registration on power on by calling the registration endpoint and can then begin to log data into the database with an assigned internal ID. The API will be used to communicate between the frontend dashboard and the database. 


## 2. Function

The API offers the following functionalities for management and data logging from sensors.


- Sensor Registration and De-registration
- Get all registered Sensors
- Get sensor by type or Internal ID
- Get all data 
- Get data for a sensor type 
- Delete a sensor



## 3. Architecture

The API is segmented to keep a clean architecture (though a work in progress). The idea is to create a clear separation between different functionalities and keep the "index.js" as clean and readable as possible. In this API, index.js is used to route to correct paths and send a response, while all the processing happens in the helper functions. 


- index.js  _// Main file used for routing._
- helper functions
  - sensorSchema.js **--->** _Used to define mongoose schema for sensor registration._ 
  - logSchema.js **--->**   _Used to define logging schema for data logging._
  - apiSchema.js **--->**   _Used to define the api_key schema for storing the and checking the api_keys in the MongoDB_
  - sensorRegistry.js **--->**   _Handles initial registration of the sensor and stores in the collection of the sensor._ 
  - logHandler.js **--->** _Handles data logging for all the sensors._
  - api_key.js **--->**  _Contains the API Validation function which returns a boolean._
  - requestHandler **--->**  _Handles all the Get request for getting sensor registration data, deleting sensors and getting all the data._
    - getSensor()
    - getData()
    - deleteSensor()
    
 ## 4. API Key 
 
 The API call can only be made with a valid API Key. The key is passed as a query string for any call made to the API. In an event of an invalid API, the call will be rejected.
 ### 4.1 Parameters
 The API key can be passed as a query string using the following parameter,
 - **api_key** - Please ask the admin for API Key before testing the API. See example below <br>
 `/hawk/api/v01/sensor/api_key=YOUR_API_KEY`
 
  ### 4.2 Responses
  - **Success** - On validation the call will respond with the right values/data.
  - **Failure** - On failure, <br> `Message: The API Key you provided could not be validated.`
 
 ## 5 Sensor Registration & Data Logging
 The sensor registration can be done by making a **POST Request** with parameters and JSON body with the following schema.
 
 ### 5.1 API Enpoint
 `/hawk/api/v01/sensor`
 
 ### 5.2 Parameters
 The post request takes two parameters,
 - **reg** - invoke the parameter to call registration function and register a sensor. <br>
 `/hawk/api/v01/sensor?api_key="YOUR_API_KEY"&type=reg`
 
 - **log** - invoke the parameter to start logging data from the sensors. Check the Log Data section below for the Data Schema. <br>
 `/hawk/api/v01/sensor?api_key="YOUR_API_KEY"&type=log`
 
 
### 5.3 Registering a sensor

### 5.3.1 API endpoint and Param
`POST` <br>
`/hawk/api/v01/sensor?api_key=YOUR_API_KEY&reg=true`

 #### 5.3.1.1 Parameters
 To register a sensor the API endpoint should contain the following parameter in addition to the API Key. 
 - **reg** - The parameter "reg" should be set to true. See example below <br>
 `/hawk/api/v01/sensor/api_key=YOUR_API_KEY&lreg=true`
 
 ### 5.3.2 Sensor MongoDB Schema for registration.
 
 Please observe the following schema for JSON payload while making the POST request. (Please see the sub-section [5.3.3 Method](#533-method) for example code)
 
 - **lat** - Latitude of the sensor from the GPS
 - **long** - Longitude of the sensor from the GPS
 - **internalId** - Internal assigned ID of the sensor. Unique value. Duplicates will not be allowed. 
 - **status** - Log running status.
 - **sensorType** - between WIFI and CV, used for advanced filtering. 
 - **timestamps** - Auto-generated. 
 
 ```javascript
 const sensorSchema = new mongoose.Schema({
  lat: { type: Number, required: true},
  long: { type: Number, required: true},
  internalId: { type: String, required: true,},
  status: { type: String },
  sensorType: { type: String, required: true }
}, {
  timestamps: {createdAt: 'created_at'}
});
```

### 5.3.3 Method
To register a sensor make <strong> POST request</strong> to the API end point with parameter "reg" set to true( eg. `/hawk/api/v01/sensor?api_key=KEY&reg=true` ) with JSON payload following the MongoDB sensor schema above. 

:warning: When using **Postman** for testing API, the float values (for GPS) are truncated at the decimal point. Please report at sp@suryaveerpatnaik.com if similar issues are found or if there is fix that I am unaware of. 

 ```javascript
// Example of the JSON 
{
"lat" : 52.52021,
"long" : 13.25021,
"internalId" : "CV001-BER",
"status" : "Active",
"sensorType" : "CV"
}
```
An example POST request made using Python and Requests Library. 

```python
import requests

url = "http://localhost:xxxx/hawk/api/v01/sensor?api_key=YOUR_API_KEY&reg=true"

payload = "{
            \r\n    \"lat\": 52.52201,\r\n    \"long\": 13.25447,\r\n    
            \"internalId\": \"CV003BER\",\r\n    \"status\": \"Active\",\r\n    
            \"sensorType\": \"CV\"\r\n
            }"
            
headers = {
  'Content-Type': 'application/json'
}

response = requests.request("POST", url, headers=headers, data = payload)

print(response.text.encode('utf8'))
```

### 5.3.4 **Responses**
- Success Message - Registration Successful 
- Failure Message - Error Message from the callback is sent as a response. <br>
```javascript
//Ex. Response of a failed POST request to register without an internalId 
CountSensor validation failed: internalId: Path `internalId` is required.
```

## 5.4 Logging Sensor Data

### 5.4.1 API endpoint and Param
`POST` <br>
`/hawk/api/v01/sensor?api_key=YOUR_API_KEY&log=true`

### 5.4.2 Logging Data to MongoDB Collection

 Please observe the following schema for JSON payload while making the POST request. (Please see the sub-section [5.4.3 Method](#543-method) for example code)

 - **internalId** - Internal assigned ID of the sensor. Unique value. - Required Value
 - **sensorType** - between WIFI and CV, used for advanced filtering. - REquired Value
 - **loggingData** - Object, recieved as data from the sensor. - Not explicitly required to pin-point errors.
 - **timestamps** - Auto-generated. 
 
 ```javascript
const logSchema = new mongoose.Schema({
  internalId: { type: String, required: true },
  sensorType: { type: String, required: true },
  loggingData: { type: Object}
}, {
  timestamps: { createdAt: 'created_at'} //Auto-generated
});
```
### 5.4.3 Method
After the sensor registration is done, a POST request with the parameter "log" set to "true" can be made to the API endpoint `/hawk/API/v01/sensor?api_key=YOUR_API_KEY&log=true` with the data as JSON payload. Please follow the log schema to avoid errors in logging. 

:warning: **The sensor data must be packaged as an object.**

```javascript
{
    "internalId": "CV001BER",
    "sensorType": "WIFI",
    "loggingData": {
        "total_in": 4,
        "total_out": 5
    }
}
```

An example POST request made using Python and Requests Library. 

```python

import requests

url = "http://localhost:4500/hawk/api/v01/sensor?api_key=xh7gz8gkewfbfcj2c63meamalyqrhe&log=true"

payload = "{\r\n    \"sensorId\": \"CV001BER\",\r\n    \"sensorType\": \"WIFI\",\r\n    
            \"loggingData\": {\r\n        \"total_in\": 4,\r\n        
            \"total_out\": 5\r\n    }\r\n
            }"

headers = {
  'Content-Type': 'application/json'
}

response = requests.request("POST", url, headers=headers, data = payload)

print(response.text.encode('utf8'))
```

## 6 Get Registered Sensors
A list of all the registered sensors can be obtained from the API.

### 6.1 API Endpoint and Params

`GET` <br>
`/hawk/api/v01/sensor?api_key=YOUR_API_KEY`

 ### 6.1.1 Parameters

The API endpoint takes two parameters in the query string, one of them is optional. 

- **type** - takes two option 
  - all -- Gets all the sensors registered in the Databse
  - one -- Gets one sensor based on the Internal ID parameter
  
- **internalId** -- Required parameter when **type** is set to **one**
  
- **stype** - Optional Parameter, takes two options.   _(:warning: Can only be used when **type** is set to **all**)_
  - CV -- for Computer Vision sensors
  - WIFI --for WIFI Sensors
  
  
  
 ### 6.2 Get all registered Sensors
  
  To get a list of all the sensors registered in the database collection. Make a GET request to the following API endpoint with the params and your API Key. 
  `Ex. /hawk/api/v01/sensor?api_key=YOUR_API_KEY&type=all`
  
  Example response:
  ```javascript
   {
            "_id" : "5f74d97c3d0b543708e9d120",
            "lat" : 46.52005,
            "long" : 13.2522,
            "internalId" : "CV002BER",
            "sensorType" : "WIFI"
            "status" : "Running",
            "__v" : 0
        },
        {
            "_id" : "5f74d9ea8566853c18916f6d",
            "lat" : 46.52477,
            "long" : 18.55245654,
            "internalId" : "CV003BER",
            "status" : "Running",
            "__v": 0
   }
  ```
  
### 6.3 Get all registered Sensors by type

It is possible to get a list of all the sensors by their type, either WIFI or CV. To get sensor by types make a GET request to the API endpoint in section [6.1 API Endpoint](#61-API-endpoint-and-params)<br>
`Ex. /hawk/api/v01/sensor?api_key=YOUR_API_KEY&type=all&stype=WIFI`

### 6.4 Get one sensor by Internal ID

To check if a particular sensor is registered or to check any other details with respect to the sensor, a GET request with the parameter **type** set to **one** and **internalId** set to the Internal ID of the sensor can be called. <br>
`Ex. /hawk/api/v01/sesnor/?api_key=YOUR_API_KEY&type=one&internalId=CV001BER`


## 7. Get Data

The data logged at the MongoDB can be queried by calling the API endpoint for Data. Data can be retrieved for all the sensor, or for a specific type or a specific sensor. 

### 7.1 API Endpoint and Params

:warning: Please pay attention to the API endpoint.

`GET` <br>
`/hawk/api/v01/data?api_key=YOUR_API_KEY`

 ### 7.1.1 Parameters

The API endpoint takes two parameters in the query string, one of them is optional. 

- **type** - takes two option 
  - all -- Gets all the data of registered sensors in the Database
  - one -- Gets data from one sensor based on the Internal ID parameter
  
- **internalId** -- Required parameter when **type** is set to **one**
  
- **stype** - Optional Parameter, takes two options.   _(:warning: Can only be used when **type** is set to **all**)_
  - CV -- for Computer Vision sensors
  - WIFI --for WIFI Sensors


 ### 7.2 Get all data
  
  To get a list of all the sensors registered in the database collection. Make a GET request to the following API endpoint with the params and your API Key. 
  `Ex. /hawk/api/v01/data?api_key=YOUR_API_KEY&type=all`

Sample response
```javascript
        {
            "_id": "5f75d7805732f45e2c8a75ee",
            "sensorId": "CV001BER",
            "sensorType": "CV",
            "loggingData": {
                "total_in": 45,
                "total_out": 47
            },
            "created_at": "2020-10-01T07:38:08.536Z",
            "updatedAt": "2020-10-01T07:38:08.536Z",
            "__v": 0
        },
```

### 7.3 Get all data by sensors type

It is possible to get data by the sensor type, either WIFI or CV. It works on the same principle as GET sensor list.  To get data by the sensor by types make a GET request to the API endpoint in section [7.1 API Endpoint and Params](#71-api-endpoint-and-params)<br>
`Ex. /hawk/api/v01/sensor?api_key=YOUR_API_KEY&type=all&stype=WIFI`

### 7.4 Get data of one sensor by Internal ID

To get the data of one particular sensor from the database, a GET request with the parameter **type** set to **one** and **internalId** set to the Internal ID of the sensor can be called. <br>
`Ex. /hawk/api/v01/data/?api_key=YOUR_API_KEY&type=one&internalId=CV001BER`

## 8. De-registration and Deletion

For the sake of simplicity, any sensor can be deleted by making a POST request to the API endpoint with the internalId of the sensor to be de-registered. 
Every sensor should initial deregistration when the shutdown sequence is initiated for the sensor. 

### 8.1 API Endpoint and Params

:warning: An internal ID of the sensor must be provided to remove 

`DELETE`<br>
`/hawk/api/v01/delete`

#### 8.1.1 Parameters

The API endpoint takes one query string parameter
- **internalId** - Provide the internal assigned ID of the Sensor to remove it from the list. 

### 8.2 Delete a sensor.

To delete a sensor make a DELETE request to the API endpoint with the sensor internal ID as the query string. <br>
`Ex. /hawk/api/v01/delete/api_key=YOUR_API_KEY&internalId=CV001BER`

:warning: In the future, the API endpoint for deletion will change. Scheduled for an update in the coming months. 

## 9. Current Status

- The API has not been deployed yet. The next step is to deploy the API on the AWS EC2 or Heroku service.
- Scheduling testing with the hardware team 

## 10. Future Updates

- Code refactoring
- Feature addition 
  - UPDATE
  - PATCH
  - PUT 
- Cleaning up the documentation, add content section
- Code consistency check
- Adding stronger validation and request limiter

## 11. Get in touch

If you want to know more about the project or want to enquire about anything else related to the project, please feel free to write me an email at sp@suryaveerpatnaik.com


