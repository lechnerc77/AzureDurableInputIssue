# Azure Durable Orchestration - Regression check for Input

## Scenario

The basic scenario is a simple Durable Functions Orchestrator scenario. The basic setup is taken from the templates of the the VS Code extension for Azure Functions. The language is TypeScript, the Node version is 10.

The setup consists of a
* Starter function via HTTP trigger
* Orchestrator function
* Activity function

There was an issue with handing over input to the durable orchestration context.
The issue was resolved with version 2.2.0 of the durable extension.
However for future quick "regression" checks you can still use this repo. Both branches should work. 

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
The extension version 2.2.0 is installed via `func extensions install -p Microsoft.Azure.WebJobs.Extensions.DurableTask -v 2.2.0`
The function type in the `function.json` file of the HTTP trigger is switched to `durableClient`. 

The following calls are made:

HTTP Call (GET)  | Body (JSON)          | Result  
---------------- | -------------------- | ------- 
http://localhost:7071/api/orchestrators/DurableFunctionsOrchestratorJS  | empty | OK 
http://localhost:7071/api/orchestrators/DurableFunctionsOrchestratorJS  | {"id": "12345"} | OK (as of version 2.2.0) 