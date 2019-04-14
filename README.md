
# HyperVerge Validation API Documentation (beta)

## Overview

This documentation describes the validation API beta. The postman collection can be found [here](https://www.getpostman.com/collections/4ba223a7648215bddb9f). 


- [HyperVerge Validation API Documentation (beta)](#hyperverge-validation-api-documentation-beta)
	- [Overview](#overview)
	- [Schema](#schema)
	- [Parameters](#parameters)
	- [Root Endpoint](#root-endpoint)
	- [Authentication](#authentication)
	- [API Call structure](#api-call-structure)
	- [Request structure](#request-structure)
		- [Validate PAN](#validate-pan)
		- [Validate Aadhaar without QR](#validate-aadhaar-without-qr)
		- [Validate Aadhaar with QR](#validate-aadhaar-with-qr)
		- [Validate Passport](#validate-passport)
		- [Validate VoterID](#validate-voterid)
	- [Response Structure](#response-structure)
	- [Rejection Codes](#rejection-codes)
	- [Error Codes](#error-codes)


## Schema

It is recommended that HTTPS is used for all API calls. For HTTPS, only TLS v1.2 is supported to ensure better security. All data is received as JSON.

## Parameters
All optional and compulsory parameters are passed as part of the request body.

## Root Endpoint
A `GET` request can be issued to the root endpoint to check for successful connection. 

	 curl https://ind-verify.hyperverge.co/api

The `plain/text` reponse of `Aok` should be received.

## Authentication

Currently, an appId, appKey combination is passed in the request header. The appId and appKey are provided on request by the HyperVerge team. If you would like to try the API, please reach out to contact@hyperverge.co

	curl -X POST https://ind-verify.hyperverge.co/\
	  -H 'appid: xxx' \
	  -H 'appkey: yyy' \
	  -H 'content-type: application/json;' \
	  -d '{}' \


On failed attempt with invalid credentials or unauthorized access the HTTP response would have a status code : 401

Please do not expose the appid and appkey on browser applications. In case of a browser application, set up the API calls from the server side.

## API Call structure


* **URL**
  - /api/validatePANInput
  - /api/validatePassportInput
  - /api/validateAadhaarInput
  - /api/validateVoterIdInput

  
* **Method**
	`POST`
	
* **Header**
	* content-type : application/json
	* appId
	* appKey

* **Request Body**
	* *userInput* : Required parameter. JSON of user inputs. The specific keys required for each endpoints are explained  in [this](#request-structure) section.
	* *ocrResultFront* : Required parameter. This is the 'details' JSON from OCR response for front of the document. 
	* *ocrResultBack* : required parameter for validatePassportInput, validateAadhaarInput, validateVoterIdInput. This is the 'details' JSON from OCR response for back of the document. 
	* *qrResult* : JSON object. Optional parameter for validateAadhaarInput. Incase qrResult is present, userInput should not be added.
	* *checkDatabase* : Optional, boolean value, applicable only for PAN and voterID validation.

## Request Structure

### Validate pan
```
{
  userInput: {
    panNumber: <required. String of length 10>,
    name: <required. String parameter>,
    dob: <required. String of format 'DD/MM/YYYY'>
  },
  ocrResult: <required. *details* json from OCR response>
  checkDatabase:<optional. Boolean value, defaults to false. If set to true, NSDL check is made>
}
```

### Validate Aadhaar without QR
```	
{
  userInput: {
    aadhaar: <required string of length 12>,
    name: <required string>,
    dob:  <required string of format 'DD/MM/YYYY'>,
    address: {
      value: <required string. If there are multiple fields, concatenate with '<comma><space>' in the correct order>,
    }
  }
  ocrResultFront: <required. *details* json from the OCR response for aadhaar front>,
  ocrResultBack: <required. *details* json from the OCR response for aadhaar back>,
}
```
	
### Validate Aadhaar with QR	  	
```
{
  qrResult: <required JSON>
  ocrResultFront: <required. *details* json from the OCR response for aadhaar front>,
  ocrResultBack: <required. *details* json from the OCR response for aadhaar back>,
}
```
### Validate Passport
```	
{
  userInput: {
    passportNumber : <required string of length 12>,
    name: <required string>,
    dob:  <required string of format 'DD/MM/YYYY'>,
    address: {
      value: <required string. If there are multiple fields, concatenate with '<comma><space>' in the correct order>,
    }
  }
  ocrResultFront: <required. *details* json from the OCR response for passport front>,
  ocrResultBack: <required. *details* json from the OCR response for passport back>,
}
```
### Validate VoterId
```
{
  userInput: {
    voterId: <required. String of length between 6 and 13>,
    name: <required string>,
    dob:  <required string of format 'DD/MM/YYYY'>,
    address: {
      value: <required string. If there are multiple fields, concatenate with '<comma><space>' in the correct order>,
    }
  }
  ocrResultFront: <required. *details* json from OCR response for voterID front>,
  ocrResultBack: <required. *details* json from OCR response for voterID back>,
  checkDatabase: <optional. Boolean value, defaults to false. If set to true, central DB check is made>
}
```
## Response Structure

* Success Response:

	* Code: 200 

	* Incase of a successful validation, the response would have the following schema.
  ```
  {
    "status" : "success",
    "statusCode" : "200",
    "result" : {
      "validated" : true
    }
  }
  ```
	* Incase the validation failed, the response would have the following schema.
  
  ```		
  {
   "status" : "success",
   "statusCode" : "200",
    "result" : {
      "validated" : false
      "code" : <Rejection code. Refer table in next section>
      "message" : <Rejection reason>
    }
  }
  ```
* Error Responses:
	 
	 * Incase of invalid request, the HTTP code `400` is returned.	 	
  ```	 	
  {
    "status": "failure",
    "statusCode": "400",
    "error": {
      "code" : <Error code. Explained later>
      "message" : <Error message>
    }
  }
  ```

## Rejection Codes

The following are various senarios where validation could fail. 

|Code|Description|
|----|----|
|101|PAN - Name from user input did not match OCR|
|102|PAN - DOB from user input did not match OCR|
|103|PAN - PAN number from user input did not match OCR|
|151|PAN - NSDL check - Duplicat match found|
|152|PAN - NSDL check - Inactive card|
|153|PAN - NSDL check - Name mismatch|
|154|PAN - NSDL check - DOB mismatch|
|155|PAN - NSDL check - PAN number incorrect|
|201|Passport - Name from user input did not match OCR|
|202|Passport - DOB from user input did not match OCR|
|203|Passport - Passport number from user input did not match OCR|
|204|Passport - Address from user input did not match OCR|
|301|Aadhaar - Name from user input did not match OCR|
|302|Aadhaar - DOB from user input did not match OCR|
|303|Aadhaar - Aadhaar number from user input did not match OCR|
|304|Aadhaar - Address from user input did not match OCR |
|351|Aadhaar - Name from QR did not match OCR|
|352|Aadhaar - DOB from QR did not match OCR|
|353|Aadhaar - Aadhaar number from QR did not match OCR|
|354|Aadhaar - Address from QR did not match OCR|
|401|VoterID - Name from user input did not match OCR|
|402|VoterID - DOB from user input did not match OCR|
|403|VoterID - VoterID from user input did not match OCR|
|404|VoterID - Address from user input did not match OCR|
|451|VoterID - Central DB check - Name mismatch|
|452|VoterID - Central DB check - DOB mismatch|
|453|VoterID - Central DB check - VoterID incorrect|
|454|VoterID - Central DB check - Address mismatch|
|999|PAN/VoterID - NSDL/Central DB unavailable|


## Error Codes
The following are the various input errors that could happen. The 'message' and 'path' received in the response body would have more details regarding each error.

|Code|Description|
|----|----|
|1000|'userInput' field is missing|
|1001|Name missing in 'userInput'|
|1002|DOB missing in 'userInput'|
|1003|Document ID missing in 'userInput'|
|1004|Address missing in 'userInput'|
|1010|Validation error with 'ocrResult' field|
|1011|Validation error with 'ocrResultFront' field|
|1012|Validation error with 'ocrResultBack' field|
|1013|Validation error with 'checkDatabase' field|
|1014|Validation error with 'qr' field|
|1099|Input validation error|
