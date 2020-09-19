# Web Application & Software Architecture 101

## Different Tires in Software Archtiecture

***Trie***: Compoents of Application for different purpose

+ Caching
+ Datatbase server
+ Backend Application server
+ Messaging
+ User Interface

|         |            Client (Access)            |          server              |
|:--------|:-------------------------------------:|:----------------------------:|
|  Single | UI, backend business logic, datatbase |                              |
|   Two   |         UI, bussinsee logic           |   backend (with datatbase)   |
|  Three  |                UI(browser)            |     backeend, database       |
|    N    |                UI(browser)            | baching, backend, messaging  |

+ Single: traditional standalone desktop application
  + No network latency
  + Easily reverse engineer, perforce depend on client machine, UI style might depend on client machine

+ Two: Online Game (WoW), ToDo list app
  + Heavy bussiness logic do on client side, mimimize call to backend
  + First time actiavtion need to download data from backend, some part business logic still on client side

+ Three: Simple web service, usually client will access by browser
  + User can only access UI (html, javascript) execute on browser, expect same experience for all service
  + UI, backend, database all on different dedicated server

+ N: Large scale Web service (modern)
  + Single Responsiblity Principle: just do your own job. Bussiness logic server handle business logic, storage server handle data store request.
  + Separation of Concerns: Stop worry about others, just concern your self. Bussiness logic server does not need to know database is shareed by other application or not, or how many queue exist in messaging server.

Notice: from ***single responsibility principle*** perspective, we should not use database **stored proceduer**. If we want to change to different database or refactor and move **stored procedure** logic into somewhere. This break the ***single responsibility principle***.

+ Tire vs Layer
  + Layer is more describe in code layer organization. For example, in our bussines logic tries, we have data layer to provide database access service, bussiness layer to process bussines service logic and view layer decide how to present view.