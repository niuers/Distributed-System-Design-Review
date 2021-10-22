# FB Live-Scaling to Billion Users

## Resources
1. Compute:
   * Frame decode/encode, analysis
2. Memory: encoding/decoding streams
3. Storage
4. Network: upload and playback
## Scaling challenges
### Number of  Concurrent Unique Streams
1. There are patterns: peak in daytime, valley in night
1. Factors Affecting Scalability
   * Ingest Protocol to work with different kinds of networks: celluar, wifi
   * Network Capacity
   * Server side encoding resources

### Number of total viewers of all streams
1. It also has some pattern
1. Delivery Protocols
2. Network Capacity:
   * Use CDN
4. Effective Caching

### Maximum number of views of single stream
1. Unpredictable
2. Caching
3. Stream Distribution

### Four Challenges for FB Live
1. Populating cache ahead of time is tricky (unlike premimum contents)
2. Predicting number of viewers for any stream is hard (not scheduled live event)
3. Planning for live events and scaling resources ahead of time is problematic
4. Predicting concurrent stream/viewer spikes caused by world event is difficult

## Protocols and Codecs
1. Time to Production
2. network Compatibility
3. End-to-End latency: sub 30 seconds, ideally in single digit seconds
   * You can't have too many buffers in your pipeline because each buffer adds a few seconds of latency
5. Application Size: it's part of FB app, so < 500KB

## Stream Ingest and Processing
1. N.B. If we use source IP as key for consistent hashing, we may endup mapping a lot of streams to the same machine, so we use  a stream ID.

## Reliability Challenges
1. Network problem in ingestion side: 
2. Thundering Herd: many requests for a popular stream.
   * Use cache blocking timeout to prevent multiple same requests goto DC, this works when response come back, but if the response doesn't come back. Everyone goes to DC, this is thundering herd.
   * Solution: Tuning cache blocking timeout
