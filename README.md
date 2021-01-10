# How to automate API with Postman

## Starter kit project source code

The project is published on Postman site with URL is [Automation Starter Kit Project](https://documenter.getpostman.com/view/13308392/TVes7mps)

## Assumption

You have basic knowledge about Postman.
You have development skill on javascript or any of the others programming language but not require.
You know what is an Restful Api and how to test that.

## What is a Restful Api

Restful Api is one of the most common think you should know when you are a quality engineer (QE). In addition, QE mean manual tester or  automation tester. If you do not have any about "What is the Restful Api" please visit <https://restfulapi.net>.

Basically, Restful Api contains some informations consist of Url, method, header, param and body. All of that information will be sent to an application. Then, the application will response informations that contain status code and response body.

## How to test a Restful Api

Restful Api basically is about sending request to an application and receiving the response. Therefore, testing a Restful Api is just verify that when I send the request with specific request, the system will response back the information as I expected. For example, we have to test a Calculation Restful Api that will receive the request and response calculated number. For more details, the request consists of method, Url and 2 params. 2 params are NumberA and NumberB that are 2 number send to the system and expect the system will response the sum of that two variable (NumberC = NumberA + NumberB).

To test the Calculation Restful Api, we have to design some test case to verify is that system work correctly. There are some basic test cases we can create to test that system. Firstly, We create the valid case about input value 1 for NumberA and 2 for NumberB so the response NumberC should be 3 and the status code is 200. Secondly, we create the test case that the numberA is 1 and the NumberB is "Two" so the response should show error message "The numberB is not a number." and the status code is 400. Finally, We need to check the method of request that the system will response error message "Not found " and status code is 404 if we input the wrong request's method.

In conclusion, Testing Restful Api is about create the test cases that ideally cover all of aspects of that Api.

## How to test an Api test case manually by Postman

To be clear, there are many type of Api but this document just focus on Restful Api so I will call the Restful Api with name is Api.

The previous section, I give you some test cases to test an Api such as happy case, validation case, but there is just one way to implement an Api test case. Therefore, I will go to detail in just an Api test cases. The Api test case that I mention in this section consist of:

- Api test case name: Get User information successfully
- Api request information:
  - Name: Get user by userId
  - Method: Get
  - Url: <https://5f9cbdef6dc8300016d3139b.mockapi.io/w33/users/{{UserId}}> (An mock Api from MockApi.com)
- Api response information:
  - Status code: 200
  - Response body: contains UserId

Note: UserId is a number that is Id of user. So, the UserId must be an Id of a valid user.

To manual test this test case, we will following this steps:

- Open Postman
- Create new Collection (Example: **W33Automate - Demo**)
- Create new folder inside **W33Automate - Demo** to contain all test cases for one Api testing type (Example: **Functional Testing**).
- Create new folder inside **Functional Testing** to contain all test cases for one Api site (Example: **MockApi**).
- Create new folder inside MockApi to contain test case **GetUserByIdSuccessfully**.
- Create new request inside **GetUserByIdSuccessfully** to execute the get-user-by-userid-Api. The request consists of:
  - Method: Get
  - Url: <https://5f9cbdef6dc8300016d3139b.mockapi.io/w33/users/50> (Suppose that 50 is an valid UserId).
- Execute the request **GetUserByIdSuccessfully** by Send button.
- Verify manually by view result on Postman API result

There are all the steps I will following to test an API test case and I work perfectly.

## How to automate your manual test script - First step

Suppose that you have already implemented a **GetUserByIdSuccessfully** test case and you feel that verify the result by your own eyes is so painful. I will guy you to do a small step to make the Postman tools verify the result automatically. To verify the status code and the response body, you open the Test tab and write some Javascript script (**Don't worry. It is so easy!**). We will have 2 small scripts to verify 2 acceptance criterial into Test tab.

- Verify the status code is 200.

```js
pm.test("Status code is 200", function () {
  pm.response.to.have.status(200);
});
```

- Verify that the response body contain the UserId.

```js
var UserId = 50;
pm.test("Your test name", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.id).to.eql(UserId);
});
```

Reference: <https://learning.postman.com/docs/writing-scripts/script-references/postman-sandbox-api-reference/>

## How to automate manual test script following standard

The standard is just create by myself after some research of other automation test scripts on internet. There are some standard I declared:

- The variable that will change base on environment like baseUrl must be stored in Environment Variable
- The structure of an automation test script:
  - The Collection is the highest level that is the root folder of the scripts
  - The folders below the Collection is test suits
  - The folders below the test suits is test cases
  - Finally, the requests inside the test cases folders is the place that we will implement the automation script for the test cases.
- Each test case should verify the **Status code** and the **response body** to make sure it retrieve the right data.
- Every variable must be declare by script instead of creating manually except Environment  and Global variable to make sure the test cases not base on each other in term of variable aspect.
- When we want to call API to setup or tear down the script, we should call it by script in Pre-request Script and Tests.

We start to automate the manual test case base on previous step following this standard (Continue automate after done the **first step**). Firstly, we will create an Environment variable is **BaseUrl** to stored its site base URL.Then we use it in the Url of the request. The BaseUrl can be change when the environment is changed. For example, we change the application's environment from SIT to UAT, so we just adjust the BaseUrl to make it run correctly. Secondly, The valid UserId = 50 is myth, We should make the absolutely value UserId by call the Create User API. We just put this source code into Pre-request Script to create new User and put the UserId to collection Variable

```js
var baseUrl = pm.environment.get("BaseUrl");
var createUrl = baseUrl + "/users";
const createRequest = {
  url: createUrl,
  method: 'POST',
  header: {
    'Content-Type': 'application/json',
    'X-Foo': 'bar'
  }
};

pm.sendRequest(createRequest, (error, response) => {
  pm.collectionVariables.set("UserId",response.json().id);
  console.log(error ? error : response.json());
});

```

Then we can put the UserId into the URL to use and put into the verifying response body

```js
URL: {{BaseUrl}}/users/{{UserId}}
```

```js
var UserId = pm.collectionVariables.get("UserId");
pm.test("Body matches string", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.id).to.eql(UserId);
});
```

Then we create create verify script like the First Step section.

```js
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

var UserId = pm.collectionVariables.get("UserId");
pm.test("Body matches string", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.id).to.eql(UserId);
});
```

Finally, We remove the User that was created throw the test case by call the Delete API in the Tests.

```js
//Tear down
//To remove the user that was created in this test case
var baseUrl = pm.environment.get("BaseUrl");
var userIdDelete = pm.collectionVariables.get("UserId");
var deleteUrl = baseUrl + "/users/" + userIdDelete;
const deleteRequest = {
  url: deleteUrl,
  method: 'DELETE',
  header: {
    'Content-Type': 'application/json',
    'X-Foo': 'bar'
  }
};

pm.sendRequest(deleteRequest, (error, response) => {
  console.log(error ? error : response.json());
});
```

## How to automate the manual script following data driven type

To automate the manual script following data driven type, we are going to adjust the manual test script. Firstly, we add the BaseUrl environment variable like the previous section (or just use its environment variable). Secondly, we add the verify script into **Tests**

```js
var Status = parseInt(pm.variables.get("Status"));
pm.test("Status code is 200", function () {
    pm.response.to.have.status(Status);
});

pm.test("Body matches string", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.id).to.eql(data.UserId + "");
});
```

These scripts is 2 ways to get variable from CSV file

```js
//To get status
pm.variables.get("Status");
//To get UserId
data.UserId;
```

We use the User.csv file to run the script. The User.csv file consists of 2 column are UserId and Status. And, It contains 4 rows data for that  2 columns. These 4 rows data will make 4 iteration when run it.
To run the test script, we click on **Runner** ,and when the Collection Runner is open, we select the **collection**, **environment variable** and **CSV file** the click **Run...**. It will run that test case in 4 times with 4 different sets of values.

## Run collection and get report by newman

Automated reporting is one of the most important parts when we do the automation testing. With Postman, there are an tool base on Node.js that support our to automatically generate a report after the test done the execution. That is **newman-reporter-htmlextra**!
To use the newman-reporter-htmlextra we should follow these steps:

- Download and install [Node.js](https://nodejs.org/en/download/)
- Install newman by cmd: npm install -g newman
- Install newman-reporter-htmlextra by cmd: npm install -g newman-reporter-htmlextra

To run and generate HTML report by newman-reporter-htmlextra, we run this command line **newman run W33Collection.json -r htmlextra** with W33Collection.json is the collection that you want to execute. In addition, if your script need environment file, you just run this command line **newman run W33Collection.json -e W33Env.json -r htmlextra** with W33Env.json is the environment file you want to run. To run with CSV file you use this command line: **newman run W33CollectionDataDriven.json -e W33Env.json -d "User.csv" -r htmlextra**

Reference: <https://www.npmjs.com/package/newman-reporter-htmlextra>

## CIDI with Jenkins

We assume that Jenkins is already setup on our machine and the Node.js and Newman is installed in that machine.

We will go throw to know how to run automation test script with Postman on the Jenkins. Firstly, we create new **Freestyle project** with name **W33_Postman_Api_Automation_Test**. Then, we chose Source Code Management with Git by inputing Repository URL and Credentials. After setup source code repository we add **Execute Windows batch command** in **Build** and add this command to run the scripts **newman run W33Collection.json -e W33Env.json**. This is the most basic way to execute Postman's API automation scripts on Jenkins, you can add more setup such as running on schedule, sending email.

Beside, we can use jenkinsfile to setup continuous integration on Jenkins easily. we just initiate new pipeline project on Jenkins and set up that project to get data from git project and make it able to find the jenkinsfile.

## Demo

The demo is stored in the [git project](https://github.com/W33HaaOrg/ApiTestPostMan.git).
