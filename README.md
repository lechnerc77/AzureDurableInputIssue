# Azure Durable Orchestration - Issue with Input

## Scenario

The basic scenario is a simple Durable Functions Orchestrator scenario. The basic setup is taken from the templates of the the VS Code extension for Azure Functions. The language is TypeScript, the Node version is 10.

The setup consists of a
* Starter function via HTTP trigger
* Orchestrator function
* Activity function

There sems to be an issue with handing over input to the durable orchestration context when the newest function extension is used as described below.

The execution and test is done locally (`AzureWebJobsStorage` is set to `UseDevelopmentStorage=true` )

### Variant 1 (Branch master )
This variant uses the extension bundle in the `host.json` file. 
The functions are started via `npm run start`.

The following calls are made:

 HTTP Call (GET) | Body (JSON)          | Result 
---------------- | -------------------- | ------- 
 http://localhost:7071/api/orchestrators/DurableFunctionsOrchestratorJS  | empty | OK 
 http://localhost:7071/api/orchestrators/DurableFunctionsOrchestratorJS  | {"id": "12345"} | OK 

In the second table line the input is available in the orchestrator function in the context (i.e. `context.bindingData.input`)

### Variant 2 (branch plainextension)
In this variant the extension bundle is removed from the `host.json`. 
The extension version 2.1.1 is installed via `func extensions install -p Microsoft.Azure.WebJobs.Extensions.DurableTask -v 2.1.1`
The function type in the `function.json` file of the HTTP trigger is switched to `durableClient`. 

The following calls are made:

HTTP Call (GET)  | Body (JSON)          | Result  
---------------- | -------------------- | ------- 
http://localhost:7071/api/orchestrators/DurableFunctionsOrchestratorJS  | empty | OK 
http://localhost:7071/api/orchestrators/DurableFunctionsOrchestratorJS  | {"id": "12345"} | NOK - Error is raised 

### Error in Variant 2

The following message is send to the caller:
```
Executed 'Functions.DurableFunctionsHttpStart' (Failed, Id=8626ff18-a7b1-49c3-8b47-ec8a2c43dd5e)
System.Private.CoreLib: Exception while executing function: Functions.DurableFunctionsHttpStart. System.Private.CoreLib: Result: Failure
Exception: Error: {"Message":"Something went wrong while processing your request","ExceptionMessage":"Object reference not set to an instance of an object.","ExceptionType":"System.NullReferenceException","StackTrace":"   at Microsoft.Azure.WebJobs.Extensions.DurableTask.HttpApiHandler.HandleStartOrchestratorRequestAsync(HttpRequestMessage request, String functionName, String instanceId) in d:\\a\\r1\\a\\azure-functions-durable-extension\\src\\WebJobs.Extensions.DurableTask\\HttpApiHandler.cs:line 666\r\n   at Microsoft.Azure.WebJobs.Extensions.DurableTask.HttpApiHandler.HandleRequestAsync(HttpRequestMessage request) in d:\\a\\r1\\a\\azure-functions-durable-extension\\src\\WebJobs.Extensions.DurableTask\\HttpApiHandler.cs:line 206"}
Stack: Error: {"Message":"Something went wrong while processing your request","ExceptionMessage":"Object reference not set to an instance of an object.","ExceptionType":"System.NullReferenceException","StackTrace":"   at Microsoft.Azure.WebJobs.Extensions.DurableTask.HttpApiHandler.HandleStartOrchestratorRequestAsync(HttpRequestMessage request, String functionName, String instanceId) in d:\\a\\r1\\a\\azure-functions-durable-extension\\src\\WebJobs.Extensions.DurableTask\\HttpApiHandler.cs:line 666\r\n   at Microsoft.Azure.WebJobs.Extensions.DurableTask.HttpApiHandler.HandleRequestAsync(HttpRequestMessage request) in d:\\a\\r1\\a\\azure-functions-durable-extension\\src\\WebJobs.Extensions.DurableTask\\HttpApiHandler.cs:line 206"}
    at DurableOrchestrationClient.<anonymous> (C:\Users\xxx\sandbox\AzureDurableTestSimple\node_modules\durable-functions\lib\src\durableorchestrationclient.js:340:43)
    at Generator.next (<anonymous>)
    at fulfilled (C:\Users\xxx\sandbox\AzureDurableTestSimple\node_modules\durable-functions\lib\src\durableorchestrationclient.js:4:58)
    at process._tickCallback (internal/process/next_tick.js:68:7).
```

### Consequence
Limitation in using Durable Functions when durable entities are needed and input must be transfered to the orchestartor