+++
title = "Measure the Launch Speed of Instances"
weight = 180
+++


### Measure the Launch Speed of Instances Launched Directly into an Auto Scaling Group

In this step we will launch an instance directly into an Auto Scaling group. The Auto Scaling group we created `ec2-workshop-asg` uses lifecycle hooks to manage the application installation process. Using userdata script configured in the launch template that execute on the instance when the instance first boots, and every time the instance starts. This script installs and starts the application. Once the application is installed, a command is executed to complete the lifecycle action and allow the instance to transition to the next lifecycle step.

You can also use a Lambda function that executes in response to Amazon EventBridge events that are generated as instances transition through their lifecycle.

#### Increase Desired Capacity

Set the desired capacity of the Auto Scaling group to 1 to launch an instance directly into the Auto Scaling group.

```bash
aws autoscaling set-desired-capacity --auto-scaling-group-name "ec2-workshop-asg" --desired-capacity 1
```

#### Measure Launch Speed

Now, let's measure the launch speed of the instance. You will need to wait a few minutes for the instance to be launched by the previous step.

```bash
activities=$(aws autoscaling describe-scaling-activities --auto-scaling-group-name "ec2-workshop-asg")
for row in $(echo "${activities}" | jq -r '.Activities[] | @base64'); do
    _jq() {
     echo ${row} | base64 --decode | jq -r ${1}
    }

   start_time=$(date -d "$(_jq '.StartTime')" "+%s")
   end_time=$(date -d "$(_jq '.EndTime')" "+%s")
   activity=$(_jq '.Description')

   echo $activity Duration: $(($end_time - $start_time))"s"
done
```

#### Observe Launch Duration

Because the instance launched directly into the Auto Scaling group, all initialization actions needed to complete to prepare the instance to be placed in-service. From the results below we can see that these actions took a long time to complete, delaying how quickly our Auto Scaling group can scale.

```
Launching a new EC2 instance: i-075fa0ad6a018cdfc Duration: 243s
```