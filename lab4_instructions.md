# Lab 4 - Edge compute with Lambda

## Overview

In the previous lab, we organised Things using gateways and ensured that there
were no unintended cross-talk between devices of different groups. In this lab,
we will extend gateway's usefulness to perform close-to-sensor computation.
Close-to-sensor computation, or edge-computation, is a new trend to manage
high-bandwidth data emerging from the sensors. By processing the data at the
edge, either completely or partially, we reduce the communication burden on the
network infrastructure. Also, for a latency critical application a gateway
reduces the response time between events and manages the risk related to
network outages.

There are several concerns with edge-computation. Unlike in the lab, you can
not always rely on the local network to SSH into the device. So a remote deployment
and management scheme is necessary. Also, the logic may vary from one gateway
to another. So we need a solution that scales and handles diverse computational
needs. 

Web-servers on gateways and a version control should meet most of the concerns
expressed above. Coupled with a dashboard to manage all the gateways, this solution
should scale if more gateways are added. AWS Lambda is a similar solution, but
without the need to setup a web-server or a dashboard. As a developer, you
would write the logic using one of the supported languages (Python 3 in this case) and deploy them on
the gateway along with the device certificates, policies etc. 

The programming model of Lambda is event driven, similar to the subscription
mechanism of MQTT. Every time a message formatted as JSON document is sent to
the lambda either through MQTT or HTTPS, an event is triggered and the function
associated with the event is spawned in a virtual container. This on-demand
event driven function runs without the need of setting up a server. A
side-effect is that in the case of multiple events, multiple independent
Lambdas are invoked and managed. AWS calls this pattern as "event-driven
serverless computing". If you are interested in running a single Lambda for all
the events, then Lambda can also be configured to run in this fashion. 

AWS Lambda is suitable for running small on-demand applications that are
reactive in nature. For a heavy duty logic that process multiple streams from
different sources, you may want to forward the data to the cloud and process
them there. The course on Cloud Computing handles this topic in detail.

In this lab, we will deploy a Lambda function on the gateway and make it
respond to events sent by cloud as well as devices in the group.

## Run Greengrass Daemon

Make sure Greengrass core is running on your RPI. For example check the AWS Console under _AWS IoT > Manage > Greengrass devices > Core devices_. It the status is not `Healthy` and the status reported is less than the amount of time since you rebooted the RPI, then reboot the PI, check e.g., the 
status of the Greengrass service
```
rpi> sudo systemctl status greengrass
```
If it is stopped start it with 
```
rpi> sudo systemctl start greengrass
```
You can also check the Greengrass log files on the PI with 
```
rpi> sudo cat /greengrass/v2/logs/greengrass.log
```

If you cannot get it working, check with the lab assistant or recreate the Greengrass core device like in Lab 3.

## Deploy AWS Lambda on AWS Greengrass Core

You are now ready to configure and deploy the Lambda function for AWS
Greengrass.

### Create and Package a Lambda Function

In order for a Python Lambda function to run on a AWS Greengrass device, it
must be packaged with specified folders from the Python AWS Greengrass core
SDK. This step has already been done and you can find an archive
`hello_world_python_lambda.zip` in the folder `SourceCode\Lab4` in the Github repository. You can view the
contents of this archive to verify that `lambda_function.py` is packaged
with AWS Greengrass core SDK, which forms the external dependency.  You are now
ready to upload your Lambda function `.zip` file to the AWS Lambda console.

1. From the AWS Management console, under _Services > Compute_, click _Lambda_. 

2. From menu on the left, click _Dashboard_. Click _Create Function_.

3. Choose _Author from scratch_ (this option may already be selected).

4. Give a unique name for your function, e.g. `HelloWorld_GROUPNAME`, and
   choose `Python 3.7` as _Runtime_. Select _Use an existing role_ and select `service-role/DefaultLambdaRole`
   in the _Existing Role_ field. Choose _Create function_.

5. On the _Code source_ section, choose _Upload from .zip file_ and then upload
   `hello_world_python_lambda.zip`.

6. Under _Runtime settings_ (scroll down) make sure that _Python 3.7_ is selected. You will also need to update the _Handler_ field. This is the function that is executed first when the lambda function is triggered: Press _Edit_, then change the field _Handler_ into `greengrassHelloWorld.function_handler`. 

   Think a moment about what `greengrassHelloWorld.function_handler` corresponds to in the code source window. 

7. Now is a good time to go through the code in `greengrassHelloWorld.py`. Change the publish topic to `saiot/GROUPNAME/localtocloud`, where `GROUPNAME` is a unique identifier, and write this down.

8. When you are ready to deploy this version press the button _Deploy_. 
   
9. To check that there are now issues with the code, you can select the _Test tab_. Note that this is not the same as the Test-button in the code source box. press the _Test_ button. Also you might need to change the _Event JSON_ input which corresponds to the input given to the lambda function.

10. When you are ready to publish a version of your lambda function (a published version can be access by other parts of AWS) select _Actions > Publish new version_. Write a description in the _Version description_ field, such as _First version_ (or leave it empty), then select _Publish_.

### Configure Lambda for AWS Greengrass

1. In the AWS IoT console, go to _AWS IoT > Manage > Greengrass devices > Components_. Press _Create component_.

2. Select _Import Lambda function_.

3. Search for the name of the Lambda you created in the previous step, and select it. 

4. For the version, choose the latest version you created.

5. Under _Event sources_ we need to setup the bindings for when the Lambda function should be executed. Press _Add event source_ and set _Topic_ to `saiot/GROUPNAME/localtocloud` and _Type_ to `Local publish/subscribe`. Press _Add event source_ again and set _Topic_ to `saiot/GROUPNAME/cloudtolocal` and _Type_ to `AWS IoT Core MQTT`. The first one will be used to send the message from the VM and the second from the AWS console. 

5. Set the _Timeout_ to `10` seconds. 

6. Select `True` under _Pinned_. A *long-lived (pinned) Lambda function* starts automatically after AWS Greengrass starts and keeps running in its own container (or sandbox). 

   Repeatedly triggering the handler of a *long-lived Lambda function* might queues up responses from the AWS IoT Greengrass core. This is in contrast to an *on-demand (not pinned) Lambda function* which might create a new container for a new invocation if the handler in previsouly created containers are still busy (i.e. processing data).

   After this session, if you have time, you can change the settings and check the difference in behaviour. You can also have a look at these two examples: [here](https://docs.aws.amazon.com/greengrass/latest/developerguide/long-testing.html) and [here](https://docs.aws.amazon.com/greengrass/latest/developerguide/on-demand.html).

7. Press _Create component_.

### Configure clients

An AWS Greengrass Lambda function can subscribe to or publish messages (using MQTT protocol):

* To and from other devices within the AWS Greengrass core.
* To other Lambda functions.
* To the AWS IoT cloud.

We will now setup the Greengrass core running on the RPI so that the messages are passed from the cloud to the edge (and the other way):

1. Select _AWS IoT > Manage > Greengrass devices > Deployments_. Select the checkmark for the deployment of your Greengrass core in the list (the _Target name_ column should be the name of your core device) and press _Revise_.

2. Press _Next_ to advance to _Step 2: select components_. Select your Lambda function under _My components_. Under _Public components_ we need to add `aws.greengrass.LegacySubscriptionRouter` to the list of activated components. Deactive the switch _Show only selected components_ and select `aws.greengrass.LegacySubscriptionRouter`. Press _Next_.

3. Select `aws.greengrass.clientdevices.mqtt.Bridge` and press _Configure component_. The box _Configuration to merge_ should be changed to:
   ```
   {
	   "mqttTopicMapping": {
         "GroupName_LocalToCloud": {
            "topic": "saiot/GROUPNAME/localtocloud",
            "source": "LocalMqtt",
            "target": "IotCore"
         },
         "GroupName_CloudToLocal": {
            "topic": "saiot/GROUPNAME/cloudtolocal",
            "source": "IotCore",
            "target": "LocalMqtt"
         }
      }
   }
   ```
   Remember to update the topic to the correct one for your group. Press _Confirm_. 

4. Select `aws.greengrass.LegacySubscriptionRouter` and press _Configure component_. The box _Configuration to merge_ should be changed to:
   ```
   {
	   "subscriptions": {
         "Greengrass_LocalToCloud": {
            "id": "Greengrass_LocalToCloud",
            "source": "component:yourlambdafunction",
            "subject": "saiot/GROUPNAME/localtocloud",
            "target": "cloud"
         }
      }
   }
   ```
   Remember to update the topic to the correct one for your group. Also the `yourlambdafunction` should be changed to the name of the Lambda function in Step 2. Press _Confirm_, _Next_, _Next_ and _Deploy_. 

4. Wait until the changes are updated on your RPI (might take a minute or two). Good idea run  
   ```
   rpi> sudo tail -f /greengrass/v2/logs/greengrass.log
   ```
   to make sure there are no errors. 

## Verify the Lambda Function is Running on the Device

If the Lambda function is deployed correctly, there should be a file created in the log directory named `lambdaname.log`.
```
rpi> sudo ls -als /greengrass/v2/logs/
rpi> sudo tail -f /greengrass/v2/logs/lambdaname.log
```
Check that there are no errors in the log files and that the log file for the lambda is updated. If there are some issues (e.g., your Python code crashes, you might need to restart the Greengrass service by issuing 
```
rpi> sudo systemctl restart greengrass
```

From _AWS IoT_ -> _Test_ -> _MQTT test client_, setup a new subscriber to the topic _saiot/GROUPNAME/localtocloud_. Select _Display payloads as strings (more accurate)_ option and then _Subscribe_.

1. From the left pane of the AWS IoT console, choose _Test_
	
2. Setup the subscriber connecting on the topic you used in step 1 of the previous section Configure clients (hello/world).

3. Publisher will send message to the topic used in step 2 of the previous section Configure clients.

Each publish is triggering the function handler and creating a new container for each invocation. The invocation count need not increment for every trigger because each on-demand Lambda function has its own container/sandbox. If you publish and trigger the function after waiting for timeout period (default 1s), you will see that the invocation count is incremented.

This shows that a container, first created from a prior invocation, is being reused, and pre-processing variables outside of function handler have been used.
	
## To do
 
1. Understand the `lambda_function.py` code that you had packaged into a Lambda function above.
2. Connect the simulated device that you created in the last lab, with the Lambda function. The setup you are looking at is given below. You can use `pubsub.py` to publish messages from your device (see Lab 3).
    1. Device -> Lambda. Device sends message on topic `saiot/GROUPNAME/localtolambda`. Tip: you will need to update some of the settings for the component and deployment above.
    2. Lambda -> IoT Cloud. Lambda parses the message sent by the device and forwards it to IoT Cloud on topic `saiot/GROUPNAME/localtocloud`.
3. In your report describe the achieved architecture and behavior of your application. Use figures to illustrate your description. (5p)
4. In your report describe the behavior differences between a long-lived Lambda function and an on-demand Lambda function deployed on a gateway. Illustrate your response taking a simple application example and provide the corresponding sequence (UML) diagrams for each. (5p)

## Optional tasks

### Asset Monitor

We will make a latency critical application for the following scenario. The scenario has a sensor that measures the temperature of an important asset and sends the value to the edge. If the temperature goes beyond 40 C, the edge responds by actuating an alarm and by indicating the cloud that alarm has been sounded. In addition to viewing the logs, a user at the cloud can also issue the command "TEMP" to get the current temperature of the asset.

You need to create a new an alarm actuator that acts as a subscriber and you can use `pubsub.py` to simulate the alarm by using it as subscriber mode. Read the code to find out how to do this. On successful actuation of the alarm, the alarm device prints out a device indicating that the alarm has been actuated. You may have to modify `pubsub.py` suitably.

To send the temperature, create a modified version of `pubsub.py` (or create a bash script that periodically sends temperatures using the normal `pubsub.py` command).

All your topics should begin with `saiot/GROUPNAME` to avoid clashes with other groups' topics.

Modify the initial Lambda code as you seem fit and establish necessary forward and backward connections to implement the scenario. 

### Hints

1. This is a considerably long task and would require you to chart out the devices, paths and topics on a piece of paper. 
2. Debate what would be a good model to run Lambda functions. Would it be _On Demand_ or _Long running_?
3. Lambda functions prints a log on Raspberry Pi gateway and this can serve as a good way to debug your implementations. 
   ```
   rpi> sudo cat /greengrass/v2/logs/greengrass.log
   rpi> sudo cat /greengrass/v2/logs/LAMBDANAME.log
   ```
