# Stomp Benchmark

A benchmarking tool for [Stomp](http://stomp.github.com) servers.

## Build Prep

* Install [sbt](http://code.google.com/p/simple-build-tool/wiki/Setup) but instead 
  of setting up the sbt script to use `sbt-launch.jar "$@"` please use `sbt-launch.jar "$*"` instead.
  
* run: `sbt update` in the stomp-benchmark project directory

## Running the Benchmark

The benchmark assumes that a Stomp 1.0 server is running on the local host on port 61613.
Use the `sbt run` command to execute the benchmark.  Run `sbt run --help` to get a listing
of all the command line arguments that the benchmark supports.

For each broker you are benchmarking you will typically execute:

    sbt run reports/foo-3.2.json

The benchmarking tool will then execute a large number of predefined 
usage scenarios and gather the performance metrics for each.  Those metrics
will get stored in a `reports/foo-3.2.json` file.  

## Updating the Report

The `reports/report.html` file can load and display the results of multiple benchmark runs.
You can updated which benchmark results are displayed by the report.html by editing
it and updating to the line which defines the `broker_files` variable (around line 32).

    var broker_files = ['foo-3.2.json', 'cheese-1.0.json']


### Running against Apollo 1.0-beta1

[Apollo](http://activemq.apache.org/apollo) is a new Stomp based 
message server from good folks at the [Apache ActiveMQ](http://activemq.apache.org/) 
project.

1. Follow the [getting started guide](http://activemq.apache.org/apollo/versions/1.0-beta1/website/documentation/getting-started.html) 
to install, setup, and start the server.

2. Run the benchmark with the admin credentials.  Example:

    sbt run --login admin --passcode password reports/ubuntu-intel-2600k/apollo-1.0-beta1.json

### Running against ActiveMQ 5.5.0

[ActiveMQ 5.5.0](http://activemq.apache.org) was the first Stomp Server implementation and as
such is sometimes considered to be the reference implementation for Stomp 1.0.

1. Update the `conf/activemq.xml` configuration file and add in the Stomp transport connector:

    <transportConnector name="stomp+nio" uri="stomp+nio://0.0.0.0:61613?transport.closeAsync=false"/>

2. Start the server by running:

    ./bin/activemq console

3. Run the benchmark using the default options.  Example:

    sbt run reports/ubuntu-intel-2600k/activemq-5.5.0.json

### Running against HornetQ 2.2.0.Final

[HornetQ](http://www.jboss.org/hornetq) provides native Stomp 1.0 Support.

1. Update the `config/stand-alone/non-clustered/hornetq-configuration.xml` file add 
stomp acceptor:
    <acceptor name="stomp-acceptor">
      <factory-class>org.hornetq.core.remoting.impl.netty.NettyAcceptorFactory</factory-class>
      <param key="protocol"  value="stomp"/>
      <param key="host"  value="0.0.0.0"/>
      <param key="port"  value="61613"/>
    </acceptor>      

2. Add the queues and topics used by the load test by updating the 
`config/stand-alone/non-clustered/hornetq-jms.xml` with the following:

    <queue name="loadq-0"><entry name="/queue/loadq-0"/></queue>
    <queue name="loadq-1"><entry name="/queue/loadq-1"/></queue>
    <queue name="loadq-2"><entry name="/queue/loadq-2"/></queue>
    <queue name="loadq-3"><entry name="/queue/loadq-3"/></queue>
    <queue name="loadq-4"><entry name="/queue/loadq-4"/></queue>
    <queue name="loadq-5"><entry name="/queue/loadq-5"/></queue>
    <queue name="loadq-6"><entry name="/queue/loadq-6"/></queue>
    <queue name="loadq-7"><entry name="/queue/loadq-7"/></queue>
    <queue name="loadq-8"><entry name="/queue/loadq-8"/></queue>
    <queue name="loadq-9"><entry name="/queue/loadq-9"/></queue>

    <topic name="loadt-0"><entry name="/topic/loadt-0"/></topic>
    <topic name="loadt-1"><entry name="/topic/loadt-1"/></topic>
    <topic name="loadt-2"><entry name="/topic/loadt-2"/></topic>
    <topic name="loadt-3"><entry name="/topic/loadt-3"/></topic>
    <topic name="loadt-4"><entry name="/topic/loadt-4"/></topic>
    <topic name="loadt-5"><entry name="/topic/loadt-5"/></topic>
    <topic name="loadt-6"><entry name="/topic/loadt-6"/></topic>
    <topic name="loadt-7"><entry name="/topic/loadt-7"/></topic>
    <topic name="loadt-8"><entry name="/topic/loadt-8"/></topic>
    <topic name="loadt-9"><entry name="/topic/loadt-9"/></topic>

3. Start the server by running:

    cd bin
    ./run.sh

4. Run the benchmark with the `--topic-prefix` and `--queue-prefix` options.  For
example:

    sbt run --topic-prefix=jms.topic. --queue-prefix=jms.queue. reports/ubuntu-intel-2600k/hornetq-2.2.0.Final.json

### Running against RabbitMQ 2.4.1

[RabbitMQ](http://www.rabbitmq.com/) is an erlang based message server..

1. Install RabbitMQ and the [Stomp Plugin](http://www.rabbitmq.com/plugins.html#rabbitmq-stomp).

2. Run the benchmark with the default guest credentatials and the --persistent-header option.  Example:

    sbt run --login guest --passcode guest reports/ubuntu-intel-2600k/rabbitmq-2.4.1.json


## Running a Custom Scenario

If there is a particular scenario you want to manually execute against the 
broker, you can do so by starting up the Scala interactive interpreter by
running `sbt console`

Then at the console you execute:

    scala> val scenario = new com.github.stomp.benchmark.NonBlockingScenario
    scenario: com.github.stomp.benchmark.Scenario = 
    --------------------------------------
    Scenario Settings
    --------------------------------------
      host                  = 127.0.0.1
      port                  = 61613
      destination_type      = queue
      destination_count     = 1
      destination_name      = load
      sample_interval (ms)  = 1000
  
      --- Producer Properties ---
      producers             = 1
      message_size          = 1024
      persistent            = false
      sync_send             = false
      content_length        = true
      producer_sleep (ms)   = 0
      headers               = List()
  
      --- Consumer Properties ---
      consumers             = 1
      consumer_sleep (ms)   = 0
      ack                   = auto
      selector              = null
      durable               = false

This creates a new NonBlockingScenario object which you can adjust it's properties and
then run by executing `scenario.run`.  For example, to run 10 producers and no
consumer on a topic, you would update the scenario object properties as follows:

    scala> scenario.producers = 10

    scala> scenario.consumers = 0

    scala> scenario.destination_type = "topic"
    
When you actually run the scenario, you it will report back the throughput metrics.
Press enter to stop the run.

    scala> scenario.run                                          
    --------------------------------------
    Scenario Settings
    --------------------------------------
      host                  = 127.0.0.1
      port                  = 61613
      destination_type      = topic
      destination_count     = 1
      destination_name      = load
      sample_interval (ms)  = 1000
  
      --- Producer Properties ---
      producers             = 10
      message_size          = 1024
      persistent            = false
      sync_send             = false
      content_length        = true
      producer_sleep (ms)   = 0
      headers               = List()
  
      --- Consumer Properties ---
      consumers             = 0
      consumer_sleep (ms)   = 0
      ack                   = auto
      selector              = null
      durable               = false
    --------------------------------------
         Running: Press ENTER to stop
    --------------------------------------

    Producer total: 345,362, rate: 345,333.688 per second
    Producer total: 725,058, rate: 377,908.125 per second
    Producer total: 1,104,673, rate: 379,252.813 per second
    Producer total: 1,479,280, rate: 373,913.750 per second
    ... <enter pressed> ...
    
    scala>
