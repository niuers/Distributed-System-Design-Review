
# Timelines
1. How to send tweets to your follower? 
2. It's unacceptable to have any user to get message in more than 5 seconds.


## Pull vs. Push
### Pull
1. Targeted
   * Twitter.com
   * home_timeline API
1. Queried
   * Search API

### Push
1. Targeted
   * User/Site Streams
   * Mobile PUSH (SMS, etc.)
1. Queried
   * Track/Follow Streams
1. Can use async DB writes
2. 
### Fan-Out
1. The biggest problem for twitter when fan-out (write) for famous person
   * In transaction processing systems, we use it to describe the number of requests to other services that we need to make in order to serve one incoming request.
   * It causes race condition
1. 400 million tweets per day, 4600 tweets per second, 7000/sec peak time, 12K/sec in large events
