---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby
  - python
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Aerofit API! This API gives access to all Aerofit related endpoints.

On the right side you’ll see code examples. We currently have JSON code samples, showing the formatting of requests and responses.

This is updated as the API is being expanded.

Right now the API is only available internally at Aerofit so don’t share anything from within this documentation with anyone outside Aerofit.

Api is primarly coded in ruby on rails. The next development stages will be continuously translated into node.js

Authors of Api are Denis Trebula and Rasmus Reiler.

# Authentication

The Aerofit API uses authentication for almost all endpoints. The authentication system is in charge of keeping a user logged in, and therefore also for scoping API requests to the users resources.

The authentication system works by using headers. On authenticated endpoints the API requires five headers to be sent with the requests. When the request is processed by the API a timer of five seconds will be set, after which the headers will be invalidated (in development the headers won’t be invalidated)

The API will include new headers on all authenticated responses. The client should use the new headers as the old ones will be invalidated. If the request and response is sent over HTTTPS, this should protect against MITM attacks.

The headers are as follows:

access-token

client

expiry

token-type

uid

Almost all endpoints can therefore return error code 403 Unauthorized.

We’re using the Ruby gem Devise Token Auth as our authentication system. If more information is needed take a look at the linked Github repository.

## SIGN UP

> To Create a new user, you can use following code:

```json
{
  "user": {
    "email": "test@test.dk",
    "password": "Test12345678",
    "password_confirmation": "Test12345678",
    "name": "test",
    "admin": false
  }
}

```
>The API will respond with 200:

```json
{
    "status": "success",
    "data": {
        "id": 5,
        "email": "test@test.dk",
        "provider": "email",
        "name": "test",
        "activity_level": 0,
        "age": -1,
        "uid": "test@test.dk",
        "created_at": "2017-08-16T08:03:39.384Z",
        "updated_at": "2017-08-16T08:03:39.620Z",
        "address": null,
        "country": null,
        "last_test": null
    }
}
```

This endpoint is used to create a new User in the database. This is preferably the first endpoint a user should hit when using the application. The endpoint is currently handled fully by Devise, meaning the application will have to PATCH the user in a later request with more information.

Creating a user with this endpoint successfully also returns the users headers, meaning there’s no need to sign in afterwards.

Assigning user role of a admin will give him more privilages and access to different endpoints for more information.

### Query Parameters

Parameter | Required | Description
-------------- | -------------- | --------------
name | true | Users name
email | true | Users email
password | true | Users password. Minimum 6 characters long, must contains lower case and upper case letter and a number.
password_confirmation | true | A repetition of the users password
admin | false | To make user into super user/admin

###HTTP Request

`POST http://207.154.254.66/auth`

###Errors
If the API call is unsuccesful the following errors can occur

`422: Unprocessable entity`


## SIGN IN


> Your request should look like following:

```json
{
  "email": "test@test.dk",
  "password": "Test12345678"
}
```
>The api will respond with 200:

```json
{
  "data": {
      "id": 5,
      "email": "test@test.dk",
      "provider": "email",
      "name": "test",
      "activity_level": 0,
      "age": -1,
      "uid": "test@test.dk",
      "address": null,
      "country": null,
      "last_test": null
  }
}
```


If a user already exists in the database you must use the sign-in endpoint to retrieve the authentication headers.

Do not forget that the user must use a password which contains:
           Must contain 6 or more characters.
           Must contain a digit.
           Must contain a lower case character.
           Must contain an upper case character.

This is important in order to achieve at least basic security measures.

### Query Parameters

Parameter | Required | Description
-------------- | -------------- | --------------
email | true | Users email
password | true | Users password

###HTTP Request
`POST http://207.154.254.66/auth/sign_in`

###Errors
If the API call is unsuccesful the following errors can occur

`401: Unauthorized`

## SIGN OUT

This endpoint is used to logout a user. The user is found through the request headers.

###HTTP Request
`DELETE http://207.154.254.66/auth/sign_out`

###Errors
If the API call is unsuccesful the following errors can occur

`404: Not found`

# Users

The User object is the central object in the API. Every user has one. It links every other object together and is used for authorization of resources. It contains information that pertains to that one specific user.

User is linked with Test_sessions, Training_sessions, Performances, Devices.

## Get User

```json
{
    "id": 5,
    "email": "test@test.dk",
    "name": "test_doc",
    "address": null,
    "country": "",
    "age": -1,
    "activity_level": 0,
    "last_test": null,
    "test_sessions": {
        "id": 1,
        "created_at": "2017-07-14T12:24:12.694Z",
        "peak_exp": 2,
        "peak_insp": -2,
        "avg_exp": 1,
        "avg_insp": -12
    },
    "training_sessions": {
        "id": 1,
        "date": null
    },
    "performance":{"id":23,"avg_weekly":{"start_date":"2018-01-10T17:42:17.127Z","1":{"avg_evaluation":5.4,"num_entries":5}},
    "avg_monthly":{"start_date":"2018-01-10T17:42:17.127Z","0":{"avg_evaluation":5.4,"num_entries":5}},
    "avg_yearly":{"start_date":"2018-01-10T17:42:17.12"}
    },
    "device": null
}
```

This endpoint responds with a user. It also embeds latest TrainingSession, TestSession, Device and Performances object.
The TrainingSession only contains the ID and date so you can show the upcoming training to the use in the application.

###HTTP Request
`GET http://207.154.254.66/users/:id`

`GET http://207.154.254.66/users/:email`

if you want to retreive users by email, email must be in following format: Instead of test@test.dk   use  test-test-dk

This is due to friendly id library.

###Errors
If the API call is unsuccesful the following errors can occur

`404: Not found`

###Super user

Super user has in this case access to every user in database just by using mentioned http requests.

## Patch User

>Request should look like following:

```json
{
  "user": {
    "name": "patch_test"
  }
}
```

>Api will respond with

```json
{
    "id": 5,
    "email": "test@test.dk",
    "name": "patch_test",
    "address": null,
    "country": "",
    "age": -1,
    "activity_level": 0,
    "last_test": null,
    "test_sessions": {
        "id": 1,
        "created_at": "2017-07-14T12:24:12.694Z",
        "peak_exp": 2,
        "peak_insp": -2,
        "avg_exp": 1,
        "avg_insp": -12
    },
    "training_sessions": {
        "id": 1,
        "date": null
    },
    "performance":{"id":23,"avg_weekly":{"start_date":"2018-01-10T17:42:17.127Z","1":{"avg_evaluation":5.4,"num_entries":5}},
    "avg_monthly":{"start_date":"2018-01-10T17:42:17.127Z","0":{"avg_evaluation":5.4,"num_entries":5}},
    "avg_yearly":{"start_date":"2018-01-10T17:42:17.12"}
    },
    "device": null
}
```

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
email	| false |	Email
password |	false |	Password
name |	false	| Name
address	| false |	The users address
country |	false |	The country the user is living in
age |	false |	Age of the user. Defaults to -1
activity_level | false | How active the user is. From 0-3
admin |	false	| If you want to make user into admin or super user

###HTTP Request
`PATCH http://207.154.254.66/user/:id`

`PATCH http://207.154.254.66/user/:email`

Take in mind that email must be in format test-test-dk

###Errors
If the API call is unsuccesful the following errors can occur

`422: Unprocessable entity`


## Updating users password

```json
{
  "password": "Test123456",
  "password_confirmation": "Test123456"
}
```

>Api will respond with:

```json
{
    "success": true,
    "data": {
        "id": 5,
        "provider": "email",
        "email": "test@test.dk",
        "name": "patch_test",
        "activity_level": 0,
        "age": -1,
        "uid": "test@test.dk",
        "created_at": "2017-08-16T08:03:39.384Z",
        "updated_at": "2017-08-16T09:11:28.119Z",
        "address": null,
        "country": "",
        "last_test": null
    },
    "message": "Your password has been successfully updated."
}
```

This endpoint is used to change a users password. The user is found through the headers sent with the request.

Parameter | Required | Description
--------- | ------- | -----------
password | true |	New password
password_confirmation |	true |	Repeat of new password

###HTTP Request
`PUT http://207.154.254.66/auth/password`

###Errors
If the API call is unsuccesful the following errors can occur

`422: Unprocessable entity`


# Test Sessions


TestSession objects are model representatives of the tests the
 will have to take during the course of his training. It is from these that the application will base the users training.

User can have multiple Test sessions but only the latest one will be used for calculations if user decides to create new Training session.

## Get multiple test sessions

>Api will respond with:

```json
[
    {
        "id": 3,
        "peak_exp": 2,
        "peak_insp": -2,
        "created_at": "2017-07-14T12:29:56.422Z"
    },
    {
        "id": 2,
        "peak_exp": 2,
        "peak_insp": -2,
        "created_at": "2017-07-14T12:29:32.564Z"
    },
    {
        "id": 1,
        "peak_exp": 2,
        "peak_insp": -2,
        "created_at": "2017-07-14T12:24:12.694Z"
    }
]
```

 1 page is 20 TrainingSessions.

You can GET a specific TestSession afterwards to get more information about it specifically.

Parameter | Required | Description
--------- | ------- | -----------
?page=1 | false	| Which page to fetch. Defaults to page 1

###HTTP Request
`GET http://207.154.254.66/users/:id/test_sessions`

###Errors
If the API call is unsuccesful the following errors can occur

`404: Not found`

### Super user

Super users have endpoints slightly different because they can access not only their test sessions but every test session possible.

`GET http://207.154.254.66/users/:adminId/test_sessions/:idOfAnyUser`

:adminId can be either id of the admin user or he email.

If admin wants to access his own test sessions, he should do it with this same url but with his Id/email used at the end aswell.

## Get one test session

>The api will respond with :

```json
{
    "id": 3,
    "peak_exp": 2,
    "peak_insp": -2,
    "avg_exp": 1,
    "avg_insp": -1,
    "rec_graph_func": "",
    "created_at": "2017-07-14T12:29:56.422Z"
}
```

This endpoint gets one (1) specific TestSession. This is done after the above request to get more information for a certain TestSession.

###HTTP Request
`GET http://207.154.254.66/users/:user_id/test_sessions/:id`

###Errors
If the API call is unsuccesful the following errors can occur

`404: Not found`

### Super user

Super user can access any test session but the end point looks like this

`GET http://207.154.254.66/users/:adminId/test_sessions/:idOfAnyUser/specific_test_session/:id`

The last :id represents the id of specific test session which super user wish to retreive.

Also instead of :adminId and :idOfAnyUser you can use emails but Id is more preferable.

##Post test sessions

>Request should look like following:

```json
{
  "test": {
    "date": "1970-01-01T00:00:00Z",
    "peak_exp": "-2",
    "peak_insp": "2",
    "avg_exp": "-1",
    "avg_insp": "1"
  }
}
```

>The respond will look like following :

```json
{
  "id": 19,
  "user_id": 2,
  "date": "1970-01-01T00:00:00.000Z",
  "peak_exp": -2,
  "peak_insp": 2,
  "avg_exp": -1,
  "avg_insp": 1,
  "rec_graph_func": null,
  "created_at": "2017-05-17T08:48:12.171Z",
  "updated_at": "2017-05-17T08:48:12.171Z"
}
```

Parameter | Required | Description
--------- | ------- | -----------
user_id	| true |	ID of the user
date | false |	Date when the training was completed
peak_exp |	false |	Peak expiratory
peak_insp |	false |	Peak inspiratory
avg_exp	| false |	Average expiratory
avg_insp |	false |	Average inspiratory


###HTTP Request
`POST http://207.154.254.66/users/:user_id/test_sessions`

###Errors
If the API call is unsuccesful the following errors can occur

`422: Unprocessable entity`

#Training sessions

TrainingSession objects are created by user, and can be created only if the user completes TestSession. This should be verified in the front end.
Posting a new Training session will will generate a training with our super secret formula.

## Get Training sessions

> The api will respond with

```json
[
    {
        "id": 18,
        "length": null,
        "evaluation_score": 0,
        "performed": false,
        "graph_func": [
            0.4,
            -0.4
        ]
    },
    {
        "id": 17,
        "length": null,
        "evaluation_score": 0,
        "performed": false,
        "graph_func": [
            0.4,
            -0.4
        ]
    }
]
```

This endpoint gets 1 page of TrainingSessions. The first index in the response will always be the latest TrainingSession (usually one that the user hasn’t completed yet).

1 page is 20 TrainingSessions.

You can GET a specific TrainingSession afterwards to get more information about it specifically.


Parameter | Required | Description
--------- | ------- | -----------
?page=1 | false	| Which page to fetch. Defaults to page 1



###HTTP Request
`GET http://207.154.254.66/users/:id/training_sessions`

###Errors
If the API call is unsuccesful the following errors can occur

`404: Not found`


###Super user

Same as with test_sessions

`GET http://207.154.254.66/users/:adminId/training_sessions/:idOfAnyUser `

For more information you can look at the Super User/Admin cards in Trello

## Get one Training session

>The Api will respond with:

```json
{
    "id": 18,
    "exp_hole_setting": 8,
    "insp_hole_setting": 8,
    "graph_func": [
        0.4,
        -0.4
    ],
    "length": null,
    "evaluation_score": 0,
    "performed": false,
    "cal_lung_capacity": null
}
```


This endpoint gets one (1) specific TrainingSession. This is done after the above request to get more information for a certain TrainingSession.

###HTTP Request
`GET http://207.154.254.66/users/:training_id/training_sessions/:id`

###Errors
If the API call is unsuccesful the following errors can occur

`404: Not found`

###Super user

Same as with Test session

`GET http://207.154.254.66/users/:adminId/training_sessions/:idOfAnyUser/specific_training_session/:id`

For more information you can look at the Super User/Admin cards in Trello

##Post training session

>The post request can look like following:

```json
{
  "training": {
    "exp_hole_setting": 8,
    "insp_hole_setting": 8,
    "graph_func": [
        0.4,
        -0.4
    ],
    "length": 12389429,
    "evaluation_score": 0,
    "performed": false,
    "cal_lung_capacity": null
  }
}
```

Training session contains the latest/newest Test Session Id based on which we do some calculations.


###HTTP Request
`POST http://207.154.254.66/users/:user_id/training_sessions`


## Patch training session

> your request shoud look like following:

```json
{
  "training": {
    "length": 12389429
  }
}
```

> The api will respond with:

```json
{
    "id": 18,
    "exp_hole_setting": 8,
    "insp_hole_setting": 8,
    "graph_func": [
        0.4,
        -0.4
    ],
    "length": 12389429,
    "evaluation_score": 0,
    "performed": false,
    "cal_lung_capacity": null
}
```

PATCHing a TrainingSession. If the TrainingSession is completed, there must be a value performed set to True.

Parameter | Required | Description
--------- | ------- | -----------
user_id	| true |	ID of the user
performed	|false |	If the training has been performed. Not required but should be set
date |	false	| The date of completion
graph_data_x	| false |	X-axis data points. Serialized as array
graph_data_y |	false	| Y-axis data points. Serialized as array
cycles |	false |	Array holding information of when a cycle har been completed. One cycle is one top and one bottom
exp_hole_setting	| false |	Diameter of expiratory hole
insp_hole_setting	 | false	| Diameter of inspiratory hole
evaluation_score | false	 | How well the user performed the training
length	| false	| Time in miliseconds
difficulty_reduction |	false	 | When and how much the difficulty has been reduced. Needs more work!!
cal_lung_capacity |	false	| Calculated user lung capacity



###HTTP Request
`PATCH http://207.154.254.66/users/:user_id/training_sessions/:id`

###Errors
If the API call is unsuccesful the following errors can occur

`422: Unprocessable entity`

#Performances

We collect statistics about user from his training session if the training session is completed and based on the values we calculate
his weekly, monthly and yearly performance.

Performances are collected automatically in the Api and are stored in the form of hash values.

##Get Performances

>The api will respond with:

```json
{"id":23,"avg_weekly":{"start_date":"2018-01-10T17:42:17.127Z","1":{"avg_evaluation":5.4,"num_entries":5}},
"avg_monthly":{"start_date":"2018-01-10T17:42:17.127Z","0":{"avg_evaluation":5.4,"num_entries":5}},
"avg_yearly":{"start_date":"2018-01-10T17:42:17.127Z","0":{"avg_evaluation":5.4,"num_entries":5}}}
```

The api will respond with the id of performance, their starting date, average evaluation, number of entries and the number of which month/week/year it is in. If it is first Week or second or third etc.

All the following calculations must be done in the front end of the application.


###HTTP Request
`Get http://207.154.254.66/users/:user_id/performances`
