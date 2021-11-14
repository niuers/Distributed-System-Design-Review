1. In large application, code change can't happen instanenously, that means old and new versions of the code, and old and new data formats, may potentially all coexist in the system at the same time. 
2. We need maintain compatibility in both directions
   * Backward compatibility: newer code can read data that was written by older code
   * Forward compatibility: older code can read data that was written by newer code
1. The details of these encodings affect not onlytheir  efficiency,  but  more  importantly  also  the  architecture  of  applications  and  your options for deploying them
2. In particular, many services need to support rolling upgrades, where a new version ofa  service  is  gradually  deployed  to  a  few  nodes  at  a  time,  rather  than  deploying  to  allnodes simultaneously. These properties are hugely benefi‐cial for evolvability, the ease of making changes to an application.
3. During rolling upgrades, or for various other reasons, we must assume that differentnodes are running the different versions of our application’s code. It is impor‐tant that all data flowing around the system is encoded in a way that provides back‐ward compatibility (new code can read old data) and forward compatibility (old codecan read new data).

# Formats for Encoding Data
1. Programs usually work with data in (at least) two different representations:
   * In memory: objects, structs, lists, hash tables,t rees etc.
   * When you want to write data to file or send it over the network, you have to encode it as some kind of self-contained sequence of bytes (e.g. JSON)
1. We need translation between the two representations.

## Language-Specific Formats
1. Java: java.io.Serializable/Python: pickle
2. Limitations
   * Encoding is tied to a particular language, and read the data in another language is very difficult
   * The decoding process needs to be able to instantiate arbitrary classes. A frequently source of security problems. 
   * Versioning data is not easy
   * Efficiency is not that good.
1. For these reasons it’s generally a bad idea to use your language’s built-in encoding for anything other than very transient purposes.
2. They often fail to provide forward and backward compatibility

## JSON, XML, and Binary Variants

### Textual Formats
1. XML: too verbose, unnecessarily complicated
2. JSON: built-in support in web browsers (a subset of javascript) and simplicity relative to XML
3. CSV: less power, still popular

#### Limitations of Textual Formats
1. A lot of ambiguity around the encoding of numbers. 
   * XML/CSV can't distinguish between a number and a string (except by an exteranl schema)
   * JSON doesn't distinguish integers and floating-point numbers, no precision
      * A problem with large numbers, integers larger than 2^53 can't be exactly represented in double-precision floating-point number.
2. JSON/XML support Unicode character strings but don't support binary strings (sequences of bytes without a character encoding) 
1. There's only optial schema support for XML/JSON.
3. CSV doesn't have any schema. 
4. their compatibility depends on how you use them. 

### Binary Encoding
1. Binary encodings for JSON, XML. 
   * since they don’t prescribe a schema, they need to include all the object field names within the encoded data. That is, in a binary encoding of the JSON document, they will need to include the strings userName, favoriteNumber, and interests somewhere.

#### Apache Thrift and Protocol Buffers (protobuf)
1. Both Thrift and Protocol Buffers require a schema for any data that is encoded.
2. there are no field names needed in their encoding. Instead, the encoded data contains field tags, which are numbers (1, 2, and 3)

##### Field Tags and Schema Evolution
1. an encoded record is just the concatenation of its encoded fields. Each field is identified by its tag number (the numbers 1, 2, 3 in the sample schemas) and annotated with a datatype.
2. You can add new fields to the schema, provided that you give each field a new tag number. If old code (which doesn’t know about the new tag numbers you added) tries to read data written by new code, including a new field with a tag number it doesn’t recognize, it can simply ignore that field. 
3. The datatype annotation allows the parser to determine how many bytes it needs to skip. This maintains forward com‐ patibility: old code can read records that were written by new code.
4. As long as each field has a unique tag number, new code can always read old data, because the tag numbers still have the same meaning. The only detail is that if you add a new field, you cannot make it required. If you were to add a field and make it required, that check would fail if new code read data written by old code, because the old code will not have written the new field that you added.
5. Similarly to removing a field.

##### Datatypes and schema evolution
1. Changing the datatype of field can be tricky: 

#### Apache AVRO
1. A subproject of Hadoop
2. that there are no tag numbers in the schema.
3. there is nothing to identify fields or their datatypes. The encoding simply consists of values concatenated together. A
4. To parse the binary data, you go through the fields in the order that they appear in the schema and use the schema to tell you the datatype of each field. This means that the binary data can only be decoded correctly if the code reading the data is using the exact same schema as the code that wrote the data. Any mismatch in the schema between the reader and the writer would mean incorrectly decoded data.
5. We use Avro to encode/decode VCV data for GRM

##### The writer’s schema and the reader’s schema
1. The key idea with Avro is that the writer’s schema and the reader’s schema don’t have to  be  the  same—they  only  need  to  be  compatible.
2. Avro  library  resolves  the  differences  by  looking  at  the  writer’s  schema  and  thereader’s  schema  side  by  side  and  translating  the  data  from  the  writer’s  schema  intothe  reader’s  schema. 
##### Schema evolution rules
1. To  maintain  compatibility,  you  may  only  add  or  remove  a  field  that  has  a  defaultvalue. 
2. There  is  an  important  question  that  we’ve  glossed  over  so  far:  how  does  the  readerknow  the  writer’s  schema  with  which  a  particular  piece  of  data  was  encoded?
   * Large file with lots of records
      * A  common  use  for  Avro—especially  in  the  context  of  Hadoop—is  for  storing  alarge file containing millions of records, all encoded with the same schema. the  writer  of  thatfile  can  just  include  the  writer’s  schema  once  at  the  beginning  of  the  file.  Avrospecifies a file format (object container files) to do this.
   * Database with individually written records
      * The simplest solution is to include a version number at the begin‐ning of every encoded record, and to keep a list of schema versions in your database
   * Sending records over a network connection
      * When  two  processes  are  communicating  over  a  bidirectional  network  connec‐tion,  they  can  negotiate  the  schema  version  on  connection  setup  and  then  usethat  schema  for  the  lifetime  of  the  connection.  The  Avro  RPC  protocol .

##### Dynamically generated schemas
1. One  advantage  of  Avro’s  approach,  compared  to  Protocol  Buffers  and  Thrift,  is  thatthe  schema  doesn’t  contain  any  tag  numbers. So Avro is friendlier to dynamically generated schemas
2. For example, if you dump data from DB, Avro deals with the case when DB schema changes, while Thrift/Proto don't.
##### Code generation and dynamically typed languages
1. Thrift and Protocol Buffers rely on code generation: after a schema has been defined,you  can  generate  code  that  implements  this  schema  in  a  programming  language  ofyour  choice.  This  is  useful  in  statically  typed  languages. because  it  allows  efficient  in-memory  structures  to  be  used  for  decoded  data,  and  itallows type checking and autocompletion in IDEs when writing programs that accessthe data structures
2. There's not much point in dynamically typed programming languages. 
3. Avro provides optional code generation for statically typed programming languages,but it can be used just as well without any code generation.

### The Merits of Schemas
1. They can be much more compact than the various “binary JSON” variants, since they can omit field names from the encoded data.
2. The  schema  is  a  valuable  form  of  documentation,  and  because  the  schema  is required  for  decoding,  you  can  be  sure  that  it  is  up  to  date  (whereas  manually maintained documentation may easily diverge from reality).
3. Keeping a database of schemas allows you to check forward and backward com‐patibility of schema changes, before anything is deployed.
4. For users of statically typed programming languages, the ability to generate codefrom the schema is useful, since it enables type checking at compile time

## Modes of Dataflow
1. some of the most common ways how data flows between processes:
   * Via databases
   * Via service calls
   * Via asynchronous message passing


### Dataflow Through Databases
1. Preservation  of  unknown fields:  There is an additional snag. Say you add a field to a record schema, and thenewer  code  writes  a  value  for  that  new  field  to  the  database.  Subsequently,  an  olderversion  of  the  code  (which  doesn’t  yet  know  about  the  new  field)  reads  the  record,updates  it,  and  writes  it  back.  In  this  situation,  the  desirable  behavior  is  usually  forthe old code to keep the new field intact, even though it couldn’t be interpreted
2. Different values written at different times
   * data outlives code: When  you  deploy  a  new  version  of  your  application ,  you  may  entirely  replace  the  old  version  with  the  new  version  within  a  few minutes. The same is not true of database contents: the five-year-old data will still be there, in the original encoding, unless you have explicitly rewritten it since then.
   * Schema evolution thus allows the entire database to appear as if it was encoded with asingle schema, even though the underlying storage may contain records encoded withvarious historical versions of the schema.
1. Archival storage
   * As the data dump is written in one go and is thereafter immutable, formats like Avroobject container files are a good fit. This is also a good opportunity to encode the datain an analytics-friendly column-oriented format such as Parquet


### Dataflow Through Services: REST and RPC
1. A  key  design  goal  of  a  service-oriented/microservices  architecture  is  to  make  the application easier to change and maintain by making services independently deployable and evolvable. For example, each service should be owned by one team, and that team should be able to release new versions of the service frequently, without having to  coordinate  with  other  teams.  In  other  words,  we  should  expect  old  and  new  versions of servers and clients to be running at the same time, and so the data encodingused  by  servers  and  clients  must  be  compatible  across  versions  of  the  service  API
#### Web Services
1. When HTTP is used as the underlying protocol for talking to the service, it is called aweb service.
   * There are two popular approaches to web services: RESTand SOAP. They are almostdiametrically opposed in terms of philosophy
1. REST is not a protocol, but rather a design philosophy that builds upon the principlesof  HTTP  [34,  35].  It  emphasizes  simple  data  formats,  using  URLs  for  identifyingresources  and  using  HTTP  features  for  cache  control,  authentication,  and  contenttype  negotiation. 
2. By  contrast,  SOAP  is  an  XML-based  protocol  for  making  network  API  requests.viiAlthough  it  is  most  commonly  used  over  HTTP,  it  aims  to  be  independent  fromHTTP and avoids using most HTTP features.
3. The API of a SOAP web service is described using an XML-based language called theWeb  Services  Description  Language,  or  WSDL.  WSDL  enables  code  generation  sothat  a  client  can  access  a  remote  service  using  local  classes  and  method  calls  (whichare encoded to XML messages and decoded again by the framework). This is useful instatically  typed  programming  languages,  but  less  so  in  dynamically  typed  ones

#### The problems with remote procedure calls (RPCs)
1. The RPC model tries to make a request to a remote net‐work service look the same as calling a function or method in your programming lan‐guage,  within  the  same  process  (this  abstraction  is  called  location  transparency).Although  RPC  seems  convenient  at  first,  the  approach  is  fundamentally  flawed 
2. A local function call is predictable and either succeeds or fails, depending only onparameters that are under your control. A network request is unpredictable
3. A  local  function  call  either  returns  a  result,  or  throws  an  exception,  or  neverreturns  (because  it  goes  into  an  infinite  loop  or  the  process  crashes).  A  networkrequest  has  another  possible  outcome:  it  may  return  without  a  result,  due  to  atimeout.
4. If  you  retry  a  failed  network  request,  it  could  happen  that  the  requests  areactually  getting  through,  and  only  the  responses  are  getting  lost.  In  that  case,retrying will cause the action to be performed multiple times, unless you build amechanism  for  deduplication  (idempotence)  into  the  protocol.
5. Every time you call a local function, it normally takes about the same time to exe‐cute.  A  network  request  is  much  slower  than  a  function  call,  and  its  latency  isalso wildly variable
6. When you call a local function, you can efficiently pass it references (pointers) toobjects in local memory. When you make a network request, all those parameters need  to  be  encoded  into  a  sequence  of  bytes  that  can  be  sent  over  the  network.That’s okay if the parameters are primitives like numbers or strings, but quicklybecomes problematic with larger objects.
7. The  client  and  the  service  may  be  implemented  in  different  programming  lan‐guages,  so  the  RPC  framework  must  translate  datatypes  from  one  language  intoanother.  This  can  end  up  ugly,  since  not  all  languages  have  the  same  types—recall  JavaScript’s  problems  with  numbers  greater  than  2^53

##### Current directions for RPC
1. Custom  RPC  protocols  with  a  binary  encoding  format  can  achieve  better  perfor‐mance  than  something  generic  like  JSON  over  REST.  
2. However,  a  RESTful  API  hasother  significant  advantages:  
   * it  is  good  for  experimentation  and  debugging  (you  can simply  make  requests  to  it  using  a  web  browser  or  the  command-line  tool  curl,without  any  code  generation  or  software  installation),  
   * it  is  supported  by  all  main‐stream programming languages and platforms, 
   * and there is a vast ecosystem of tools available  (servers,  caches,  load  balancers,  proxies,  firewalls,  monitoring,  debuggingtools, testing tools, etc.)
1. The mainfocus of RPC frameworks is on requests between services owned by the same organi‐zation, typically within the same datacenter

##### Data encoding and evolution for RPC
1. For  evolvability,  it  is  important  that  RPC  clients  and  servers  can  be  changed  anddeployed  independently.  
2. It  is  reasonable  to  assume  that  all  the  servers  will  be  updated  first,and  all  the  clients  second. Thus,  you  only  need  backward  compatibility  on  requests,and forward compatibility on responses.
3. The backward and forward compatibility properties of an RPC scheme are inheritedfrom whatever encoding it uses
   * Thrift, gRPC (Protocol Buffers), and Avro RPC can be evolved according to thecompatibility rules of the respective encoding format.
   * RESTful  APIs  most  commonly  use  JSON  (without  a  formally  specified  schema)for  responses,  and  JSON  or  URI-encoded/form-encoded  request  parameters  forrequests. 

### Message-Passing Dataflow
1.  Asynchronous  message-passing  systems,which are somewhere between RPC and databases. 
2.  They are similar to RPC in that aclient’s  request  (usually  called  a  message)  is  delivered  to  another  process  with  low latency. They are similar to databases in that the message is not sent via a direct network connection, but goes via an intermediary called a message broker (also called a message queue or message-oriented middleware), which stores the message temporarily
3. Using a message broker has several advantages compared to direct RPC:
   * It  can  act  as  a  buffer  if  the  recipient  is  unavailable  or  overloaded,  and  thus improve system reliability.
   * It  can  automatically  redeliver  messages  to  a  process  that  has  crashed,  and  thus prevent messages from being lost.
   * It  avoids  the  sender  needing  to  know  the  IP  address  and  port  number  of  the recipient  (which  is  particularly  useful  in  a  cloud  deployment  where  virtual machines often come and go).
   * It allows one message to be sent to several recipients.
   * It  logically  decouples  the  sender  from  the  recipient  (the  sender  just  publishes messages and doesn’t care who consumes them). 
1. However,  a  difference  compared  to  RPC  is  that  message-passing  communication  is usually one-way: a sender normally doesn’t expect to receive a reply to its messages. It is possible for a process to send a response, but this would usually be done on a separate  channel.  This  communication  pattern  is  asynchronous:  the  sender  doesn’t  wait for the message to be delivered, but simply sends it and then forgets about it.

#### Message brokers
1. Products: RabbitMQ,  ActiveMQ,  Hor‐netQ,  NATS,  and  Apache  Kafka  have  become  popular.
2. message brokers are used as follows: one process sends a message to a named queue  or  topic,  and  the  broker  ensures  that  the  message  is  delivered  to  one  or  more consumers of or subscribers to that queue or topic. There can be many producers and many consumers on the same topic.
3. If  a  consumer  republishes  messages  to  another  topic,  you  may  need  to  be  careful  topreserve  unknown  fields,  to  prevent  the  issue  described  previously  in  the  context  ofdatabases
##### Distributed actor frameworks
1. The actor model is a programming model for concurrency in a single process. Rather than  dealing  directly  with  threads  (and  the  associated  problems  of  race  conditions, locking, and deadlock), logic is encapsulated in actors. Each actor typically representsone client or entity, it may have some local state (which is not shared with any otheractor),  and  it  communicates  with  other  actors  by  sending  and  receiving  asynchronous  messages.  Message  delivery  is  not  guaranteed:  in  certain  error  scenarios,  messages  will  be  lost.  Since  each  actor  processes  only  one  message  at  a  time,  it  doesn’t need  to  worry  about  threads,  and  each  actor  can  be  scheduled  independently  by  the framework.
2. In distributed actor frameworks, this programming model is used to scale an application across multiple nodes. The same message-passing mechanism is used, no matter whether the sender and recipient are on the same node or different nodes. If they are on  different  nodes,  the  message  is  transparently  encoded  into  a  byte  sequence,  sent over the network, and decoded on the other side. 
3. Location transparency works better in the actor model than in RPC, because the actor model  already  assumes  that  messages  may  be  lost,  even  within  a  single  process. Although  latency  over  the  network  is  likely  higher  than  within  the  same  process, there  is  less  of  a  fundamental  mismatch  between  local  and  remote  communication when using the actor model.
4. A  distributed  actor  framework  essentially  integrates  a  message  broker  and  the  actor programming model into a single framework. However, if you want to perform rolling  upgrades  of  your  actor-based  application,  you  still  have  to  worry  about  forward and  backward  compatibility,  as  messages  may  be  sent  from  a  node  running  the  newver sion to a node running the old version, and vice versa. Three popular distributed actor frameworks handle message encoding as follows:
   * Akka uses Java’s built-in serialization by default, which does not provide forward or backward compatibility. However, you can replace it with something like Protocol Buffers, and thus gain the ability to do rolling upgrades.
   * Orleans  by  default  uses  a  custom  data  encoding  format  that  does  not  support rolling  upgrade  deployments;  to  deploy  a  new  version  of  your  application,  you need to set up a new cluster, move traffic from the old cluster to the new one, and shut down the old one. Like with Akka, custom serialization plugins canbe used.
   * In Erlang OTP it is surprisingly hard to make changes to record schemas (despitethe system having many features designed for high availability); rolling upgrades are  possible  but  need  to  be  planned  carefully.


