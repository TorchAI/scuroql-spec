# scuroql-spec

ScuroQL is a query language for APIs created by TorchAI. 

The ScuroQL will be used accompanied with Scuro Server. The Scuro Server will return the result as ScuroQL query requested in a json format. For more information about Scuro, please refer [here](https://github.com/TorchAI/scuro).

Go to wiki tab of this repo to see the documentation of ScuroQL.


## Overview

This is a Working Draft of the Specification for ScuroQL, a query language for APIs created by TorchAI.

In order to be broadly adopted, ScuroQL will have to target a wide
variety of backends, frameworks, and languages, which will necessitate a
collaborative effort across projects and organizations. This specification serves as a point of coordination for this effort.


## Getting started

ScuroQL is intuitive and easy to learn. I will try my best to explain the syntax to you.

We are going to understand and write a json request following ScuroQL syntax.

Let's get started by an example!

Say, you are developing a question-answering website/app. For a specfic page, you want to get all the users id and their questions id in a json format.

You can format the following json and sent it to Scuro server.

```
test_array_json = {
    "dummy[]": {
        "SELECT":  ["id", 'created_at', {"ARRAY_JSON": {
                                                "ALIAS": 'user_answers',
                                                "SELECT": ['question_id', 'created_at'],
                                                "FROM": "answer",
                                                "WHERE": {
                                                    "=":
                                                        {'answer.user_id': 'user_account.id'}
                                                }
                                    }}],
        "FROM": ["user_account"]
        }
    }
```
which is equivalent to SQL query:
```SQL
SELECT array_agg(row_to_json(tmp)) AS dummy
FROM   (SELECT id, 
               created_at, 
               (SELECT array_to_json(array_agg(row_to_json(tmp))) 
                FROM   (SELECT question_id, 
                               created_at answer 
                        WHERE  ( answer.user_id = user_account.id )) tmp) AS 
                      user_answers 
        FROM   user_account) tmp
```

Let's break it down.


The outermost json is:

```
"dummy[]": {}

```
Here, `dummy` will be the name of the returned result of its json body.
`[]` tells that the the value of `dummy[]` contains all the information of a single, independent, full SQL query.


```
"Dummy[]": {
     "SELECT":  ["id", 'created_at'],
     "FROM": ["user_account"]
 }       
 ```
`"SELECT": []` specifies the columns will be seleced.
`"FROM": []` tells the table to select from.

Here it will select `"id", 'created_at'` columns from `user_account` table.


There is another element in `SELECT`:

```
{"ARRAY_JSON": {
                "ALIAS": 'user_answers',
                "SELECT": ['question_id', 'created_at'],
                "FROM": "answer",
                "WHERE": {
                    "=":
                        {'answer.user_id': 'user_account.id'}
 }
 ```
 
To understand it, I will explain a bit on how query is structured in ScuroQL.

In a ScuroQL query, the key will usually be regarded as a operator. And any operation will be wrapped up as a json. 

For example: 
```
 {
    "=":
        {'answer.user_id': 'user_account.id'}
}
 ```
This part represents `answer.user_id = user_account.id` in the final parsed SQL query. You can view it as: the `=` operator takes two inputs from its json body, and output the result `answer.user_id = user_account.id`

 
 Now, let's get back to the orignal query.
```
{"ARRAY_JSON": {
                "ALIAS": 'user_answers',
                "SELECT": ['question_id', 'created_at'],
                "FROM": "answer",
                "WHERE": {
                    "=":
                        {'answer.user_id': 'user_account.id'}
 }
 ```
 
 From above, we know `ARRAY_JSON` is an opearator, and it takes the elements in its json body as input parameters.
 
 `ARRAY_JSON` tells it will apply `array_to_json(array_agg(row_to_json()))` operation on its value.
 
 The `"ALIAS": 'user_answers'` parameters tells that the aggregated result will be viewed as `user_answers`
 
 Finally, the Scuro server will wrap the parsed query with `array_agg(row_to_json())` to return json as result.
 
 So the above json represents a subquery:
 
 ```
 SELECT array_to_json(array_agg(row_to_json(tmp))) 
 FROM   ( 
           SELECT question_id, 
                  created_at answer 
           WHERE  ( 
                     answer.user_id = user_account.id)) tmp) as user_answers
 ```

The ScuroQL query at the very beginning corresponds the following SQL:

```
SELECT array_agg(row_to_json(tmp)) AS dummy
FROM   (SELECT id, 
               created_at, 
               (SELECT array_to_json(array_agg(row_to_json(tmp))) 
                FROM   (SELECT question_id, 
                               created_at answer 
                        WHERE  ( answer.user_id = user_account.id )) tmp) AS 
                      user_answers 
        FROM   user_account) tmp
```


The returned body will look like this:

```
{
  "dummy": [
    "{\"id\":1,\"created_at\":\"6:47:48\",\"user_answers\":[{\"question_id\":1,\"created_at\":\"2019-07-22\"}]}",
    "{\"id\":2,\"created_at\":\"5:10:53\",\"user_answers\":[{\"question_id\":2,\"created_at\":\"2019-07-06\"}]}",
    "{\"id\":3,\"created_at\":\"20:40:36\",\"user_answers\":[{\"question_id\":3,\"created_at\":\"2019-09-13\"}]}",
    "{\"id\":4,\"created_at\":\"0:29:56\",\"user_answers\":[{\"question_id\":4,\"created_at\":\"2019-10-14\"}]}",
    "{\"id\":5,\"created_at\":\"22:37:02\",\"user_answers\":[{\"question_id\":5,\"created_at\":\"2019-07-26\"}]}",
    "{\"id\":6,\"created_at\":\"16:05:24\",\"user_answers\":[{\"question_id\":6,\"created_at\":\"2019-11-23\"}]}",
    "{\"id\":7,\"created_at\":\"16:10:51\",\"user_answers\":[{\"question_id\":7,\"created_at\":\"2019-07-07\"}]}",
    "{\"id\":8,\"created_at\":\"2:23:41\",\"user_answers\":[{\"question_id\":8,\"created_at\":\"2020-02-06\"}]}",
    "{\"id\":9,\"created_at\":\"19:08:15\",\"user_answers\":[{\"question_id\":9,\"created_at\":\"2019-05-19\"}]}",
    "{\"id\":10,\"created_at\":\"17:35:55\",\"user_answers\":[{\"question_id\":10,\"created_at\":\"2020-03-16\"}]}",
    "{\"id\":11,\"created_at\":\"20:51:09\",\"user_answers\":[{\"question_id\":11,\"created_at\":\"2020-04-30\"}]}",
    "{\"id\":12,\"created_at\":\"10:16:13\",\"user_answers\":[{\"question_id\":12,\"created_at\":\"2019-06-30\"}]}",
    "{\"id\":13,\"created_at\":\"21:40:50\",\"user_answers\":[{\"question_id\":13,\"created_at\":\"2019-12-10\"}]}",
    "{\"id\":14,\"created_at\":\"2:10:31\",\"user_answers\":[{\"question_id\":14,\"created_at\":\"2019-07-06\"}]}",
    "{\"id\":15,\"created_at\":\"2:36:59\",\"user_answers\":[{\"question_id\":15,\"created_at\":\"2019-11-29\"}]}",
    "{\"id\":16,\"created_at\":\"23:38:22\",\"user_answers\":[{\"question_id\":16,\"created_at\":\"2019-12-22\"}]}",
    "{\"id\":17,\"created_at\":\"10:59:30\",\"user_answers\":[{\"question_id\":17,\"created_at\":\"2020-04-16\"}]}",
    "{\"id\":18,\"created_at\":\"13:38:55\",\"user_answers\":[{\"question_id\":18,\"created_at\":\"2019-12-01\"}]}",
    "{\"id\":19,\"created_at\":\"8:43:30\",\"user_answers\":[{\"question_id\":19,\"created_at\":\"2019-07-16\"}]}",
    "{\"id\":20,\"created_at\":\"21:30:13\",\"user_answers\":[{\"question_id\":20,\"created_at\":\"2019-11-06\"}]}"
  ]
}
```


## Motivation
1. Save bandwidth

    a. Scuro can query certain fields of data, no over/under fetching

2. Clean standards

    a. common fields are pre-defined: 
        i. number of pages: count
        ii. index of page: page
        iii. total number: total

3. Type checking

    a. the type of fields will be validated

4. HTTP status codes
    
5. Decoupling with API

    Return the json body as you requested

6. Reduce the API development workflow


## Comparison with GraphQL


