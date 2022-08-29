# FRT Project
# Azure Resume
## 1.Frontend
- Website template (HTML/CSS/JavaScript)
- visit counter (JavaScript)

## 2.Backend
- CosmosDB
- Azure Functions

I've written all the code in Visual Studio Code with Azure Tools extensions. It's available on GitHub
#Building frontend
Website template
I downloaded a juicy template written in the standard frontend technologies: HTML, CSS, JavaScript.  I applied JavaScript to fetch data and prepare animations.
#Visit counter
Here I present JavaScript code for getting a counter value from a database when the DOM content is loaded.
#
// Get data when DOM content is loaded
window.addEventListener('DOMContentLoaded', (event) =>{
  getVisitCount();
})

const functionApiUrl = 'https://getresumecounteracgdl.azurewebsites.net/api/GetResumeCounter?code=KOrzyAs7Lcp96TjxjSvHLbMxuAcG08WKQby42gUtrGe6L3OhfgqJRA=='
const localFunctionApi = 'http://localhost:7071/api/GetResumeCounter';

const getVisitCount = () => {
  let count = 30;
  fetch(functionApiUrl).then(response => {
    return response.json()
  }).then(response => {
    console.log("Website called function API");
    count = response.count;
    document.getElementById("counter").innerText = count;
  }).catch(function(error) {
    console.log(error);
  });
  return count;
}
#
Finally, I deployed the website to **Azure Blob storage**.

# Building backend

## CosmosDB
CosmosDB is a database service that allowed me to quickly set up a database that stores a Counter container and one item with two fields: "id" and "count" for the actual value of a counter. Other fields such as "_rid" are system-generated properties.
![Screenshot (10)](https://user-images.githubusercontent.com/108383497/187209349-f0dfc89f-b273-4bad-ba30-d4f16dce0ef4.jpg)

# Azure Functions
We need a tool to increment the counter value when a specific event happens, that is, when the page is visited. Azure Functions is suitable for this task. Azure Functions is a cloud service that eases backend work. I don't need to manage infrastructure and resources; Functions will do this for me. It is an event-driven service, so in my case, HTTP requests will trigger interaction with the database. Here is the code for HTTP-triggered Azure Function implemented in C# (.NET Core 3 LTS):
#
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using System.Net.Http;
using System.Text;

namespace Company.Function
{
    public static class GetResumeCounter
    {
        [FunctionName("GetResumeCounter")]
        public static HttpResponseMessage Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            [CosmosDB(databaseName: "AzureResume", collectionName: "Counter", ConnectionStringSetting = "AzureResumeConnectionString", Id = "1", PartitionKey = "1")] Counter counter,
            [CosmosDB(databaseName: "AzureResume", collectionName: "Counter", ConnectionStringSetting = "AzureResumeConnectionString", Id = "1", PartitionKey = "1")] out Counter updatedCounter,
            ILogger log)
        {
            // Here is where the counter gets updated.
            log.LogInformation("C# HTTP trigger function processed a request.");
            updatedCounter = counter;
            updatedCounter.Count += 1;

            var jsonToReturn = JsonConvert.SerializeObject(counter);

            return new HttpResponseMessage(System.Net.HttpStatusCode.OK)

            {
                Content = new StringContent(jsonToReturn, Encoding.UTF8, "application/json")
            };

        }
    }
}
#
# Binding between CosmosDB and Azure Functions
Additionally, I have to prepare a binding for CosmosDB and Azure Functions. 
