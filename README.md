import json
import boto3
import uuid

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('ToDoList')

def lambda_handler(event, context):
    http_method = event['httpMethod']
    path_parameters = event.get('pathParameters', {})
    body = json.loads(event['body']) if event.get('body') else {}

    if http_method == "POST":
        # Add Task
        task_id = str(uuid.uuid4())
        body['id'] = task_id
        table.put_item(Item=body)
        return {
            'statusCode': 200,
            'body': json.dumps({'message': 'Task added successfully', 'id': task_id})
        }

    elif http_method == "GET":
        # Get Tasks
        response = table.scan()
        return {
            'statusCode': 200,
            'body': json.dumps(response['Items'])
        }

    elif http_method == "PUT":
        # Update Task
        task_id = path_parameters.get('id')
        if not task_id:
            return {'statusCode': 400, 'body': json.dumps({'message': 'Task ID is required'})}
        
        table.update_item(
            Key={'id': task_id},
            UpdateExpression="set task_name=:n, task_status=:s",
            ExpressionAttributeValues={
                ':n': body.get('task_name'),
                ':s': body.get('task_status')
            },
            ReturnValues="UPDATED_NEW"
        )
        return {
            'statusCode': 200,
            'body': json.dumps({'message': 'Task updated successfully'})
        }

    elif http_method == "DELETE":
        # Delete Task
        task_id = path_parameters.get('id')
        if not task_id:
            return {'statusCode': 400, 'body': json.dumps({'message': 'Task ID is required'})}
        
        table.delete_item(Key={'id': task_id})
        return {
            'statusCode': 200,
            'body': json.dumps({'message': 'Task deleted successfully'})
        }

    else:
        return {
            'statusCode': 400,
            'body': json.dumps({'message': 'Unsupported HTTP method'})
        }
