
## METHOD -I

### we will use Postman to collect API requests and manually build out a collection.

In the instance where there is no documentation and no specification file, you will have to reverse-engineer the API based on your interactions with it.

#### Mapping an API with several endpoints and a few methods can quickly grow into quite a large attack surface. To manage this process, build the requests under a collection in order to thoroughly hack the API. Postman can help you keep track of all of these requests.

#### There are two ways to manually reverse engineer an API with Postman.

##### I. One way is by constructing each request. While this can be a bit cumbersome, it will allow you to add the precise requests you care about.

#### II. The other way is to proxy web traffic through Postman, then use it to capture a stream of requests. This process makes it much easier to construct requests within Postman, but you’ll have to remove or ignore unrelated requests.

```
#To start the crAPI
sudo docker-compose -f docker-compose.yml --compatibility up -d

```




## METHOD - II

Steps:

I. in command line:

First, we will begin by proxying all web application traffic using mitmweb.
Simply use a terminal and run:
```
mitmweb
```

This will create a proxy listener using port 8080. You can then open a browser and use FoxyProxy to proxy your browser to port 8080 using our Burp Suite option.

II.  Once the proxy is set up, you can once again use the target web application as it was intended.

_Note: if you are reverse engineering crapi.apisec.ai you may run into certificate issues if you have not yet added the mitmweb cert. Please return back to Kali Linux and More MITMweb Certificate Setup for instructions._

Every request that is created from your actions will be captured by the mitmweb proxy. You can see the captured traffic by using a browser to visit the mitmweb web server located at http://127.0.0.1:8081.

III. Continue to explore the target web application until there is nothing left to do. Once you have exhausted what can be done with your target web app, return to the mitmweb web server and click **File > Save** to save the captured requests.

IV. Selecting **Save** will create a file called flows. We can use the "flows" file to create our own API documentation.

V. Using a great tool called mitmproxy2swagger, we will be able to transform our captured traffic into an Open API 3.0 YAML file that can be viewed in a browser and imported as a collection into Postman.

VI. 
```
$sudo mitmproxy2swagger -i /Downloads/flows -o spec.yml -p [http://crapi.apisec.ai](http://crapi.apisec.ai/) -f flow
```

VII.
After running this you will need to edit the spec.yml file to see if mitmproxy2swagger has ignored too many endpoints. Checking out spec.yml reveals that there are several endpoints that were ignored and the title of the file can be updated.

Update the YAML file so that "ignore:" is removed from the endpoints that you want to include. Once you have edited the file it should look something similar to the image below. Note that the "title" has been updated to "crAPI Swagger" and that the endpoints no longer contain "ignore:".

VIII. Save the updated spec.yml file and run the mitmproxy2swagger again. This time around add the "--examples" flag to enhance your API documentation.

IX. After running mitmproxy2swagger successfully a second time through, your reverse-engineered documentation should be ready. You can validate the documentation by visiting [https://editor.swagger.io/](https://editor.swagger.io/) and by importing your spec file into the Swagger Editor.

X. Use **File>Import file** and select your spec.yml file. If everything has gone as planned then you should see something like the image below. This is a pretty good indication of success, but to be sure we can also import this file as a Postman Collection that way we can prepare to attack the target API.

XI. To import this file as a collection we will need to open Postman. At the top left of your Postman Workspace, you can click the "Import" button. Next, select the spec.yml file and import the collection.

XII. Once you import the file you should see a relatively straightforward API collection that can be used to analyze the target API and exploit with future attacks.

