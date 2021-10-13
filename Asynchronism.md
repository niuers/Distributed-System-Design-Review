# Asynchronism
## Pattern 1
1. Do the time-consuming work in advance and serving the finished work with a low request time.
1. Very often this paradigm is used to turn dynamic content into static content.  
   * Pages of a website, maybe built with a massive framework or CMS, are pre-rendered and locally stored as static HTML files on every change. 
   * Often these computing tasks are done on a regular basis, maybe by a script which is called every hour by a cronjob.


## Pattern 2
1. Handle tasks asynchronously using a message broker

### Message Brokers
#### RabbitMQ

