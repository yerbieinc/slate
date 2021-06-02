# Retrying

To initiate a job, the client library requests a job from a specific queue. Once popped, the library has 10 seconds to send an ack to Yerbie to acknowledge that it successfully ran the job. In the case where it doesn't run the job successfully, Yerbie will make the job available again.

In the case where an exception is thrown, the client library will re-queue the job according to the job's retry policy.

If you don't specify a retry policy when creating a job, Yerbie uses the default of a fixed interval 30 second retry policy with 4 total retries. This means that the job will run unsuccessfully at most 5 times (since the first run is not a retry).

## Fixed Interval Retry Policy
This retry policy will make the job available on the queue in 30 seconds, and will retry 4 times, each waiting 30 seconds after the initial run if the job is still unsuccessful.

```java
import yerbie.job.FixedRetryPolicy;

long initialDelaySeconds = 30;
int totalRetries = 4;
yerbieClient.scheduleJob(Duration.ofSeconds(10), "high_priority_queue", new TestJobData("naota_nandaba"), new FixedRetryPolicy(initialDelaySeconds, totalRetries));
```

## Exponential Retry Policy
This retry policy will make the job available after an exception in 5 seconds, then 25, then 125, then 625 before stopping.

```java
import yerbie.job.ExponentialRetryPolicy;

long initialDelaySeconds = 5;
int totalRetries = 4;
yerbieClient.scheduleJob(Duration.ofSeconds(10), "high_priority_queue", new TestJobData("naota_nandaba"), new ExponentialRetryPolicy(initialDelaySeconds, totalRetries));
```

