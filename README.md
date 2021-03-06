# RealTimeAnalyticsDashboard

Real-time analytics dashboard to visualize the number of orders getting shipped every minute to improve the performance of their logistics of a Ecommerce company

## Overview:

Built a real-time analytics dashboard to visualize the number of orders getting shipped every minute to improve the performance of their logistics of a ecommerce company.

## Technology stack

![](stack.png)

<table>
<thead>
<tr>
<th>Area</th>
<th>Technology</th>
</tr>
</thead>
<tbody>
	<tr>
		<td>Front-End</td>
		<td> HTML5, Bootstrap, CSS3, Socket.IO, Highcharts.js </td>
	</tr>
	<tr>
		<td>Back-End</td>
		<td>Express.js , Node.js</td>
	</tr>
  <tr>
		<td>Cluster Computing Framework</td>
		<td>Apache Spark (Python)</td>
	</tr>
	<tr>
		<td>Message Broker</td>
		<td>Apache Kafka</td>
	</tr>
	<tr>
		<td>Environment</td>
		<td>CloudxLab</td>
	</tr>
</tbody>
</table>

## Architecture

![](architecture.png)

 **Step1:**
 **Data set containing CSV files**

 Since we do not have an online e-commerce portal in place, we took a dataset containing CSV files for simulating a ecommerce portal.

 An entry in the CSV file is in below format

 DateTime, OrderId, Status

  2016-07-13 14:20:33,xxxxx-xxx,processing

  2016-07-13 14:20:34,xxxxx-xxx,shipped

  2016-07-13 14:20:35,xxxxx-xxx,delivered

 **Step2:**
 **Creation of a Topic using Apache Kafka**

 Command to create topic 

 kafka-topics.sh --create --zookeeper zookeeperhostname:port --replication-factor 1 --partitions 1 --topic topicname
 
 Push Dataset to Kafka topic
 
 using the shell script to serve the purpose

 /bin/bash put_order_data_in_topic.sh ../data/order_data/ zookeeperbrokerhostname:port topicname

 In the above command put_order_data_in_topic.sh is a shell script which takes broker hostname:port and topic name as input as send data into the topic.

</br>

 **Step3:**
 **Spark Streaming and Kafka integration**

 Spark streaming code takes data from Kafka topic in a window of 60 seconds, process it so that we have the total count of each unique order status in that 60 seconds window. After processing the total count of each unique order status gets pushed to new Kafka topic.

command to create topic

 kafka-topics.sh --create --zookeeper zookeeperhostname:port --replication-factor 1 --partitions 1 --topic topicname
 
 Open spark_streaming_order_status.py 

 change order-one-min-data to your new one minute Kafka topic

 Replace localhost with the hostname where zookeeper server is running

 Finally submitting a spark Job

 spark-submit --jars spark-streaming-kafka-assembly_2.10-1.6.0.jar spark_streaming_order_status.py zookeeperhostname:port topic used for  csv files.

 **Step4:**
 **now one minute topic has info like below**

 {
    "shipped": 657,
    "processing": 987,
    "delivered": 1024
 }

  Open index.js 

  Replace localhost with zookeeper server hostname

  Replace order-min-data with your one minute topic name
  
  Save the file

 Run Node.js server  and then access dashboard from browser 

 Command:

 node index.js
 
**Initial output:**

![](initial.png)

**Later:**

 As soon as a new message is available in the one minute Kafka topic, node process consumes it. The consumed message then gets emitted to the web browser via Socket.IO

 As soon as socket.io-client in the web browser receives a new ‘message’ event, data in the event gets processed. If the order status is “shipped” in the received data, it gets added to HighCharts series and gets displayed on the browser.

![](final.png)
