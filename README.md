# WFA Scheduler

POC WFA Scheduler based on AWS serverless stack.

## Requirements for enterprise based scheduling

- ***High precision (delivery time)***
	-  Events delivered as close as possible to scheduled delivery time.
- ***High precision (granularity)***
	-  High granularity of possible delivery time (i.e. 1 minute precision instead of 15 minute precision).
- ***Scale (# of pending task)***
	- Target solution must support large number of scheduled tasks (millions) waiting to be executed.
- ***Scale (# of task to be executed at the same time)***
	- Target solution must be able to execute many tasks (millions) in parallel at the same time. This implies quick retrieval of events to be executed from underlying storage.
- ***Low TCO***
	- Ideally serverless solution that can scale in and out flexibily without extensive administration and maintenance.
	- Acceptable costs for target solution capacity.

## Potential solutions

TBD

## Target solution

TBD


### JSON event data

Sample AWS EventBridge event:

```json
{
	'version': '0',
	'id': '93ea9825-264f-90fd-16a7-d640d156f5e9',
	'detail-type': 'Scheduled Event',
	'source': 'aws.events',
	'account': '287378523389',
	'time': '2022-01-24T11:51:18Z',
	'region': 'eu-central-1',
	'resources': ['arn:aws:events:eu-central-1:287378123456:rule/serverless-scheduler-SchedulerFunctionDispatchJobs-1SXUZFVOXE0EP'],
	'detail': {}
}
```

Sample DynamoDB payload whene **View DynamoDB JSON** is enabled:

```json
{
  "pk": {
    "S": "j#2015-03-20T09:45"
  },
  "sk": {
    "S": "2015-03-20T09:46:47.123Z#564ade05-efda-4a2e-a7db-933ad3c89a83"
  },
  "detail": {
    "M": {
      "action": {
        "S": "send-reminder"
      },
      "userId": {
        "S": "16f3a019-e3a5-47ed-8c46-f668347503d1"
      },
      "taskId": {
        "S": "6d2f710d-99d8-49d8-9f52-92a56d0c6b81"
      },
      "params": {
        "M": {
          "reminder_volume": {
            "N": "0.5"
          },
          "can_skip": {
            "BOOL": true
          }
        }
      }
    }
  },
  "detail_type": {
    "S": "job-reminder"
  }
}
```

Sample DynamoDB payload whene **View DynamoDB JSON** is disabled:

```json
{
  "pk": "j#2015-03-20T09:45",
  "sk": "2015-03-20T09:46:47.123Z#564ade05-efda-4a2e-a7db-933ad3c89a83",
  "detail": {
    "action": "send-reminder",
    "userId": "16f3a019-e3a5-47ed-8c46-f668347503d1",
    "taskId": "6d2f710d-99d8-49d8-9f52-92a56d0c6b81",
    "params": {
      "can_skip": false,
      "reminder_volume": 0.5
    }
  },
  "detail_type": "job-reminder"
}
```

### Partitions in general

We are using composite primary key (Details see [here](https://aws.amazon.com/blogs/database/choosing-the-right-dynamodb-partition-key/))

- **pk** represents the partition (can be shared by many items)
- **sk** represwnts sorting key. This value must be **uniqueue** for every item.

### Partitions calculation

See https://www.programiz.com/python-programming/online-compiler/

Script:

```python
from datetime import timedelta
import dateutil.parser

partition_interval_in_minutes = 5

def get_partitions(event_time_in_utc):
    date_hour, strminute, *_ = event_time_in_utc.split(':')
    
    print("date_hour=" + date_hour)
    print("strminute=" + strminute)
    
    minute = int(strminute)

    current_partition = f'{date_hour}:{(minute - minute % partition_interval_in_minutes):02d}'
    print("current_partition=" + current_partition)
    
    current_partition_date_time = dateutil.parser.parse(current_partition)
    print("current_partition_date_time=" + str(current_partition_date_time))
    previous_partition_date_time = current_partition_date_time - \
        timedelta(minutes=partition_interval_in_minutes)
        
    print("previous_partition_date_time=" + str(previous_partition_date_time))
    
    previous_partition = previous_partition_date_time.strftime(
        '%Y-%m-%dT%H:%M')

    print("previous_partition=" + previous_partition)

    return previous_partition, current_partition

previous_partition, current_partition = get_partitions("2022-01-24T11:57:12Z")
print("previous_partition=" + previous_partition)
print("current_partition=" + current_partition)
```

will produce following partitions:

```
date_hour=2022-01-24T11
strminute=57
current_partition=2022-01-24T11:55
current_partition_date_time=2022-01-24 11:55:00
previous_partition_date_time=2022-01-24 11:50:00
previous_partition=2022-01-24T11:50
previous_partition=2022-01-24T11:50
current_partition=2022-01-24T11:55
> 
```
## Links

Resources used for initial analysis

* https://dev.to/aws-builders/delayed-event-processing-part-1-40pa
* https://medium.com/swlh/how-to-get-past-the-15-minute-delay-limit-in-amazon-sqs-fba3c50daf0b
* https://theburningmonk.com/2019/03/dynamodb-ttl-as-an-ad-hoc-scheduling-mechanism/
* https://aws.amazon.com/blogs/architecture/serverless-scheduling-with-amazon-eventbridge-aws-lambda-and-amazon-dynamodb/
* https://dev.to/rohanmehta_dev/scheduled-task-processing-with-dynamodb-and-eventbridge-439c
* https://aws.amazon.com/premiumsupport/knowledge-center/primary-key-dynamodb-table/
* https://aws.amazon.com/blogs/database/choosing-the-right-dynamodb-partition-key/
* https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-sharding.html