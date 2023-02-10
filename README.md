# Triggering Lambda with API Gateway to Send Messages to SQS

For this project we will create an SQS Queue using Python with boto3. We will then create a Lambda Function to send a message to the SQS queue. Finally we will set an API Gateway, HTTP API trigger to invoke the function.


### Prerequisites
* AWS account
* Cloud9 IDE or IDE of your choosing, with AWS CLI, Python, and botos installed.

### Step 1

First we will create the SQS queue using Cloud9 IDE. Create a new python file, save it and input the following gist.


```
import boto3 
# Get the service resource
sqs = boto3.resource('sqs')

# Create the queue. This returns an SQS.Queue instance
queue = sqs.create_queue(QueueName='george-queue')

# You can now access identifiers and attributes
print(queue.url)
```



Choose your own Queue name. Then click Run. The output will give you a URL.


![image](https://user-images.githubusercontent.com/115881685/218207538-72ade253-162d-415d-92d4-1025dece8175.png)


You can head over to Amazon SQS to see that the Queue was created.



![image](https://user-images.githubusercontent.com/115881685/218207755-619b7ac7-9e58-4e31-8def-5c1837822430.png)



### Step 2

Next we will create the Lambda function. In the AWS console navigate to Lambda > Create function. Choose to Author from scratch. Input a name for your function and for Runtime choose Python 3.7 or higher. Under Permissions select to create a new role and then click Create function.


![image](https://user-images.githubusercontent.com/115881685/218207924-2ef80ef3-cad7-437d-8363-e83f7db07ae3.png)
![image](https://user-images.githubusercontent.com/115881685/218208298-3f0c0da9-9608-4416-865e-515f88de8837.png)



Now that the Lambda function is created we will need to edit the permissions. To do this click on Configuration > Permissions and select the Execution role.


![image](https://user-images.githubusercontent.com/115881685/218208505-61e0035a-ff01-4c2a-92c7-d07b21364ed7.png)


By clicking the role you will be brought to IAM. Here you can add permissions by attaching a policy or edit the basic policy that was created for your Lambda Function. I used the visual editor to add Allow Send Message. To do this click on the policy > Edit policy. Click on the Visual editor tab to add additional permissions or click the JSON tab and input the following permissions policy. Then click Review policy > Save changes.



```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "sqs:SendMessage",
                "logs:CreateLogGroup"
            ],
            "Resource": "arn:aws:sqs:us-east-1:361970161148:george-queue"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:us-east-1:361970161148:log-group:/aws/lambda/sqs_lambda:*"
        }
    ]
}
```


### Step 3


Navigate back to Lambda, here we will modify our function to send a message to SQS queue. The message will contain a random string of numbers. To do this copy and paste the Python code below into the Code source of the Lambda function. Change the QueueURL to the URL for your SQS queue.



```
import boto3
import json
import random
import string

def lambda_handler(event, context):

    num = string.digits
    random_num = ( ''.join(random.choice(num) for i in range(10)))
    message = "You are user: "+str(random_num)
    
    sqs = boto3.client('sqs')
    
    sqs.send_message(
        QueueUrl="https://sqs.us-east-1.amazonaws.com/361970161148/george-queue",
        MessageBody=random_num)

    return {
        'statusCode': 200,
        'body': json.dumps(message)
        }
```



Once you have input the code click on Deploy and then Test.


![image](https://user-images.githubusercontent.com/115881685/218209205-0770419c-dbf1-44ad-b37a-3d7ca774d1d7.png)



You will be brought to a Configure test event screen. Click on Create new event, name it, and choose the Template > apigateway-aws-proxy. Click Save.



![image](https://user-images.githubusercontent.com/115881685/218209421-d6ee797f-8ac9-4bf5-8627-ca3fe01451fa.png)
![image](https://user-images.githubusercontent.com/115881685/218209533-be1a7cf7-a9cd-4323-b914-8618213f282b.png)



Run the test by clicking on Test. You will see the results in a new Execution tab.


![image](https://user-images.githubusercontent.com/115881685/218209663-99969ae8-77b5-4832-b159-0fb5841e6f09.png)



### Step 4

Now to add the Trigger to our Lambda function. Click on Add trigger.

![image](https://user-images.githubusercontent.com/115881685/218209777-1384204e-bfa2-4e23-a34a-487716150192.png)



In the Trigger configuration settings select API Gateway as the trigger. Select Create an API and choose HTTP API. For Security select Open and click on Add.


![image](https://user-images.githubusercontent.com/115881685/218209921-0ddfc50f-34a0-41e7-a148-4a8518996120.png)


Scroll down to Triggers. Here you will find the API endpoint URL.


![image](https://user-images.githubusercontent.com/115881685/218210242-2d7fa1b2-cbba-4409-97ae-43fe3c087ae5.png)


Click on it and you should see the output of our code, “You are user:” and a random string of numbers. Click on it a few times to accumulate more messages in your queue.


![image](https://user-images.githubusercontent.com/115881685/218210518-31093958-3469-48e8-8b66-75f8ac603df4.png)



### Step 5

Finally we will check out the SQS queue to see that it is receiving messages from Lambda. Head over to SQS in the AWS console. Find your Queue and it shows we have a few messages!



![image](https://user-images.githubusercontent.com/115881685/218210826-f2ad5ab9-518f-44f6-b257-68bec10a6df4.png)




Go ahead and click on your queue and then click on Send and receive messages > Poll for messages.



![image](https://user-images.githubusercontent.com/115881685/218211055-00a4ac3e-4cf6-4b4c-bcdb-084546a9ddcc.png)




Click on one of the messages and you will see the random number output.



![image](https://user-images.githubusercontent.com/115881685/218211331-13df82ce-69c6-433d-bef3-aedbb1f2f160.png)



In conclusion we created an SQS queue, using Python, to receive messages from a Lambda function that was triggered by an API.

















