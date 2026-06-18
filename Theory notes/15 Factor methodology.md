#### One codebase one application :  
For each application we should have separate code bases, and if any application or code is shared between them that is also have separate code base. Each application should be maintained in a separate repository
#### Api first
In an application development treat api development as the most important step, and then only we should develop the database and ui and glue all of them together. 
In this way we can mock the ui using dummy apis if the api design is agreed. 

#### Dependency management
We should not expect the host system to have each dependency installed we need to have a package manager like gradle or pom in which we need to specify all the dependencies we need and it should download and work automatically.

#### Design, build release, run
Strictly separate these stages . if anything happens in the run stage like a bug we should go back to the design stage and fix the bug and release it and run it.

#### Configuration, Credentials & code
Store all the configuration and credentials in the environment not in the code . 
Anything that varies between "Dev" and "Production" (database passwords, API keys for Gemini, secret tokens) must be kept separate from the code.
This will prevent the password from leaking in to git hub and to other people

#### Logs
An app shouldn't worry about storing the logs it just write to the console and a separate application then collect those logs and stores them.

#### Disposability
The ultimate goal of disposability is that your app should be so resilient that it doesn't matter if it's a planned shutdown or a sudden hardware failure

**. Fast Startup (Getting "Ready" in Seconds)**  
The goal is to minimize the time between you hitting "Run" and the app being ready to take requests.  

- **Why?** If your biker app suddenly gets a spike in traffic (e.g., a massive group ride starts in Kochi), you need to be able to spin up 10 more instances of your server instantly. If they take 5 minutes to boot, the app will crash before the new servers are ready.  
    

**2. Graceful Shutdown (Leaving Cleanly)**  
This is the part most people overlook. When you stop a process, it shouldn't just "die."  

- **For Web Processes:** It should stop listening for _new_ requests but finish the ones it is currently processing before exiting.  
    
- **For Worker Processes:** If a server is processing a biker’s "Trip Memory" upload and the server is told to shut down, it should return that job to the queue so another server can pick it up.


#### Backing services
Treat backing services such as databases message queues as an attached resource we should be able to change it whenever we want . Do not tightly couple them.

#### Environment parity
Keep development staging and production as similar as possible.

#### Administrative processes
Tasks like database migration etc should run in the same environment as app code using the same configuration.

#### Port binding
The app shouldn't run on an external server or anything it should be self contained it should not depend up on any servers. And if any other application wants to communicate to this should communicate via port which is exposed by the application.


#### Stateless processes
The app should not store any data in it's own in memory anything that needs to be persisted should be stored in the database.

#### Concurrency
Instead of making a massive server run many versions of app side by side

#### Telemetry
Design a real time monitoring system for applications.

#### Authentication and authorisation
Treat everything as a threat. Authenticate and authorise every users.