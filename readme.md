# Nodejs

- In software we perform two types of Operation
  - Non-blocking operations
  - Blocking operations
- Non-blocking operations:
  - Non-blocking refers to code that doesn't block execution.
  - any operations which are non-blocking in nature are performed in L1, L2 and L3 cache
  - bacically in RAM
  - number of cycles requires are very less
  - example : mathematical calculations
- Blocking operations:
  - number of cycles requires are millions
  - ex: data-base operations, file i/o operations, network operations
  - js engine is single threaded i.e- one execution stack in js

# How Nodejs Works

- In Nodejs we have event loop
- whenever we send request to nodejs these are rquest will be queued under Event Queues
- Event loop constantly monitor the event queue and also place each events/operations from event queue to execution stack
- If operation that was requested is non-blocking operation, then operation will be processed to execution stack and response will send back to the caller
- If operation that was requested is blocking operation, then event loop will handle this operation to worker threads, later onces the operation is completed in worker thread, then it will be queued back to execution stack where all the callback functions will be processed, Response will be send back to the caller
- Thus nodejs can easily handle blocking and non-blocking operations using its event-loop, event-queue and execution stack
- But why should we think of Nodejs cluster, bcoz- nodejs only uses single thread, even-though we hardware of multi-core machine, nodejs does not take full advantage of all the cores, As nodejs will only use one core.

---

# Nodejs Cluster Module

- A single instance of nodejs runs in a single thread
- Cluster module allows to take advatage of multi-core system
- It helps to create child processes which shares server ports and distribute incoming loads/connections

# Working of Cluster Module

- Worker processes are spawned using the fork() method
- These processes can communicate with parent via IPC (Inter process communication channel)
- They can also communicate from parent to child and child to parent
- The cluster module supports two methods for distributing incoming connections:
  - Round Robin
    - This is default on all platform except windows
    - The master process listens on a PORT, accepts a new connection and distribute them accross the workers in a round-robin fashin
  - Second Method
    - The master process creates the listen socket and sends it to interested workers
    - The workers then accept incoming connections directly

## Cluster Modules: Methods, Properties and Events

- There are several events, methods and props which we can be handled during Nodejs clustering:

- EVENTS

  - Online : (when new worker is forked by master then this worker will response with online, when master recieves online message then master will emit this event)
  - Fork : (when a new worker is forked, the cluster module will emit fork event, this can be used to log the worker activity and create custom time-out )
  - Message : (when cluster master recieves a message from any woker this event is emitted)
  - Disconnect : (If worker IPC channel is disconnected then this event is emitted, It can happen if worker exit gracefully (or) killed (or) diconnect manually)
  - Exit : (If any worker dies this event is emitted)

- METHODS

  - Fork() : [This method is used to spawn a new worker process, this method can only be called from master process]
  - Diconnect(): [call this method on diconnection of each worker process, if all the worker process are diconnected, all internal handles will be closed, which allow master process to die gracefully]

- PROPERTIES

  - IsMaster : [if process is master proces then its value is true, it is determined by nodeUniqueId <--- find in process.env ]
  - IsWorker: [if process is worker proces then its value is true]

- NOTE: Cluster Module also provides: Worker Class which also have props, methods and events

## WORKER Class: Methods, Properties and Events

- There are several events which we can handle provided by worker class

- EVENTS

  - Online
  - Message
  - Disconnect
  - Exit

- METHODS

  - Diconnect() : [it will close all servers, diconnect IPC channel, thus worker will nolonger accept new connection]
  - IsConnected() : [if worker is connected to its master then return true via its IPC channel]
  - IsDead() : [if worker process is terminated then return true]
  - Kill() : [this will kill the worker]
  - Send() : [it will send message to master process]

- PROPERTIES

  - Id : [when a new worker is created, it is assgined with new unique id, this id value will be stored in this prop]
  - Process : [all workers are created using child_process.fork() -> this method return object which is stored in this prop]

## How to get CPU count ?

- To know how many logical CPU are avaliable in our machine, we can make use of OS module
- os.cpus()
  - Returns an array of objects containing information about each logical CPU cores
- os.cpus().length
  - Return the count of logical CPUs

---

# Nodejs Cluster Load Test

- Use tools to test nodejs cluster:
  - Load Testing using 'loadtest' package
    - To install nodejs cluster, install 'loadtest' package using npm install command
    - sudo npm i -g loadtest
    - https://www.npmjs.com/package/loadtest
    - Set max number of req : -n
    - Concurrent requests : -c
    - number of requests per second : --rps
    - command : loadtest -n 1000 -c 100 --rps 200 http://localhost:3000?number=10
  - Load Testing using 'artillery' package
    - To load test Nodejs cluster, install 'artillery' package using command
    - sudo npm install -g artillery --allow-root --unsafe-perm=true
    - https://www.npmjs.com/package/artillery
    - used for ad-hoc testing : quick
    - used for creating virtual users : --count
    - number of request per virtual user : -n
    - command : artillery quick --count 10 -n 20 http://localhost:3000?number=20
    - DOC: https://artillery.io/docs/getting-started/
- Usage of loadtest:
  - terminal1 : nodemon nodejs-cluster/server.js
  - terminal2 : loadtest -n 1000 -c 100 --rps 200 http://localhost:3000?number=10
- Usage of artillery:
  - \$ artillery -V
  - terminal1 : nodemon nodejs-cluster/server.js
  - terminal2 : artillery quick --count 10 -n 20 http://localhost:3000?number=20

---

# PM2

- PM2 --> used to run nodejs application in cluster
- Reason why do we need PM2
  - If we create/fork worker process, in production env these child process may get kill due to many reasons, thus starting this kill process with no-down time can be tricky
  - Restart one or more child process can be tricky
  - How do we keep our nodejs application alive forever in production env ?
- What is PM2
  - PM2 is a production process manager for Nodejs applications with a built-in load-balancer
  - PM2 allows to :
    - Keep applications alive forever
    - Reload them without downtime
    - Facilitate common system admin tasks like:
      - Monitoring, stopping, deleting processes etc
- PM2 Working
  - PM2 makes usage of Nodejs cluster module
  - The scaled application's child processes can automatically share server ports
  - You can start Nodejs application using PM2 by using
    - Command line options
    - Process configuration file [js/yaml/json file]
- Install PM2 using NPM
  - https://www.npmjs.com/package/pm2
  - sudo npm i -g pm2
- Creating Process file
  - You can create a process file to:
    - Fine-tune the behavior fo applications
    - Options
    - Environment variables
    - Log files of each application
  - To create a process file use command:
    - pm2 ecosystem
  - After creating a process file, use command:
    - pm2 [start|restart|stop|delete] ecosystem.config.js
- PM2 Commands
  - PM2 start command
    - pm2 start <process_config_file>
    - pm2 start ecosystem.config.js
  - To monitor the processes, use
    - pm2 monit
  - To stop any/all processes use
    - pm2 stop all/<app_name>
  - To delete any/all processes use
    - pm2 delete all/<app_name>
  - To list all the applications use
    - pm2 list
