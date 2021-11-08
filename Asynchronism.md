# Asynchronism
1. Asynchronous workflows help reduce request times for expensive operations that would otherwise be performed in-line. They can also help by doing time-consuming work in advance, such as periodic aggregation of data.

## Message queues
1. Message queues receive, hold, and deliver messages. If an operation is too slow to perform inline, you can use a message queue with the following workflow:
   * An application publishes a job to the queue, then notifies the user of job status
   * A worker picks up the job from the queue, processes it, then signals the job is complete
1. The user is not blocked and the job is processed in the background. During this time, the client might optionally do a small amount of processing to make it seem like the task has completed. For example, if posting a tweet, the tweet could be instantly posted to your timeline, but it could take some time before your tweet is actually delivered to all of your followers.

1. Redis is useful as a simple message broker but messages can be lost.
1. RabbitMQ is popular but requires you to adapt to the 'AMQP' protocol and manage your own nodes.
1. Amazon SQS is hosted but can have high latency and has the possibility of messages being delivered twice.

## Task queues
1. Tasks queues receive tasks and their related data, runs them, then delivers their results. They can support scheduling and can be used to run computationally-intensive jobs in the background.

1. Celery has support for scheduling and primarily has python support.

1. To put it simply: Task or message queues, they can be thought of or used interchangeably. It's the asynchronous operation that matters. (repeat that last line to yourself :)) The point of having a queue is that one guy can ask to do something or say something and forget about it, and another guy can follow up on it.

1. A task/message queue: Think of this like a passive data structure which acts like a queue, it 'stores' tasks and keeps them for processing later on. True, there is a process who manages this queue, but it doesn't do more, it just stores things and gives them when asked in an orderly way.

1. Righto, great. But who processes it? The worker threads.
1. Okay. So now, we have got 3 things.
   * A task/message.
   * A process who adds the tasks in to the task queue (data structure), and maintains this task queue.
   * Worker processes who take the tasks/messages from this queue and execute them.

1. So you mean workers can read this queue and remove tasks from them? Yes they can, but this leads to problems (er.. race conditions and such).
1. Enter the 'message broker'. The 'message broker' is the one who takes the tasks from the task queue and distributes them to worker processes (even if they are on different machines) and manages the integrity of task completion, retrying tasks if workers fail, giving faster workers more tasks, and some other housekeeping for distributed processing.

1. Aha! So that's what the message broker does! Wait, why can't the 'Task/Message queue' manage the giving away of tasks to these workers, instead of just managing the queue?
1. They can and do so, this is why there is confusion sometimes. Redis, primarily, a high performance passive data store, also does message broking/scheduling
Some message brokers (like rabbitMQ) use their own task queues to store and schedule them.
The roles/functions may overlap, but the concept is essentially the same.


## Back pressure
1. How should a system respond when under sustained load?  Should it keep accepting requests until its response times follow the deadly hockey stick, followed by a crash?
2. Having the entire system degrade is not the ideal service we want to give our customers.  A better approach would be to process transactions at our systems maximum possible throughput rate, while maintaining a good response time, and rejecting requests above this arrival rate.

3. If queues start to grow significantly, the queue size can become larger than memory, resulting in cache misses, disk reads, and even slower performance. Back pressure can help by limiting the queue size, thereby maintaining a high throughput rate and good response times for jobs already in the queue. Once the queue fills up, clients get a server busy or HTTP 503 status code to try again later. Clients can retry the request at a later time, perhaps with exponential backoff.
1. Within our systems the available capacity is generally a function of the size of our thread pools and time to process individual transactions.  These thread pools are usually fronted by queues to handle bursts of traffic above our maximum arrival rate.  If the queues are unbounded, and we have a sustained arrival rate above the maximum capacity, then the queues will grow unchecked. We can do this by designing our systems to apply “Back Pressure”.

2. Let’s assume we have asynchronous transaction services fronted by an input and output queues, or similar FIFO structures.  If we want the system to meet a response time quality-of-service (QoS) guarantee, then we need to consider the three following variables:
   * The time taken for individual transactions on a thread
   * The number of threads in a pool that can execute transactions in parallel
   * The length of the input queue to set the maximum acceptable latency
    max latency = (transaction time / number of threads) * queue length
    queue length = max latency / (transaction time / number of threads)

By allowing the queue to be unbounded the latency will continue to increase.  So if we want to set a maximum response time then we need to limit the queue length.
By bounding the input queue we block the thread receiving network packets which will apply back pressure up stream.  If the network protocol is TCP, similar back pressure is applied via the filling of network buffers, on the sender.  This process can repeat all the way back via the gateway to the customer.  For each service we need to configure the queues so that they do their part in achieving the required quality-of-service for the end-to-end customer experience.

One of the biggest wins I often find is to improve the time taken to process individual transaction latency.  This helps in the best and worst case scenarios.

### Worst Case Scenario
   * Let’s say the queue is unbounded and the system is under sustained heavy load.  Things can begin to go wrong very quickly in subtle ways before memory is exhausted.  What do you think will happen when the queue is larger than the processor cache?  The consumer threads will be suffering cache misses just at the time when they are struggling to keep up, thus compounding the problem.  This can cause a system to get into trouble very quickly and eventually crash.  Under Linux this is particularly nasty because malloc, or one of its friends, will succeed because Linux allows “Over Commit” by default, then later at the point of using that memory, the OOM Killer will start shooting processes. When the OS starts shooting processes, you just know things are not going to end well!

### What About Synchronous Designs?
1. You may say that with synchronous designs there are no queues.  Well not such obvious ones.  If you have a thread pool then it will have a lock, or semaphore, wait queues to assign threads.  If you are crazy enough to allocate a new thread on every request, then once you are over the huge cost of thread creation, your thread is in the run queue for a processor to execute.  Also, these queues involve context switches and condition variables which greatly increase the costs.  You just cannot run away from queues, they are everywhere!  Best to embrace them and design for the quality-of-service your system needs to deliver to its customers.  If we must have queues, then design for them, and maybe choose some nice lock-free ones with great performance.
1. When we need to support synchronous protocols like REST then use back pressure, signalled by our full incoming queue at the gateway, to send a meaningful “server busy” message such as the HTTP 503 status code.  The customer can then interpret this as time for a coffee and cake at the café down the road.

### Subtleties To Watch Out For...
1. You need to consider the whole end-to-end service.  What if a client is very slow at consuming data from your system?  It could tie up a thread in the gateway taking it out of action.  Now you have less threads working the queue so the response time will be increasing.  Queues and threads need to be monitored, and appropriate action needs to be taken when thresholds are crossed.  For example, when a queue is 70% full, maybe an alert should be raised so an investigation can take place?  Also, transaction times need to be sampled to ensure they are in the expected range.

### Summary
If we do not consider how our systems will behave when under heavy load then they will most likely seriously degrade at best, and at worst crash.  When they crash this way, we get to find out if there are any really evil data corruption bugs lurking in those dark places.  Applying back pressure is one effective technique for coping with sustained high-load, such that maximum throughput can be delivered without degrading system performance for the already accepted requests and transactions.


## Disadvantage(s): asynchronism
1. Use cases such as inexpensive calculations and realtime workflows might be better suited for synchronous operations, as introducing queues can add delays and complexity.

## Pattern 1
1. Do the time-consuming work in advance and serving the finished work with a low request time.
1. Very often this paradigm is used to turn dynamic content into static content.  
   * Pages of a website, maybe built with a massive framework or CMS, are pre-rendered and locally stored as static HTML files on every change. 
   * Often these computing tasks are done on a regular basis, maybe by a script which is called every hour by a cronjob.


## Pattern 2
1. Handle tasks asynchronously using a message broker

### Message Brokers
#### RabbitMQ

