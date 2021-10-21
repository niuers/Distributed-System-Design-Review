We will try to find useful ways of thinking about data systems—not just **how they work**, but also **why they work that way**, and **what questions we need to ask**.

# Key Characteristics of Distributed Systems
## 1. Reliability
### Definition 

The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software **faults**, and even human error). The probability a system will fail in a given period.

1. In simple terms, a distributed system is considered reliable if it keeps delivering its services even when one or several of its software or hardware components fail.
1. **Fault-Tolerant or Resilient Systems**: The systems anticipate (certain types of ) faults and can cope with them.
1. It is impossible to reduce the probability of a fault to zero; therefore it is usually best to design fault-tolerance mechanisms that prevent faults from causing failures. 
2. We generally prefer tolerating faults over preventing faults.

### Examples
1. AMAZON: A purchase shouldn't be cancelled due to the failure of the machine which runs the transaction.

### Faults

#### Software Faults
1. These are systematic errors in the system.
2. Solutions
   * Carefully thinking about assumptions and interactions in the system
   * Thorough testing
   * Process isolation
   * Allowing processes to crash and restart
   * Measuring, monitoring, and analyzing system behavior in production.
#### Hardware Faults
1. We usually think of hardware faults as being random and independent from each other, i.e. weakly correlated.
1. Hard Disks Crash
   * Hard disks *mean time to failure (MTTF)*: 10-50 years.
   * On a storage with 10,000 disks, 1 disk dies per day on average.
3. RAM Faults
4. Power Outage
5. Network Outage
6. Solutions
   * Add redundancy to the individual hardware components to reduce the failure rate of the system. 
   * Multi-machine redundancy (for high availability): Due to large data volumes, more applications begin to use larger number of machines, there's a move toward systems that can *tolerate the loss of entire machines*, by using software fault-tolerance techniques in preference or in addition to hardware redundancy.
      * Rolling Upgrade
      * 
#### Human Error
1. Solutions
   * Design systems in a way that minimizes opportunities for error. Make it easy to do “the right thing” and discourage “the wrong thing.”
   * Decouple the places where people make the most mistakes from the places where they can cause failures.
   * Test thoroughly at all levels, from unit tests to whole-system integration tests and manual tests
   * Allow quick and easy recovery from human errors, to minimize the impact in the case of a failure.
   * Set up detailed and clear monitoring, such as performance metrics and error rates.
   * Implement good management practices and training
### Techniques to Increase Reliability
1. Build Reliable Systems from Unreliable Parts
2. 


## 2. Scalablility
### Definition
The ability of a system to deal with increased load.

1. A system can become unreliable if the load increases a lot.
1. If the system grows in a particular ways, what are our options for coping with the growth? 
2. How can we add computing resources to handle the additional load?


### Describing Load
1. We use **load parameters** to describe load
   * Requests per Second to a web server
   * The ratios of reads to writes in DB
   * The number of  simultaneously active users in a chat room
   * The hit rate on a cache
   * 
## 3. Maintainability
## 4. Availability
## 5. Efficiency

# References
1. Designing Data-Intensive Applications (DDIA), The Big Ideas Behind Reliable, Scalable, and Maintainable Systems, Martin Kleppmann, 2017.
2. Grokking the System Design Interview

# Twitter
## Main Operations
1. post tweet
   * Scaling challenge not primarily due to tweet volume but due to fan-out. 
3. home timeline

# Resources
1. [CS75 (Summer 2012) Lecture 9 Scalability Harvard Web Development David Malan](https://youtu.be/-W9F__D3oY4)
2. Check Bronson's TAO pdf paper in same link as video
3. Check Nishtala's pdf paper in same link as video
4. Lamport's ordering paper: https://lamport.azurewebsites.net/pubs/time-clocks.pdf
5. Paxos (simpler version, not the greek parliament metaphor version): https://www.microsoft.com/en-us/research/publication/paxos-made-simple/
6. Raft: http://thesecretlivesofdata.com/raft/ and also http://kanaka.github.io/raft.js/
7. Dynamo: https://www.allthingsdistributed.com/2007/10/amazons_dynamo.html

