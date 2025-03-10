# json-py
import boto3
import uuid

comprehend = boto3.client('comprehend')
dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')

DYNAMODB_TABLE = "CustomerFeedback"
SNS_TOPIC_ARN = "arn:aws:sns:us-east-1:123456789012:NegativeFeedbackAlerts"

feedback_list = [
    "I love this product! It's amazing and easy to use.",
    "Terrible experience! The service was very bad and slow.",
    "The delivery was okay, but the packaging was damaged.",
    "Absolutely fantastic! Best purchase I have ever made."
]

def analyze_sentiment(feedback):
    response = comprehend.detect_sentiment(Text=feedback, LanguageCode="en")
    return response['Sentiment']

def store_feedback(feedback, sentiment):
    table = dynamodb.Table(DYNAMODB_TABLE)
    feedback_id = str(uuid.uuid4())
    table.put_item(
        Item={
            'FeedbackID': feedback_id,
            'FeedbackText': feedback,
            'Sentiment': sentiment
        }
    )
    return feedback_id
    
def send_alert(feedback):
    sns.publish(
        TopicArn=SNS_TOPIC_ARN,
        Message=f"Negative feedback detected: {feedback}",
        Subject="Urgent: Customer Complaint"
    )


for feedback in feedback_list:
    sentiment = analyze_sentiment(feedback)
    feedback_id = store_feedback(feedback, sentiment)
    
    print(f"Feedback ID: {feedback_id}")
    print(f"Text: {feedback}")
    print(f"Sentiment: {sentiment}")
    print("-" * 50)
    
    # Send alert if sentiment is negative
    if sentiment == "NEGATIVE":
        send_alert(feedback)
        print("🔴 SNS Alert Sent for Negative Feedback!\n")
