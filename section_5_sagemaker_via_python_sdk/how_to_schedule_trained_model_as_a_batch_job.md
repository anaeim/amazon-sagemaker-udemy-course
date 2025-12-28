**Q.** you have a trained ml model in amazon sagemaker, and you want to schedule to run it as a batch job at a specific time. what is the best approach?<br>
**A.** since you want to schedule your trained model to run at a specific time on data, the best approach in SageMaker is usually a Batch Transform Job by Lambda function, triggered on a schedule using Amazon EventBridge (CloudWatch Events). This avoids keeping a real-time endpoint running 24/7, which saves cost.

# Deploying and Scheduling a Trained SageMaker Model

This guide shows how to deploy a trained Amazon SageMaker model and schedule it to run automatically on your data.

---

## 1. Use Batch Transform Job

A Batch Transform Job takes input data from S3, runs predictions, and writes output to S3.

```python
from sagemaker.sklearn import SKLearn

# Assume you have a trained estimator
estimator = SKLearn.attach("your-training-job-name")

# Create a transformer for batch predictions
transformer = estimator.transformer(
    instance_count=1,
    instance_type='ml.m5.large',
    output_path='s3://my-bucket/predictions/',
    strategy='SingleRecord'  # Optional: how to handle multiple files
)
```

---

## 2. Create a Lambda Function to Run the Batch Job

def lambda_handler(event, context):
Lambda function will call the batch transform job (`create_transform_job`) to 
- load trained ML model from s3 bucket
- run the model on the input data
- store the predictions.

```python
import boto3

sagemaker_client = boto3.client('sagemaker')

def lambda_handler(event, context):
    response = sagemaker_client.create_transform_job(
        TransformJobName='my-batch-job-' + context.aws_request_id,
        ModelName='your-trained-model-name',  # Can get from estimator.model_data
        MaxConcurrentTransforms=1,
        MaxPayloadInMB=6,
        BatchStrategy='SingleRecord',
        TransformInput={
            'DataSource': {
                'S3DataSource': {
                    'S3DataType': 'S3Prefix',
                    'S3Uri': 's3://my-bucket/input-data/'
                }
            },
            'ContentType': 'text/csv',
            'SplitType': 'Line'
        },
        TransformOutput={'S3OutputPath': 's3://my-bucket/predictions/'},
        TransformResources={'InstanceType': 'ml.m5.large', 'InstanceCount': 1}
    )
    print(response)
    return response
```
**NOTE**: here the `ModelName` is not S3 model artifact path (like s3://bucket/model.tar.gz).<br>
`ModelName` is name of a SageMaker Model resource, and you can manually specify it after you trained and created model on S3:
```python
model = estimator.create_model(
    name='my-custom-model-name'
)
```
---

## 3. Schedule the Lambda Using EventBridge

1. Go to **Amazon EventBridge → Rules → Create rule**.
2. Choose **Schedule**.
3. Specify your cron or rate, e.g., `cron(0 8 * * ? *)` → every day at 8 AM UTC.
4. Set the target as the Lambda function created above.

✅ Your model will now automatically run on your input data at the scheduled time, with outputs saved in S3.

---

## Optional: Real-Time Endpoint

If you prefer real-time predictions instead of batch:

- Deploy the model as a SageMaker endpoint.
- Use Lambda + EventBridge to call the endpoint at a schedule.
- Note: This incurs cost for a running endpoint, even when idle.

---

This setup allows for automated, scheduled inference of your SageMaker model using AWS services efficiently and cost-effectively.




# Q. explain AWS Lambda and EventBridge 

## 1. AWS Lambda

**AWS Lambda** is a serverless compute service that allows you to run code without provisioning or managing servers.

**Key points:**

- Write a function that starts your SageMaker Batch Transform job.
- Lambda executes automatically when triggered.
- Pay only for execution time.

### Example Lambda function:

```python
import boto3

sagemaker_client = boto3.client('sagemaker')

def lambda_handler(event, context):
    response = sagemaker_client.create_transform_job(
        TransformJobName='my-batch-job-' + context.aws_request_id,
        ModelName='your-trained-model-name',
        MaxConcurrentTransforms=1,
        MaxPayloadInMB=6,
        BatchStrategy='SingleRecord',
        TransformInput={
            'DataSource': {
                'S3DataSource': {
                    'S3DataType': 'S3Prefix',
                    'S3Uri': 's3://my-bucket/input-data/'
                }
            },
            'ContentType': 'text/csv',
            'SplitType': 'Line'
        },
        TransformOutput={
            'S3OutputPath': 's3://my-bucket/predictions/'
        },
        TransformResources={
            'InstanceType': 'ml.m5.large',
            'InstanceCount': 1
        }
    )
    print(response)
    return response
```

- `lambda_handler` is the entry point.
- `boto3.client('sagemaker')` lets Lambda interact with SageMaker.
- `context.aws_request_id` ensures unique job names.

---

## 2. Amazon EventBridge

**EventBridge** is a serverless event service that can trigger actions on a schedule or in response to events.

**Key points:**

- Create a rule triggered by a **cron schedule** or **rate**.
- The rule’s **target** can be Lambda, Step Functions, SQS, or other AWS services.
- Automates your ML batch job without manual intervention.

### Example cron schedule:
- `cron(0 8 * * ? *)` → triggers every day at **8:00 AM UTC**.
- The rule invokes the Lambda function automatically.

---

## 3. How Lambda and EventBridge work together

1. EventBridge triggers Lambda at the scheduled time.
2. Lambda executes and calls `create_transform_job()` in SageMaker.
3. SageMaker runs the batch job on input data from S3.
4. Output is saved to the configured S3 bucket.
5. Lambda finishes execution; EventBridge waits for the next schedule.

✅ This setup automates model predictions without needing a continuously running server or manual intervention.

---

**Optional:** Create a diagram to visualize the flow:

```
EventBridge (Scheduled Trigger)
        |
        v
   Lambda Function
        |
        v
SageMaker Batch Transform Job
        |
        v
       S3 Output
```

This shows the sequence of scheduled ML model execution and output storage.
