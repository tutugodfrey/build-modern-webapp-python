# Building Modern Python Application on AWS

## Get started 

Create a virtual environment

```bash
python3 -m venv ~/.bma
```

Source the environment

```bash
source ~/.bma/bin/activate
```

### Save the name of bucket and file to parameter store

```bash
aws ssm put-parameter --name dragon_data_bucket_name --value bma-dragon-bucket-7405 --type String
```

```bash
aws ssm put-parameter --name dragon_data_file_name --value bma-dragon-file --type String
```
### Get the name of the bucket and file (confirmation)

```bash
aws ssm get-parameters --name dragon_data_bucket_name --query Parameters[*].Value
```

```bash
aws ssm get-parameters --name dragon_data_file_name --query Parameters[*].Value
```



## Create public API Gateway

```bash
ws apigateway create-rest-api --name dragon --region us-east-1
```
Output

```bash
{
    "id": "xk36t54lld",
    "name": "dragon",
    "createdDate": 1626573015,
    "apiKeySource": "HEADER",
    "endpointConfiguration": {
        "types": [
            "EDGE"
        ]
    }
}
```

## Create an API Resource

```bash
aws apigateway create-resource --rest-api-id xk36t54lld --parent-id noeegz5qa9 --path-part dragons --region us-east-1
```

Output

```bash
{
    "id": "3x7u6l",
    "parentId": "noeegz5qa9",
    "pathPart": "dragons",
    "path": "/dragons"
}

```

## Create the get method
```bash
aws apigateway put-method --rest-api-id xk36t54lld --resource-id 3x7u6l --http-method GET --authorization-type NONE --region us-east-1
```

Output

```bash
{
    "httpMethod": "GET",
    "authorizationType": "NONE",
    "apiKeyRequired": false
}
```

## Create api integration of type MOCK

```bash
aws apigateway put-integration --rest-api-id xk36t54lld  --resource-id 3x7u6l --http-method GET --type MOCK --region us-east-1
```

Output

```bash
{
    "type": "MOCK",
    "passthroughBehavior": "WHEN_NO_MATCH",
    "timeoutInMillis": 29000,
    "cacheNamespace": "3x7u6l",
    "cacheKeyParameters": []
}
```

### Create a model (schema) for the api

```bash
aws apigateway create-model --rest-api-id xk36t54lld --content-type application/json --name dragonModel --schema '{ "type" :"object" }' --region us-east-1
```

```bash
{
    "id": "ftrwes",
    "name": "dragonModel",
    "schema": "{ \"type\" :\"object\" }",
    "contentType": "application/json"
}
```

### model for 404 status code
```bash
aws apigateway create-model --rest-api-id xk36t54lld --content-type application/json --name dragonModel404 --schema '{ "type" :"object" }' --region us-east-1
```

```bash
{
    "id": "gv5rad",
    "name": "dragonModel404",
    "schema": "{ \"type\" :\"object\" }",
    "contentType": "application/json"
}
```

### Create a method response
```bash
aws apigateway put-method-response --rest-api-id xk36t54lld --resource-id 3x7u6l --http-method GET --status-code 200 --response-models '{"application/json": "dragonModel"}' --region us-east-1
```

```bash
{
    "statusCode": "200",
    "responseModels": {
        "application/json": "dragonModel"
    }
}
```

### create a method for 404 status code

```bash
aws apigateway put-method-response --rest-api-id xk36t54lld --resource-id 3x7u6l --http-method GET --status-code 404 --response-models '{"application/json": "dragonModel404"}' --region us-east-1
```

```bash
{
    "statusCode": "404",
    "responseModels": {
        "application/json": "dragonModel404"
    }
}
```

### Delete a method response

```bash
aws apigateway delete-method-response --rest-api-id $API_ID --resource-id $R_ID --http-method GET --state-code 200 --region us-east-1
```

### Get the api method

```bash
aws apigateway get-method --rest-api-id xk36t54lld --resource-id 3x7u6l --http-method GET --region us-east-1
```

```bash
{
    "httpMethod": "GET",
    "authorizationType": "NONE",
    "apiKeyRequired": false,
    "methodResponses": {
        "200": {
            "statusCode": "200",
            "responseModels": {
                "application/json": "dragonModel"
            }
        }
    }
}
```

### Create 404 mock integration 

```bash
aws apigateway put-integration --rest-api-id $API_ID --resource-id $R_ID --http-method GET --type MOCK --request-templates '{ "application/json": "{\"statusCode\": 404}"}' --region us-east-1
```

```bash
{
    "type": "MOCK",
    "requestTemplates": {
        "application/json": "{\"statusCode\": 404}"
    },
    "passthroughBehavior": "WHEN_NO_MATCH",
    "timeoutInMillis": 29000,
    "cacheNamespace": "3x7u6l",
    "cacheKeyParameters": []
}
```

```bash
aws apigateway put-integration-response --rest-api-id $API_ID  --resource-id $R_ID --http-method GET --status-code 404 --response-templates '{"application/json": "{\"json\": \"template\"}"}' --region us-east-1
```

```bash
{
    "statusCode": "404",
    "responseTemplates": {
        "application/json": "{\"json\": \"template\"}"
    }
}
```

```bash
aws apigateway put-integration-response --rest-api-id $API_ID  --resource-id $R_ID --http-method GET --status-code 200 --selection-pattern 200 --response-templates '{"application/json": "{\"json2\": \"template202\"}"}' --region us-east-1
```

```bash
{
    "statusCode": "200",
    "selectionPattern": "200",
    "responseTemplates": {
        "application/json": "{\"json2\": \"template202\"}"
    }
}
```


**Workflow:**
- Create api gateway
- create a resource
- create a method
- create model(s) (that can be used for different method response)
- create a method response using a particular model that fit

## Example model mapping
[
  #if( $input.params('family') == 'red')
     {
      "description_str": "Frealu has an ice breath that can freeze her enemies into a paralyzed state. She is from the souther water tribe.",
      "dragon_name_str": "Frealu",
      "family_str": "blue",
      "location_city_str": "mesa",
      "location_country_str": "usa",
      "location_neighborhood_str": "e adobe st",
      "location_state_str": "arizona"
   },  {
      "description_str": "Frealu has an ice breath that can freeze her enemies into a paralyzed state. She is from the souther water tribe.",
      "dragon_name_str": "Frealu",
      "family_str": "blue",
      "location_city_str": "mesa",
      "location_country_str": "usa",
      "location_neighborhood_str": "e adobe st",
      "location_state_str": "arizona"
   }
  #elseif( $input.params('family') == 'blue')
    {
      "description_str": "Frealu has an ice breath that can freeze her enemies into a paralyzed state. She is from the souther water tribe.",
      "dragon_name_str": "Frealu",
      "family_str": "blue",
      "location_city_str": "mesa",
      "location_country_str": "usa",
      "location_neighborhood_str": "e adobe st",
      "location_state_str": "arizona"
    }, {
      "description_str": "Frealu has an ice breath that can freeze her enemies into a paralyzed state. She is from the souther water tribe.",
      "dragon_name_str": "Frealu",
      "family_str": "blue",
      "location_city_str": "mesa",
      "location_country_str": "usa",
      "location_neighborhood_str": "e adobe st",
      "location_state_str": "arizona"
    }
  #elseif( #input.params('dragonName') == 'Atlas')
    {
      "description_str": "Frealu has an ice breath that can freeze her enemies into a paralyzed state. She is from the souther water tribe.",
      "dragon_name_str": "Frealu",
      "family_str": "blue",
      "location_city_str": "mesa",
      "location_country_str": "usa",
      "location_neighborhood_str": "e adobe st",
      "location_state_str": "arizona"
    }, {
      "description_str": "Frealu has an ice breath that can freeze her enemies into a paralyzed state. She is from the souther water tribe.",
      "dragon_name_str": "Frealu",
      "family_str": "blue",
      "location_city_str": "mesa",
      "location_country_str": "usa",
      "location_neighborhood_str": "e adobe st",
      "location_state_str": "arizona"
    }
  #else
    {
      "description_str": "Frealu has an ice breath that can freeze her enemies into a paralyzed state. She is from the souther water tribe.",
      "dragon_name_str": "Frealu",
      "family_str": "blue",
      "location_city_str": "mesa",
      "location_country_str": "usa",
      "location_neighborhood_str": "e adobe st",
      "location_state_str": "arizona"
    }, {
      "description_str": "Frealu has an ice breath that can freeze her enemies into a paralyzed state. She is from the souther water tribe.",
      "dragon_name_str": "Frealu",
      "family_str": "blue",
      "location_city_str": "mesa",
      "location_country_str": "usa",
      "location_neighborhood_str": "e adobe st",
      "location_state_str": "arizona"
    }, {
      "description_str": "Frealu has an ice breath that can freeze her enemies into a paralyzed state. She is from the souther water tribe.",
      "dragon_name_str": "Frealu",
      "family_str": "blue",
      "location_city_str": "mesa",
      "location_country_str": "usa",
      "location_neighborhood_str": "e adobe st",
      "location_state_str": "arizona"
    }, {
      "description_str": "Frealu has an ice breath that can freeze her enemies into a paralyzed state. She is from the souther water tribe.",
      "dragon_name_str": "Frealu",
      "family_str": "blue",
      "location_city_str": "mesa",
      "location_country_str": "usa",
      "location_neighborhood_str": "e adobe st",
      "location_state_str": "arizona"
    }
  #end
]


### Make api request from the cli

export METHOD=GET # change to POST for post request

```bash
curl -H "Content-Type: aation/json" -X $METHOD -d @APIGatewayDemo/dragon_payload.json https://0sap2mrw58.execute-api.us-east-1.amazonaws.com/test/dragons
```

```bash
curl -H "Content-Type: aation/json" -X $METHOD -d @APIGatewayDemo/dragon_payload.json https://0sap2mrw58.execute-api.us-east-1.amazonaws.com/test/dragons?dragonName=Atlas
```

```bash
curl -H "Content-Type: application/json" -X $METHOD -d @APIGatewayDemo/dragon_payload.json https://0sap2mrw58.execute-api.us-east-1.amazonaws.com/test/dragons?family=blue
```

aws s3 website s3://my-bucket/ --index-document --error-document error.html

## Lambda functions versions and Alias

Publishing a function version

```bash
aws lambda publish-version --function-name list-dragons
```

Create an Alias for a function version

```bash
aws lambda create-alias --function-name list-dragons --name dragon-test --function-version 10 --description list-dragon-alias
```

Update the version for the alias

```bash
aws lambda update-alias --function-name list-dragons --name dragon-test --function-version 11
```


```bash
aws lambda publish-version --function-name test-execution-context --region us-west-2

aws lambda create-alias --function-name test-execution-context --name test-execution-context-test --function-version 1 --description test-execution-context-alias --region us-west-2
aws lambda invoke --function-name test-execution-context outfile --region us-west-2

aws lambda publish-version --function-name test-execution-context --region us-west-2 # create a new version of the function
aws lambda update-alias --function-name test-execution-context --name test-execution-context-test --function-version 2 --region us-west-2 # update the alias with the new version number
```

## Create List Dragon function

```bash
cd LambdaCode/Python/list-dragons/

pip3 install --target ./list-dragon-package boto3

cd list-dragon-package/

zip -r9 ${OLDPWD}/pythonlistDragonFunction.zip .

cd ..

zip -g pythonlistDragonFunction.zip listDragons.py
```

```bash
export BUCKET_NAME=$(aws ssm get-parameter --name dragon_data_bucket_name --query Parameter.Value --output text)

export DRAGON_DATA_FILE=$(aws ssm get-parameter --name dragon_data_file_name --query Parameter.Value --output text)
```

### Create IAM role for 

`Permissions/lambda-trust-policy.json`

```bash
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

```bash
aws iam create-role --role-name dragon-data-lambda-role --assume-role-policy-document file://Permissions/lambda-trust-policy.json
```

Output

```bash
{
    "Role": {
        "Path": "/",
        "RoleName": "dragon-data-lambda-role",
        "RoleId": "AROAXIBDD2N66DZZTLH7Q",
        "Arn": "arn:aws:iam::498289857405:role/dragon-data-lambda-role",
        "CreateDate": "2021-07-19T15:47:16Z",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "lambda.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        }
    }
}

```

### Create listDragon function

```bash
cd LambdaCode/Python/list-dragons

aws lambda create-function --function-name list-dragons --runtime python3.8 --role arn:aws:iam::498289857405:role/dragon-data-lambda-role --handler listDragons.listDragons --publish --zip-file fileb://pythonlistDragonFunction.zip

aws iam create-policy --policy-name dragon-data-lambda-policy --policy-document file://Permissions/lambda-role-policy.json --description "grant lambda permission to aws services"

aws iam attach-role-policy --role dragon-data-lambda-role --policy-arn "arn:aws:iam::498289857405:policy/dragon-data-lambda-policy"

aws iam attach-role-policy --role dragon-data-lambda-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole # attach lambda basic execution policy


export BUCKET_NAME=$(aws ssm get-parameter --name dragon_data_bucket_name --query Parameter.Value --output text)

export FILE_NAME=$(aws ssm get-parameter --name dragon_data_file_name  --query Parameter.Value --output text)

cp DragonData/dragon_stats_one.json ${FILE_NAME}

aws s3 cp ${FILE_NAME} s3://${BUCKET_NAME}

aws lambda invoke --function-name list-dragons output.txt


aws lambda delete-function --function-name list-dragons 
```

### Create validateDragon function

```bash
cd LambdaCode/Python/validate-dragon/

pip3 install --target ./validate-dragon-package boto3

cd validate-dragon-package/

zip -r9 ${OLDPWD}/pythonValidateDragonFunction.zip .

cd ..

zip -g pythonValidateDragonFunction.zip validateDragon.py

aws lambda create-function --function-name validateDragon --runtime python3.8 --role arn:aws:iam::498289857405:role/dragon-data-lambda-role --handler validateDragon.validate --publish --zip-file fileb://pythonValidateDragonFunction.zip

aws lambda invoke --function-name validateDragon output.txt --payload file://newDragonPayload.json

aws lambda invoke --function-name validateDragon output.txt --payload file://duplicateDragonPayload.json
```

### Create addDragon function

```bash
cd LambdaCode/Python/add-dragon/

pip3 install --target ./add-dragon-package boto3

cd add-dragon-package/

zip -r9 ${OLDPWD}/pythonAddDragonFunction.zip .

cd ..

zip -g pythonAddDragonFunction.zip addDragon.py

aws lambda create-function --function-name addDragon --runtime python3.8 --role arn:aws:iam::498289857405:role/dragon-data-lambda-role --handler addDragon.addDragonToFile --publish --zip-file fileb://pythonAddDragonFunction.zip

aws lambda invoke --function-name addDragon output1.txt --payload file://newDragonPayload.json
```


```bash
{
  "dragonName": "GreEE",
  "description": "New dragon",
  "family": "green",
  "city": "Houston",
  "country": "USA",
  "state": "TX",
  "neighborhood": "Downtown",
  "reportingPhoneNumber": "15555555555",
  "confirmationRequired": false
}

#set($data = $input.path('$'))

#set($input = " { ""dragon_name_str"" : ""$data.dragonName"", ""description_str"" : ""$data.description"", ""family_str"" : ""$data.family"", ""location_country_str"" : ""$data.country"", ""location_state_str"" : ""$data.state"", ""location_neighborhood_str"" : ""$data.neighborhood"", ""reportingPhoneNumber"" : ""$data.reportingPhoneNumber"", ""confirmationRequired"" : ""$data.confirmationRequired"", }")

{
    "input": "$util.escapeJavaScript($input).replaceAll("\\'", "'")",
    "stateMachineArn": "arn:aws:states:us-east-1:498289857405:stateMachine:Dragon-data-state-machine"
}
```

This is for dragon API is us-east-2 (Ohio)

```bash
curl -H "Authorization: $AUTH_HEADER" -H "Content-Type: application/json" -X POST --data-binary "@/Users/godfreytutu/Desktop/projects/bma/LambdaCode/Python/add-dragon/newDragonPayload.json" https://jtcecqkrs8.execute-api.us-east-2.amazonaws.com/test/dragons
```