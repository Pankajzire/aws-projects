
# Building event-driven architectures on AWS

Many customers are choosing to build event-driven application architectures – those in which subscriber or target services automatically perform work in response to events triggered by publisher or source services. This pattern can enable development teams to operate more independently so they can release new features faster, while also making their applications more scalable.


* Time to Complete: 2-4 hours

* AWS Services: Amazon EventBridge, Amazon SNS, Amazon SQS, API Gateway and more.


## Cloudformation Tamplate

#### Use this tamplate to create necessary resources

1. Enter a stack name (or just keep the default name)
2. Check the boxes in the Capabilities section
3. Click Create stack



```http
  https://aws-event-driven-architecture-workshop-assets.s3.amazonaws.com/master-v2.yaml
```


## Event Generator setup

Configure the Event Generator

Once the stack creation has been completed navigate to the stack Outputs tab. Here you will find all the values necessary to configure the Event Generator to work with your account.

1. EventGeneratorConfigurationUrl - this url will pre-populate the Event Generator with all the settings you need.

2. CognitoPassword - your password for logging into the Event Generator

3. CognitoUsername - your user name (defaults to "user")

4. Right-click the url in the EventGeneratorConfigurationUrl output, opening a new browser window.

5. This will open the Event Generator website and pre-populate the modal dialog box with values for a Cognito User Pool and Cognito Identity Pool provisioned in your AWS account. Click Configure Cognito User Pool to view the sign-in page.

6. On the Sign In page add your the CognitoUsername and CognitoPassword from the CloudFormation Stack Output page.

7. Click the Sign In button.

8. After successful sign in, you verify that the Cognito user has been configured and that your account id has been pre-populated in the AWS Account ID field.







## First event bus and targets

* Create a custom EventBridge event bus, Orders, and an EventBridge rule, OrderDevRule, which matches all events sent to the Orders event bus and sends the events to a CloudWatch Logs log group, /aws/events/orders.

* The technique of logging all events to CloudWatch Logs is useful when implementing EventBridge rules.

## Step 1: Create a custom event busHeader anchor link

1. Open the AWS Management Console for EventBridge  in a new tab or window, so you can keep this step-by-step guide open.

2. On the EventBridge homepage, under Events, select Event buses from the left navigation.]

3. Click Create event bus.

4. Name the event bus Orders.

5. Leave Event archive and Schema discovery disabled, Resource-based policy blank.

6. Click Create.

## Step 2: Set up Amazon CloudWatch target (for development work)

1. From the left-hand menu, select Rules.

2. From the Event bus dropdown, select the Orders event bus

3. Click Create rule

4. Define rule details    
* Add 'OrdersDevRule' as the Name of the    rule

* Add Catchall rule for development purposes for Description

* select Rule with an event pattern for the Rule type

5. In the next step, Build event pattern

 * under Event source, choose Other
    
* Under Event pattern, further down the screen, enter the following pattern to catch all events from com.aws.orders:
    
       {"source": ["com.aws.orders"]}

6. Select your rule target:

 * From the Target dropdown, select CloudWatch log group

 * Name your log group /aws/events/orders

7. Skip through the configure tags section, review your rule configuration and click Create.

## Step 3: Test your dev rule

1. Select the EventBridge tab in the Event Generator .

2. Make sure that the Event Generator is populated with the following 
  
  * AWS Region should be to the region in which you are running the workshop

* Event Bus selected to Orders

* Source should be com.aws.orders

* In the Detail Type add Order Notification

* JSON payload for the Detail Template should be:

      {"category": "lab-supplies","value": 415,"location": "eu-west"}

3. Click Publish.

4. Open the AWS Management Console for CloudWatch  in a new tab or window, so you can keep this step-by-step guide open

5. Choose Log groups in the left navigation and select the /aws/events/orders log group.

6. Select the Log stream.

7. Toggle the log event to verify that you received the event.

## Step 4: Review event structure

In the following sections, you will use event data to implement EventBridge custom rules to route events. Due to the OrdersDevRule that you created in this section, all events to the Orders event bus will be sent to CloudWatch Logs, which you can use to view sample data in order to implement and troubleshoot rules matching logic.

    {
    "version": "0",
    "id": "c04cc8c1-283c-425e-8cf6-878bbc67a628",
    "detail-type": "Order Notification",
    "source": "com.aws.orders",
    "account": "111111111111",
    "time": "2020-02-20T23:10:29Z",
    "region": "us-west-2",
    "resources": [],
    "detail": {
    "category": "lab-supplies",
    "value": 415,
    "location": "eu-west"
    } 
    }

## Working with EventBridge rules

The steps to create an Orders event bus rule to match an event with a com.aws.orders source and to send the event to an Amazon API Gateway endpoint, invoke a AWS Step Function, and send events to an Amazon Simple Notification Service (Amazon SNS) topic.

## Rule matching basics

* Events in Amazon EventBridge are represented as JSON objects and have the following envelope signature:

      {
       "version": "0",
      "id": "6a7e8feb-b491-4cf7-a9f1-bf3703467718",
       "detail-type": "EC2 Instance State-change Notification",
       "source": "aws.ec2",
       "account": "111111111111",
       "time": "2017-12-22T18:43:48Z",
       "region": "us-west-1",
       "resources": [
       "arn:aws:ec2:us-west-1:123456789012:instance/ i-1234567890abcdef0"
       ],
       "detail": {
       "instance-id": " i-1234567890abcdef0",
       "state": "terminated"
       }
       }


* Rules use event patterns to select events and route them to targets. A pattern either matches an event or it doesn't. Event patterns are represented as JSON objects with a structure that is similar to that of events. For example, the following event pattern allows you to subscribe to only events from Amazon EC2.

      {
       "source": [ "aws.ec2" ]
      }

* The pattern simply quotes the fields you want to match and provides the values you are looking for.

* The sample event above, like most events, has a nested structure. Suppose you want to process all instance-termination events. Create an event pattern like the following.

      {
       "source": [ "aws.ec2" ],
       "detail-type": [ "EC2 Instance State-change Notification" ],
       "detail": {
       "state": [ "terminated" ]
      }
      }

 It is important to remember the following about event pattern matching:

 * For a pattern to match an event, the event must contain all the field names listed in the pattern. The field names must appear in the event with the same nesting structure.

 * Other fields of the event not mentioned in the pattern are ignored; effectively, there is a "*" : "*" wildcard for fields not mentioned.

 * The matching is exact (character-by-character), without case-folding or any other string normalization.

 * The values being matched follow JSON rules: Strings enclosed in quotes, numbers, and the unquoted keywords true, false, and null.

 * Number matching is at the string representation level. For example, 300, 300.0, and 3.0e2 are not considered equal.


## API Destination

## Step 1: Identify the API URL

1. Open the AWS Management Console for CloudFormation . You can find the API URL for this challenge in the Outputs of the CloudFormation Stack with a name containing ApiUrl.

## Step 2: Configure the EventBridge API Destination with basic auth securityHeader anchor link

1. Open the AWS Management Console for EventBridge  in a new tab or window, so you can keep this step-by-step guide open.

2. On the EventBridge homepage, select API destinations from the left navigation.

3. On the API destinations page, select Create API destination.

4. On the Create API destination page
   * Enter api-destination as the Name
* Enter the API URL identified in Step 1 as the API destination endpoint

* Select POST as the HTTP method

* Select Create a new connection for the Connection

* Enter basic-auth-connection as the Connection name

* Select Basic (Username/Password) as the Authorization type

* Enter myUsername as the Username

* Enter myPassword as the Password

5. Click Create

## Step 3: Configure an EventBridge rule to target the EventBridge API Destination

1. From the left-hand menu, select Rules.

2. From the Event bus dropdown, select the Orders event bus.

3. Click Create rule

4. On the Define rule detail page

  * Enter OrdersEventsRule as the Name of the rule
Enter Send com.aws.orders source events to API Destination for Description

5. Under Build event pattern

   * Choose Other for your Event source
Copy and paste the following into the Event pattern, and select Next to specify your target:

         {
         "source": [
         "com.aws.orders"
         ]
         }


6. Select your rule target:

   * Select EventBridge API destination as the target type.
Select api-destination from the list of existing API destinations

7. Click Next and finish walking through the rest of the walk-through to create the rule.

## Step 4: Send test Orders event

Below is sample data to test your rule, with a link to open the sample data in the Event Generator. If you don't remember how to use Event Generator to send an event, please refer to the previous section.

* Using the Event Generator, send the following Order Notification events from the source com.aws.orders:

       { 
       "category": "lab-supplies", 
       "value":  415, 
       "location": "us-east" 
       }


## Step 5: Verify API Destination

If the event sent to the Orders event bus matches the pattern in your rule, then the event will be sent to an API Gateway REST API endpoint.

1. Open the AWS Management Console for CloudWatch Log Groups  in a new tab or window, so you can keep this step-by-step guide open.

2. Select the Log group with an API-Gateway-Execution-Logs prefix.

3. Select the Log stream.

4. Toggle the log event to verify the basic authorization was successful and the API responds with a 200 status.

## Step Functions

## Step 1: Implement an EventBridge rule to target Step Functions

Use the EventBridge Console to:

1. Add a rule to the Orders event bus with the name EUOrdersRule

2. Define an event pattern to match events with a detail location in eu-west or eu-east:

       {
         "source": ["com.aws.orders"],
         "detail":{
           "location": ["us-west", "us-east"]
       }
       }



3. target the OrderProcessing Step Functions state machine

Here is a sample event to reference when writing the event pattern:

    {
    "version": "0",
    "id": "6e6b1f6d-48f8-5dff-c2d2-a6f22c2e0086",
    "detail-type": "Order Notification",
    "source": "com.aws.orders",
    "account": "111111111111",
    "time": "2020-02-23T15:35:41Z",
    "region": "us-east-1",
    "resources": [],
    "detail": {
    "category": "office-supplies",
    "value": 300,
    "location": "eu-west"
    }
    }


## Step 2: Send test EU Orders events

Using the Event Generator, send the following Order Notification events from the source com.aws.orders:
 
 1. 

    { "category": "office-supplies", "value": 300, "location": "eu-west" }
2. 

    { "category": "tech-supplies", "value": 3000, "location": "eu-east" }

## Step 3: Verify Step Functions workflow execution

If the event sent to the Orders event bus matches the pattern in your rule, then the event will be sent to the OrderProcessing Step Functions state machine for execution.

1. Open the AWS Management Console for Step Functions  in a new tab or window, so you can keep this step-by-step guide open.

2. On the Step Functions homepage, open the left hand navigation and select State machines.

3. Enter OrderProcessing in the Search for state machines box and verify the state machine execution has succeeded.

## Extra credit

1. Implement an event pattern using Prefix Matching  that matches events with a location the begins with eu-. Do not modify the Step Function target and verify backwards compatibility by reusing the sample data above. Modify the test data to use a location, eu-south, to verify that additional EU locations trigger execution of the Step Functions state machine.

       {
       "source": ["com.aws.orders"],
       "detail": {
       "location": ["prefix": "eu-"]
       }
       }


2. Implement an event pattern using Prefix Matching  and Numeric Matching  that matches events with a location the begins with eu- AND an Order value greater than 1000. Do not modify the Step Function target and verify backwards compatibility by reusing the sample data above.

       {
       "source": ["com.aws.orders"],
       "detail": {
       "location": ["prefix": "eu-"],
       "value" : [ { "numeric": [ ">", 1000 ] } ] 
       }
       }


## SNS

## Step 1: Implement an EventBridge rule to target SNS

Use the EventBridge Console to:

1. Add a rule to the Orders event bus with the name USLabSupplyRule

2. With an event pattern to match events with a detail location in us-west or us-east, and a detail category with lab-supplies.

       {
       "source": ["com.aws.orders"],
       "detail": {
       "location": ["us-west", "us-east"],
       "category": ["lab-supplies"]
       }
       }

3. Target the Orders SNS topic

Here is a sample event to reference:

    {
    "version": "0",
    "id": "6e6b1f6d-48f8-5dff-c2d2-a6f22c2e0086",
    "detail-type": "Order Notification",
    "source": "com.aws.orders",
    "account": "111111111111",
    "time": "2020-02-23T15:35:41Z",
    "region": "us-east-1",
    "resources": [],
    "detail": {
    "category": "lab-supplies",
    "value": 300,
    "location": "us-east"
    }
    }

## Step 2: Send test US Orders events

Using the Event Generator, send the following Order Notification events from the source com.aws.orders:

1. 
       { "category": "lab-supplies", "value": 415, "location": "us-east" }
2.   
       { "category": "office-supplies", "value": 1050, "location": "us-west", "signature": [ "John Doe" ] }


## Extra credit

* Implement an event pattern using Exists Matching  that matches events which do NOT require a signature (ie. signature does not exists). Do not modify the SNS target and verify backwards compatibility by reusing the sample data above.

       {
       "detail": {
       "signature": [ { "exists": false  } ]
       }
       }

## Workshop cleanup

####  Delete the workshop CloudFormation stack

