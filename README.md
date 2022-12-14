# NCC Group's AWS-Inventory

AWS-Inventory is a tool that is developed and maintained by NCC Group to discover and list resources from every service 
and region in an AWS account.

## Introduction

This is a tool that tries to discover all 
[AWS resources](https://docs.aws.amazon.com/general/latest/gr/glos-chap.html#resource) created 
in an account. AWS has many offerings (a.k.a. services) with new ones constantly being added and existing ones expanded 
with new features. The ecosystem allows users to piece together many different services to form a customized cloud 
experience. The ability to instantly spin up services at scale comes with a manageability cost. It can quickly become 
difficult to audit an AWS account for the resources being used. It is not only important for billing purposes, but also 
for security. Dormant resources and unknown resources are more prone to security configuration weaknesses. 
Additionally, resources with unexpected dependencies pose availability, access control, and authorization issues.

AWS Inventory uses [botocore](https://github.com/boto/botocore) to discover 
[AWS services](https://botocore.readthedocs.io/en/latest/reference/index.html) and what 
[regions](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) they run in. It is 
also used for invoking the service APIs. The invoked APIs are those which should list or describe resources. 
The results can be printed to stdout in JSON format. They can also be written across several files:

* Raw responses from API endpoints can be written to a file specified on the commandline. The file format is [Python pickle](https://docs.python.org/3/library/pickle.html).
* Exceptions raised during tool execution can be written to a file specified on the commandline. The file format is [Python pickle](https://docs.python.org/3/library/pickle.html).
* gui/aws_inventory_data-&lt;environment_name&gt;.json - JSON format. Parsed responses structured for input to the GUI.

## Installation 

1. Ensure that Python 3.5+ and `pip` are installed on your machine.
2. Clone this repository onto your machine.
2. Install the prerequisites with `pip install -r requirements.txt`
3. Install this repository by calling `pip install .` from the root of the cloned repository.

# Usage

You can run the script without any parameters. It will search for your AWS creds in your shell environment, 
instance metadata, config file, then credentials file. You can also provide a CSV file, containing your creds, on the 
commandline. You will want a user that has permissions like the AWS managed policy 
[ViewOnlyAccess](arn:aws:iam::aws:policy/job-function/ViewOnlyAccess). If you are feeling lucky, you could just pipe 
the output of the tool to a JSON parser like *jq*.

The tool could take a long time (dozens of minutes) to complete if no restrictions are placed on which operations to 
invoke for each service across each region. Filtering by service and region can be done on the commandline while 
filtering by service operation can be done via configuration file. A [pre-configured file](operation_blacklist.conf) 
was created and checked into the repository. It will be used by default. 

Aside from the commandline output, you can view the results locally in a [React](https://reactjs.org/) 
[single-page app](https://en.wikipedia.org/wiki/Single-page_application). No web server needed. Just open the 
[HTML file](gui/dist/index.html) in a browser and select the generated JSON file when prompted.  

The app uses [jsTree](https://www.jstree.com/) to display the data in a hierarchical, tree-like structure. There is 
also a search feature.

**NOTE:** When invoking APIs, if an API returns an exception then it won't be called again for the other regions. The 
known sources of exceptions are:

* Required API parameter(s) not included in the request
* Lack of authorization for calling the API (check the policies of your IAM User/Role)
* Network errors

## Examples

* Run with defaults.

`$ python aws_inventory.py`

* List AWS services known to *botocore*. This is all done locally by reading service model files.

```
$ python aws_inventory.py --list-svcs
acm
apigateway
application-autoscaling
appstream
autoscaling
batch
budgets
clouddirectory
cloudformation
cloudfront
.
.
.
```

- List service operations known to *botocore*. This is all done locally by reading service model files.

```
$ python aws_inventory.py --list-operations
[shield]
DescribeSubscription
ListAttacks
ListProtections

[datapipeline]
ListPipelines

[firehose]
ListDeliveryStreams
.
.
.
[glacier]
# NONE

[stepfunctions]
ListActivities
ListStateMachines

Total operations to invoke: 4045
```

* Print what APIs would be called for a service. This is all done locally.

`$ python aws_inventory.py --debug --dry-run`

# Screenshots

![invoking apis on commandline](screenshots/invoking%20apis%20on%20commandline.png)

![data in browser](screenshots/data%20in%20browser.png)
