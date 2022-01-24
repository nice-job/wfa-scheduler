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

## Links

Resources used for initial analysis

* https://dev.to/aws-builders/delayed-event-processing-part-1-40pa
* https://medium.com/swlh/how-to-get-past-the-15-minute-delay-limit-in-amazon-sqs-fba3c50daf0b
* https://theburningmonk.com/2019/03/dynamodb-ttl-as-an-ad-hoc-scheduling-mechanism/
* https://aws.amazon.com/blogs/architecture/serverless-scheduling-with-amazon-eventbridge-aws-lambda-and-amazon-dynamodb/
* https://dev.to/rohanmehta_dev/scheduled-task-processing-with-dynamodb-and-eventbridge-439c