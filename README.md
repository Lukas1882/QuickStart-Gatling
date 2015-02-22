# Quick Start for Gatling, the capable load testing tool
#### Yaolong Li - [Huffington Post](http://www.huffingtonpost.com/)
![](http://blog.groupeafg.com/wp-content/uploads/2013/02/gatling-logo-630x200.png)

This is brief guide for people need a quick start in Gatling. For more details please visit [Gatling documentation](http://gatling.io/#/docs).

## 1.What is Gatling
****[Gatling ](http://gatling.io/#/)****is a is an open source load testing framework based on Scala, Akka and Netty.
It can be used as a load testing tool for analyzing and measuring the performance of a variety of services, with a focus on web applications.

Gatling officially supports the following protocols: HTTP, WebSockets, Server Sent Events and JMS. Simulations are code, written thanks to a user-friendly DSL. This allows users to add custom behavior through many hooks.

* High performance: get more virtual users.
* Ready-to-present HTTP reports.
* Scenario recorder and developer-friendly DSL.

**To user Gatling, you needn't to have good Scala knowledges, although the scenario is written in Scala.**
## 2. Setup Gatling tools
Firstly, download [Gatling bundle(zip)](http://goo.gl/o14jQg) from [Gatling download page](http://gatling.io/#/download) and unpack the Gatling folder to a folder of your choice. Gatling also supports Maven and SBT.

To run Gatling, you need have Java 7+ or later in you machine. In my macbook, it is java 7. Open your terminal and cd to the Gatling folder, then go to the folder "bin". Run the following command to start Gatling.
```
sh recorder.sh
```
If everything is right, you have opened you Gatling Recorder GUI window gracefully.
![](http://gatling.io/docs/2.1.4/_images/recorder1.png)

Now, we need to setup our Firefox browser. Open "preference", go to the "advanced" subpage, then go to the "network" tag. Update the connection settings as shown in the picture.

![](http://gatling.io/docs/2.0.0-RC2/_images/recorder-browser_advanced_settings.png)

## 3. Run the Gatling Recorder
Now, we are ready to get some records by Gatling.

In the Gatling Recorder GUI window, we can change the class name and output folder. In this example, I set class name as "MyTest", output folder as "~/mytest". 

* 1.Click the "start button" in the Gatling Recorder GUI window to start the Gatling monitor.
* 2.Open Firefox and go to the page "http://computer-database.herokuapp.com/computers", this is a simple web application demo. We choose it because it has no extra css, JQuery and other web requests, which can simplify our test.
  *  **If you did not start Gatling first, your Firefox will be blocked.**
  *  **Change back the Firefox setting after playing with Gatling to make sure you can go to your Facebook in Firefox~**

* 3.In this web application, search "mac", then click the "imac" from the search result. In the next new page,  click "save this computer".
* 4.Go back to the Gatling window and "save & stop".

## 3.Gatling Scenario
Go to your Gatling folder and you will find a new Scala file called **"MyTest"** created in the folder **"~/user-files/mytest"**. This is the Gatling scenario creaded by Gatling while you did the operations in that web application.

``` scala
import scala.concurrent.duration._

import io.gatling.core.Predef._
import io.gatling.http.Predef._
import io.gatling.jdbc.Predef._

class MyTest extends Simulation {

	val httpProtocol = http
		.baseURL("http://computer-database.herokuapp.com")
		.inferHtmlResources()
		.acceptHeader("text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8")
		.acceptEncodingHeader("gzip, deflate")
		.acceptLanguageHeader("en-US,en;q=0.5")
		.connection("keep-alive")
		.contentTypeHeader("application/x-www-form-urlencoded")
		.userAgentHeader("Mozilla/5.0 (Macintosh; Intel Mac OS X 10.10; rv:35.0) Gecko/20100101 Firefox/35.0")

	val headers_1 = Map("Accept" -> "image/png,image/*;q=0.8,*/*;q=0.5")

    val uri1 = "http://computer-database.herokuapp.com"

	val scn = scenario("MyTest")
		.exec(http("request_0")
			.get("/computers")
			.resources(http("request_1")
			.get(uri1 + "/favicon.ico")
			.headers(headers_1)
			.check(status.is(404)),
            http("request_2")
			.get(uri1 + "/favicon.ico")
			.check(status.is(404))))
		.pause(5)
		.exec(http("request_3")
			.get("/computers?f=mac"))
		.pause(6)
		.exec(http("request_4")
			.get("/computers/25"))
		.pause(10)
		.exec(http("request_5")
			.post("/computers/25")
			.formParam("name", "iMac")
			.formParam("introduced", "1998-01-01")
			.formParam("discontinued", "")
			.formParam("company", "1"))

	setUp(scn.inject(atOnceUsers(1))).protocols(httpProtocol)
}
```
This senario describs what you have done while the Gatling monitored the Firefox. Let's read these codes together.
* 1.The class declaration, extending ```Simulation```.
 ```
class MyTest extends Simulation
```
* 2.The ```baseURL``` that will be prepended to all relative urls.
```
.baseURL("http://computer-database.herokuapp.com")
```
* 3.Common HTTP headers that will be sent with all the requests.
```
.acceptHeader("text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8")
```
* 4.```Scenario``` is the way to bootstrap a new scenario. You can use any character in the name of the scenario except tabulations.

 In this test, we only have one scenario. Multy-scenario is supported in Gatling.
```
val scn = scenario("MyTest")
```
* 5.The method ```exec``` is used to execute an action. Actions are usually requests (HTTP, LDAP, POP, IMAP, etc) that will be sent during the simulation. Any action that will be executed will be called with exec.

  Following are the request 0 and request 3 in this example.
```
scenario("MyTest")
     .exec(http("request_0")
         .get("/computers")
```
and
```
.exec(http("request_3")
    .get("/computers?f=mac"))
```
* 6.```resources``` helps the Gatling to get the target web element requests.
```
.resources(http("request_1")
	 .get(uri1 + "/favicon.ico")
	 .headers(headers_1)
```
* 7.```Check``` is used for verifying that the response to a request matches expectations and capturing some elements in it.
Checks are performed on a request with the check method. For example, on an HTTP request:
```
.check(status.is(404)),
```
One can of course perform multiple checks:
```
http("My Request").get("myUrl").check(status.not(404), status.not(500)))
```
* 8.The method ```pause``` acts like the method "sleep" in Java. When a user sees a page he/she often reads what is shown and then chooses to click on another link. To reproduce this behavior, the pause method is used.
```
.pause(5)
```
* 9.HTTP protocol requires 2 mandatory parameters: the method and the URL.
Gatling provides built-ins for the most common methods. Those are simply the method name in minor case:
```
get(url: Expression[String])
put(url: Expression[String])
post(url: Expression[String])
delete(url: Expression[String])
head(url: Expression[String])
patch(url: Expression[String])
options(url: Expression[String])
```
* 10.```Setup``` is where you define the load you want to inject to your server.
You can configure assertions and protocols with these two methods:
 * assertions: set assertions on the simulation, see the dedicated section here
 * protocols: set protocols definitions, see the dedicated section here.

 In this example, we assert only one user one time in http Protocol.
```
setUp(scn.inject(atOnceUsers(1))).protocols(httpProtocol)
```

## 4. Modify the Scenario
We have get the Scala scenario by Gatling monitor above and we need to test our target by our own scenarios. Therefore, we need to modify the created scenario or create new ones.

In Gatling folder "~/user-files/simulations/computerdatabase" and "~/user-files/simulations/computerdatabase/advanced", there are some existing scenarios and we can use them or follow them to make our own.

## 5. Get Test Report
Now it is the most exciting part of Gatling, we need to run Gatling to get HTML report by our scenario.

Firstly, no matter where is your Scala scenario file, please put it into the folder "~/user-files/simulations/". In this folder, you can create your own sub-folder and put your scenario file into it. Gatling will scan the folder "simulations" and get all the Scala files.

Then, type into your terminal with the following command:
```
./gatling.sh
```
Then you will get a list of the scenarios in "simulations" folder. Choose the scenario of your choice by type in the index number. After seconds, a folder of this reports will be created in "~/results". The new folder named like "mytest-1424577449810" stores all the HTML report files and you can copy this folder to your document.

To view the report, you just need to open the "index.html" file in that folder, enjoy it.

