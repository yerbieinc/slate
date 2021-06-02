# Quick Start

### Prerequisites

- Docker
- Redis (>= 3.0)

1. Pull the Yerbie image locally by running `docker pull yerbie/yerbie-server`.
2. Start the Yerbie server with `docker run yerbie/yerbie-server`.

Yerbie assumes Redis is accessible on `localhost` port `6379` and will start listening for requests on port `5865`.

## Library 
The client library exposes an easy to use interface to send requests to Yerbie and to run jobs from a queue when they become available.

> Add the Yerbie client library as a dependency with the latest version:

```java
// Yerbie is available on the Maven Central Repository
<dependencies>
    <dependency>
        <groupId>dev.yerbie</groupId>
        <artifactId>yerbie-java</artifactId>
        <version>1.1.0</version>
    </dependency>
</dependencies>
```

## Job Data
In order to store relevant data that your job can access when it runs, you need to create a serializable representation of that data in JSON.
The maximum data size is 512 megabytes.

> Create your own job data:

```java
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;

public class TestJobData {
  private final String username;

  @JsonCreator
  public TestJobData(@JsonProperty("username") String username) {
    this.username = username;
  }

  public String getUsername() {
    return username;
  }
}
```

## Jobs
A job is a simple interface that has a `run` method. When job data is pulled from a queue, a job instance is created and the `run` method is invoked with the data as a parameter.

> Create a job:

```java
import yerbie.job.Job;
import your.org.jobdata.TestJobData;

public class TestJob implements Job<TestJobData> {

  @Override
  public void run(TestJobData testJobData) {
    System.out.println("Hello " + testJobData.getUsername());
  }
}
```

## Client Initialization

By default, a Yerbie server listens for requests on port `5865`.

> To initialize a client:

```java
import yerbie.client.ClientProvider;
import yerbie.client.YerbieClient;

ClientProvider clientProvider =
  new ClientProvider(
    "localhost", 5865);

// The client takes in an ObjectMapper to perform serialization and deserialization of job data.
YerbieClient yerbieClient = clientProvider.initializeClient(new ObjectMapper());
```

## Scheduling Jobs

At its simplest, Yerbie takes a delay in seconds, a queue name, and an instance of a job data.

The example tells Yerbie to make `TestJobData` available on `high_priority_queue` in 10 seconds.

Yerbie can easily handle delays of days or even weeks, but be cognizant that any changes to `TestJobData` will need to support old versions of the data.

Yerbie supports retrying on failure, and using this method will use the default retry mechanism of 30 seconds in between retries, with a total of 4 retries. This means that the job can run at most 5 times (the first run does not count as a retry), before no longer running.

> To schedule a job in 10 seconds to queue high_priority_queue:

```java
import your.org.jobdata.TestJobData;
import yerbie.serde.JSONJobData;

yerbieClient.scheduleJob(Duration.ofSeconds(10), "high_priority_queue", new TestJobData("naota_nandaba"));
```

## Deleting Jobs

To delete a job that is going to be executed, Yerbie allows you to delete a job via its job token. A job that is already enqueued into a queue cannot be deleted.
The client returns `true` if succesful, or `false` if it can't find the job (or it has already been enqueued).

> To delete a job with token `aa555552-c21c-4d0a-a7a0-713f2b02b169`:

```java
boolean result = yerbieClient.deleteJob("a555552-c21c-4d0a-a7a0-713f2b02b169");
```

## Running Workers

This tells the library to start polling Yerbie for available work.

If you've been following along, with our test job data and job implementation, this job will print `Hello naota_nandaba` to the system output.

> To run a worker:

```java
import yerbie.client.JobRepositoryBuilder;
import yerbie.client.YerbieConsumer;
import yerbie.client.ClientProvider;
import your.org.jobdata.TestJobData;
import your.org.job.TestJob;

JobRepositoryBuilder jobRepositoryBuilder = new JobRepositoryBuilder();
jobRepositoryBuilder.withJobData(TestJobData.class, TestJob::new);

ClientProvider clientProvider =
  new ClientProvider(
    "localhost", 5865);

YerbieConsumer yerbieConsumer =
  clientProvider.initializeYerbieConsumer("high_priority_queue", jobRepositoryBuilder.build());

yerbieConsumer.start();
```

> Your worker will now poll on the `high_priority_queue`. When it receives a `TestJobData`, it creates a new instance of `TestJob` and will invoke the `run` function with the job data.
