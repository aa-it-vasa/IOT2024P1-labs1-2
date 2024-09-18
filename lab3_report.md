# System Architecture of IoT - Lab 3 report

**Description:** _Establish a MQTT connection that is safe from (unintended) snooping and is private using a gateway._

**Group members:** _Fill group member names here_

**Lab dates:** _Fill out the dates you did your labs here_

## Work description (3 p)

Provide a short description of the setup (hardware and software) and how the different components interact. Explain the steps you performed in the lab, e.g., using screenshots and photos including the received messages on the AWS dashboard.

Explain how the Thing connects to your gateway, including TLS mutual authentication. Use relevant diagrams. Check the folder groupCA in your directory. 

## Measure the latency of establishing the connection to the gateway (7 p)

Modify `pubSub.py` to measure the latency of establishing the connection to the gateway. Use `time`
   module and use `perf_counter()` to measure the time. See
   [documentation](https://docs.python.org/3/library/time.html#time.perf_counter). Compare the measured time with the ping value optained from the VM to your gateway. 

## Optional tasks

If you are doing the optional tasks, write your results here.