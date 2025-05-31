# AWS Wildrydes

## WildRydes
Created a website for a unicorn ride-sharing service called Wild Rydes. It is a serverless application that was fully created through aws services

## Functionality
This app allows users to create an account and log in, followed by a secure authentication process. Once authenticated, users can easily request a ride by clicking on a location on the map.

## List of AWS services used
Amazon Cognito: Implements authentication and authorization for web applications by creating a user portal. Users are required to provide their information and password, after which they receive a verification message along with an authorization token.

Amazon IAM: Creates a role for Lambda and DynamoDB.

Amazon Amplify: Used to build and makes changes to the web application.

API Gateway:Collect unicorn requestion which then calls the lambda function to send a unicorn.

AWS Lambda:Processes API requests triggered by user login, which are handled by Unicorn and routed through the API Gateway. The request is then processed and stored in DynamoDB.

Amazon DynamoDB: Creates table that collects and record the unicron request.

## Welcome page
![1719366594950](https://github.com/user-attachments/assets/b4226b3f-26b3-4f83-beee-eaf6dc545183)

## Login
![Screenshot 2025-05-31 175948](https://github.com/user-attachments/assets/aa94b8e8-812a-42fd-8a24-9b0637ab02e7)

## Map
![image](https://github.com/user-attachments/assets/f7f3d9b2-78b0-4bd2-96c6-c2d10f24af21)



## The Lambda Function Code
```node
import { randomBytes } from 'crypto';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const ddb = DynamoDBDocumentClient.from(client);

const fleet = [
    { Name: 'Angel', Color: 'White', Gender: 'Female' },
    { Name: 'Gil', Color: 'White', Gender: 'Male' },
    { Name: 'Rocinante', Color: 'Yellow', Gender: 'Female' },
];

export const handler = async (event, context) => {
    if (!event.requestContext.authorizer) {
        return errorResponse('Authorization not configured', context.awsRequestId);
    }

    const rideId = toUrlString(randomBytes(16));
    console.log('Received event (', rideId, '): ', event);

    const username = event.requestContext.authorizer.claims['cognito:username'];
    const requestBody = JSON.parse(event.body);
    const pickupLocation = requestBody.PickupLocation;

    const unicorn = findUnicorn(pickupLocation);

    try {
        await recordRide(rideId, username, unicorn);
        return {
            statusCode: 201,
            body: JSON.stringify({
                RideId: rideId,
                Unicorn: unicorn,
                Eta: '30 seconds',
                Rider: username,
            }),
            headers: {
                'Access-Control-Allow-Origin': '*',
            },
        };
    } catch (err) {
        console.error(err);
        return errorResponse(err.message, context.awsRequestId);
    }
};

function findUnicorn(pickupLocation) {
    console.log('Finding unicorn for ', pickupLocation.Latitude, ', ', pickupLocation.Longitude);
    return fleet[Math.floor(Math.random() * fleet.length)];
}

async function recordRide(rideId, username, unicorn) {
    const params = {
        TableName: 'Rides',
        Item: {
            RideId: rideId,
            User: username,
            Unicorn: unicorn,
            RequestTime: new Date().toISOString(),
        },
    };
    await ddb.send(new PutCommand(params));
}

function toUrlString(buffer) {
    return buffer.toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}

function errorResponse(errorMessage, awsRequestId) {
    return {
        statusCode: 500,
        body: JSON.stringify({
            Error: errorMessage,
            Reference: awsRequestId,
        }),
        headers: {
            'Access-Control-Allow-Origin': '*',
        },
    };
}
```

## The Lambda Function Test Function
Here is the code used to test the Lambda function:

```json
{
    "path": "/ride",
    "httpMethod": "POST",
    "headers": {
        "Accept": "*/*",
        "Authorization": "eyJraWQiOiJLTzRVMWZs",
        "content-type": "application/json; charset=UTF-8"
    },
    "queryStringParameters": null,
    "pathParameters": null,
    "requestContext": {
        "authorizer": {
            "claims": {
                "cognito:username": "the_username"
            }
        }
    },
    "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
}
```

