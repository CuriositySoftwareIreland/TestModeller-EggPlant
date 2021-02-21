# TestModeller.io Eggplant

Eggplant client library for the Test Modeller platform.
Perform and retrieve data allocations, synchronise run results, and execute jobs.

1. Associate the CuriosityAllocationHandler.script handler with your Eggplant project.

## Data Allocation
Test data allocation works in three phrases. 

1. The test data required must be specified and exposed as a test data criterion within a data catalogue.

2. The automation framework must call the data catalogue API to execute and run the data allocations (this is what finds and makes the right data then assigns it to each test instance). 

3. Each test can retrieve the allocated data values which can then be used within the automation framework to perform the required actions across user-interfaces, APIs, mainframes, ETL environments, and much more. This is built to plug directly into an automation framework independent of the language or type of automation being executed.

Firstly you need to configure the curiosity library with your details
```
CuriosityAllocationHandler.initConfigurations "APIUrl", "APIKey", "ServerName"
```

Then you need to specify what test groups you want to allocate.
```
set poolName to "Eggplant"
set allocationTypes to [{"poolName": poolName, "suiteName": "FindAmericanUsers", "allocationTestName": "Find american users"}]
```

Here you can specify the data allocation to connect the test with. This corresponds to three parameters:
1.	poolName – Name of the allocation pool the tests reside in.

2.	suiteName – Name of the test suite the test resides in.

3.	allocationTestName – The tests to perform allocation on. These are the allocation tests associated with the current test being tagged. Wildcards can be used to match multiple test names. The groups tag also takes a list so multiple test types can be specified.

These three parameters must match the data values specified for each matching test case specified within the appropriate allocation pool within the portal.

```
CuriosityAllocationHandler.runAllocation poolName, allocationTypes

```
Before the tests are executed we have defined a function which takes all the collected allocations tagged against any tests about to be executed and then call the data allocation API to perform the associated executions. 

It is more efficient to perform these operations in bulk which is why they are collected into one list and then sent for allocation as opposed to directly performing the allocation inside each individual script. 

```
CuriosityAllocationHandler.retrieveAllocationResult poolName, "FindAmericanUsers", "Find american users"
set res to the result

log res 
set firstRow to item 1 of res.dataRows
log "Password result: " & firstRow.Username 
```

Within the test case you can retrieve the results using the 'CuriosityAllocationHandler.retrieveAllocationResult’ function. Here you can specify the pool, suite name, and test name to retrieve the results for. Again, this must match the specifications given in the associated allocation pool within the portal. The return result contains the functions to retrieve results by the column names, and column indexes as specified in the initial test criteria.

## Job Execution
When executing a VIP job through the Test Modeller portal there is the option to "Download Executor". This downloads a ZIP folder containing:
1. config.xml - This has the parameters and value pairs for each value that can be input into the selected process.
2. run.bat - A batch file for running the task.
3. TestModeller.jar - A java executable JAR which wraps around the TestModeller API to execute the selected task.

![Download Executor](https://curiositysoftware.ie/Resources/downloadExecutor.gif)

The job can be executed via the command line by running the selected bat file. Any values can be directly edited within the config.xml. The batch file can be wrapped into Eggplant code as below.
```
// Run a job using the Test Modeller JAR executor
shell("c:\vip\runJob.bat")

put "c:\vip\resultFile.zip" into resultFile
repeat until there is a file resultFile
	wait 1
end repeat

log "Job Processed"
```

Any values can be overriden at runtime by passing the arguments as a JSON message into the batch file. For example:
```
java -jar TestModeller.jar config.xml output.zip "{\"parProfileId\":\"OverridingProfileId\"}"
```

Note: The java location may need to be directly hard coded if it is not available in the systems environment PATH.
