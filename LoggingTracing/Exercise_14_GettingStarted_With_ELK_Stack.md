Exercise 14: Getting Started with ELK
=====================================

## Learning Goal
The task of this exercise is to get familiar with the log output of Cloud Foundry and learn how you can analyze logs in Kibana as part of the provided ELK stack.

## Prerequisite
Continue with your solution of the last exercise. If this does not work, you can checkout the branch [`origin/solution-13-Use_SLF4J_Features`](https://github.com/ccjavadev/cc-bulletinboard-ads-spring-webmvc/tree/solution-13-Use-SLF4J-Features)

Ensure that you've provided a PostgreSQL service with name `postgres-bulletinboard-ads` on CF [as explained in Exercise 10](/ConnectDatabase/Exercise_10_DeployAdsWithDBServiceOnCF.md).


## Step 1.1: Create CF Logging Service
Create on Cloud Foundry a service instance with name `application-logs`. 

```
cf create-service application-logs lite applogs-bulletinboard
```

Note: You can get the exact names of the available services and its plans in the Service Marketplace (`cf marketplace`). Furthermore note that the created backing service is only *available* within the current targeted space and can be bound only to the applications within the same space.

## Step 1.2: Update manifest.yml
Make sure you specify the service entry in context of your application configuration, i.e. place it with the correct indentation.

```
---
applications:
- name: bulletinboard-ads
  ...
  services:
  - applogs-bulletinboard
```

## Step 1.3: Provide a custom `logback.xml`
**Note that this step is only relevant, when you package your web application as a `war`-file and when using the `sap_java_buildpack`.**
The SAP Java Buildpack bundles amongst others also the SLF4J / logback and the [Java Logging Support for Cloud Foundry](https://github.com/SAP/cf-java-logging-support) logging libraries as documented [here](https://wiki.wdf.sap.corp/wiki/display/xs2java/SAP+Java+Buildack+for+Cloud+Foundry). Furthermore this buildpack provides a central `logback.xml` configuration, firstly to configure the runtime logs and secondly to be (re-)used by the application. **Please note that the configured log level is by default `ERROR`.** 

In order to replace the provided `logback.xml` with yours, you need to provide a custom `logback.xml` as part of our `war`-file in a dedicated `/META-INF` directory as specified [here](https://wiki.wdf.sap.corp/wiki/display/xs2java/Overriding+Resources+in+Droplet). We can do that by using the [Maven WAR Plugin](http://maven.apache.org/plugins/maven-war-plugin/) in order to copy the `logback.xml` resource (`/src/main/resources`) into the specified target directory.

### Specify Maven Plugin
For this open the `pom.xml` using the XML view of Eclipse. Then add `maven-war-plugin` within the `<build><plugins>` section:
```xml
<plugin>
  <!-- Copy logback.xml to overwrite the one provided by the sap_java_buildpack -->
  <!-- see https://wiki.wdf.sap.corp/wiki/display/xs2java/Overriding+Resources+in+Droplet  -->
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-war-plugin</artifactId>
  <version>2.6</version>
  <configuration>
    <failOnMissingWebXml>false</failOnMissingWebXml> <!-- default in version 3.0.0 --> 
    <webResources>
      <resource>
        <directory>src/main/resources</directory>
        <includes>
          <include>logback.xml</include>
        </includes>
        <targetPath>META-INF/sap_java_buildpack/resources/tomcat/conf/</targetPath>
      </resource>
    </webResources>
  </configuration>
</plugin>
```

## Step 1.4: Push your Service and create some log messages on CF

For this exercise you need to deploy the application once more:
- perform a `mvn clean verify` and push the bulletinboard-ads microservice to your trial space (`cf push –n bulletinboard-ads-d012345`). 

Create some log messages on Cloud Foundry via `Postman` by sending
- a single GET-request for an advertisement that **doesn't exist** and
- a single GET-request for an advertisement that **exists**

> Note: During deployment (`cf push`) you may get the following log message `[APP/PROC/WEB/0] ERR SLF4J: Class path contains multiple SLF4J bindings.`. It informs you that your war file comes with another dependency to SLF4J even though it is already provided by the `sap_java_buildpack`. You can ignore this ERROR message for now.

## [Optional] Step 2: Inspect log output of Cloud Foundry
Before analyzing logs with `Kibana`, let's have a look at the log output of Cloud Foundry. Enter 
```
cf logs bulletinboard-ads --recent
```
to analyze the buffered logs. Be aware that the size of the buffer is restricted.

With `cf logs bulletinboard-ads` you can stream the current log output to the command line. Do not stop the command inside the terminal and with `Postman` perform a single GET request for an advertisement that doesn't exist. Please expect some delay until the log messages appear in the command line. Press `Ctrl+C` in your command line to stop streaming the log output.


## Step 3: Login to `Kibana`
Finally we want to analyze the logs generated by (your) application in `Kibana`.

- Go to Kibana: [https://logs.cf.sap.hana.ondemand.com](https://logs.cf.sap.hana.ondemand.com) and login with your CF user, i.e. usually your domain user and GLOBAL domain password.
- The landing page of `Kibana` is the `Dashboard` page providing some other dashboards like Usage, Perfomance, Network load and others. 
- At the top right you can adapt the timelines to your need and enable "Auto-refresh". The default is that log messages of the last 15 minutes are shown.
- On the `Visualize` page you may create and share your own dashboards. Creating dashboards is not part of this exercise.

## Step 4: Search for messages from your component
The `Overview`-`Dashboard` page shows your CF organizations and spaces, and also all applications (in the "Components" view).
You can limit the shown log messages by clicking on the corresponding org/space/application.

For this exercise step, limit the logs to the "bulletinboard-ads" application. You may also limit to the "trial" organization and your "D012345" space.
Notice that the selected values appear as green filters on the top, below the search field.

To see the actual log messages, you have to navigate to the `Requests and Logs`-`Dashboard` page (in the "Navigation" view).
Note that the filter settings are lost when you switch the page.
To work around this, go to the `Overview`-`Dashboard` page and re-apply the filters by selecting your application.
Then, hover the mouse pointer over each filter and pin it down by clicking on the pin icon. __Pinned filters__ will stay active even when you switch pages.


## Step 5: Organize the columns in the application logs view
Again, navigate to the `Requests and Logs`-`Dashboard` view and examine the log messages in the "Requests" and "Application Logs" views. The "Application Logs" view by default shows some columns like the component_name, correlation_id, level, msg,	logger.

To add more columns to the view, expand the first **application log message of your `AdvertisementController` logger** by clicking on the arrow on the left side of the log. In the table view scroll down, find the `endpoint` parameter (which you set in the MDC as part of exercise 13) and click on the symbol you can see in the screenshot below:

![](images/Screenshot-SetKibanaColumn.png)

Then, collapse the log list entry you have expanded. Now you can see that the endpoint is displayed as an additional column.

You can remove columns from the overview list either by again clicking the symbol shown in the screenshot above. Alternative approach: mouse over the headline of a column to be removed and click on the "x" (mouse-over-text: Remove column).

## Step 6: Add and remove filters
In Kibana you can define positive and negative filters for messages.
We already added (positive) filters by selecting entries in the dashboard, which suffices for the basic scenarios.

If a positive filter is used, only messages matching the filter are shown. For example, you can define a positive filter so that only log messages for a specific Java class are shown.

- Find and expand an application log entry that contains the `endpoint` information. Scroll down to the parameter `endpoint` and click on the magnifier symbol with the plus symbol inside. When collapsing the entry you should see only log entries matching the filter, which appears with a *green background* directly under the search field.
- To remove a set filter, mouse over to the filter which is displayed directly under the search field and click on the garbage can symbol.

Messages matching a negative filter are not shown. This can be used to remove certain messages from the output, for example you can define a negative filter removing with log level `INFO` (so that you can better concentrate on warnings and errors).

- Choose a log entry where the value of the parameter `level` is `INFO`. Scroll down to the parameter `level` and click on the magnifier symbol with the minus symbol inside. Now all log entries with log level `INFO` should be filtered out and you should see this filter with a *red background* directly under the search field.
- To remove a set filter, mouse over to the filter which is displayed directly under the search field and click on the garbage can symbol. Perform this for the filter you just have set (`level: "info"`).

**Note:** If you add another / new field, you need to wait up to 60 minutes, till it is available/visible in Kibana, as a cronjob actualizes the indices once per hour.


### Optional ideas
- Create at least two instances of you app (`cf scale -i 2 bulletinboard-ads`). Then perform a few get requests and have a look at the `source_instance` field in Kibana.
- Explore the other pages of the dashboard and filter based on response times, HTTP status codes, ...
- For the "raw" experience, play around in the "discover" view. Notice that all different kinds of logs (check the `layer` column) are shown here. For example `layer`=`RTR` are CF Router specific logs, `STG` logs are emitted when an application is `(re-)staged`. Find further log types [here](https://docs.cloudfoundry.org/devguide/deploy-apps/streaming-logs.html). Find out which `layer` your application specific log message is assigned to?

## Used frameworks and tools
- [Kibana App Performance Cockpit 4 CF](https://logs.cf.sap.hana.ondemand.com/)
- [Kibana documentation](https://www.elastic.co/products/kibana)
- [CloudFoundry Loggregator](https://github.com/cloudfoundry/Loggregator)
- [Logging Library](https://github.com/SAP/cf-java-logging-support)

## Further Reading
- [PerfX Jam Group - primary ELK stack offering at SAP CP](https://jam4.sapjam.com/groups/about_page/pleCfjogSvhtRhOssiLyWl)
- [Howto change the log-level for a single thread by providing a token in the request header](https://github.com/SAP/cf-java-logging-support/wiki/Dynamic-Log-Levels)


***
<dl>
  <dd>
  <div class="footer">&copy; 2018 SAP SE</div>
  </dd>
</dl>
<hr>
<a href="/LoggingTracing/Exercise_13_Use_SLF4J_Features.md">
  <img align="left" alt="Previous Exercise">
</a>
<a href="/LoggingTracing/Exercise_15_Debug_CF_Application.md">
  <img align="right" alt="Next Exercise">
</a>
