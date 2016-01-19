---
layout: post
title:  "A Mule RESTful API Client Connector"
date:   2016-01-09 21:06:19 -0500
tags: technical
subclass: 'post tag-technical'
categories: 'dlwhitehurst' 
---
I've recently been learning about the Mule ESB and Integration platform. I wanted to create my own custom connector to work with a RESTful API and use the data in a Mule flow. The connector was written to connect to an application created using a [JHipster][] Yeoman 
generator. The beauty of JHipster is the ability to generate a Spring-boot/AngularJS application quickly and also to generate entities with CRUD operations for persistence. JHipster's Angular(UI) layer uses Spring REST Controllers(Back-end) for it's CRUD operations. 

I choose to create a general account entity in JHipster. The account has a String name and Long id. The connector will call the JHipster API e.g. /api/genaccounts and the returned listing of general accounts (JSON array) will be listed in the browser window using the Mule HTTP connector in the flow design.

 ![mules]({{ site.baseurl }}assets/images/mule-3.jpg)
The connector project was created in AnyPoint Studio (Mule's Eclipse Environment) using 
the AnyPoint Connector Devkit that had to be downloaded and installed in AnyPoint. Devkit can be downloaded here https://docs.mulesoft.com/anypoint-connector-devkit/v/3.8/ . I used version 3.8. This may have changed since this writing. I obtained this software via the Eclipse update process using the IDE. Once I agreed to all the EULA's that MuleSoft provided, the software installed and I was ready to begin my new connector.

Since, I've already created a working connector, I'm going to create a new connector project and document everything as I implement the connector and test it. This way I'll be sure to note anything difficult or confusing as we go. Let's open AnyPoint Studio and create a new project. Select File/New/AnyPoint Connector Project or if the choice doesn't exist, select File/New/Other... and choose the Mule wizard and then select the AnyPoint Connector Project. Press the Next> button to proceed. 

The dialog that appears provides two choices for the type of connector project. You will want to select SDK Based and then the Next> button to proceed. 

The dialog that is now presented collects metadata for the project. This is where you will name the connector. My connector name is CIWiseREST. You do not have to add the text "connector". The project generator will do this for you. Please notice that the project name and namespace text boxes will fill automatically as you name the connector itself. Be sure to select the "Generate sample operations and configurable fields" checkbox. We will use a very simple operation to prove that the connector will install and can be used in a Mule flow on a running server instance. The API Type is Java SDK and the Authentication drop down should be set to "None". We will use authentication but we'll implement that
code ourselves. Nothing else should be checked. Select Next>.

The dialog presented contains Maven and Github configuration information. We will unselect the Maven default checkbox and add information for the group-id as "org.ciwise.modules". Select Next>.

This last dialog provides for customization of the connector icon and also a text layer over the icon. I changed the label to read "REST". That's it. We're done and can now select Finish. The code for the project will now be generated.

Two Java classes are generated:
- org.ciwise.modules.ciwiserest.CIWiseRESTConnector.java
- org.ciwise.modules.ciwiserest.config.ConnectorConfig.java

The connector class is called the @Connector class my the MuleSoft folks because this class must be annotated with @Connector. This is not optional. Mule has a code you don't see that allows the connector to be used in a drag-and-drop window and integrate with other connectors that make up a Mule flow. Here's a code listing of this class:

**Listing 1 - CIWiseRESTConnector.java**

{% highlight java %}
package org.ciwise.modules.ciwiserest;

import org.mule.api.annotations.Config;
import org.mule.api.annotations.Connector;
import org.mule.api.annotations.Processor;
import org.ciwise.modules.ciwiserest.config.ConnectorConfig;

@Connector(name="ci-wise-rest", friendlyName="CIWiseREST")
public class CIWiseRESTConnector {

    @Config
    ConnectorConfig config;

    /**
     * Custom processor
     *
     * @param friend Name to be used to generate a greeting message.
     * @return A greeting message
     */
    @Processor
    public String greet(String friend) {
        /*
         * MESSAGE PROCESSOR CODE GOES HERE
         */
        return config.getGreeting() + " " + friend + ". " + config.getReply();
    }

    public ConnectorConfig getConfig() {
        return config;
    }

    public void setConfig(ConnectorConfig config) {
        this.config = config;
    }

}
{% endhighlight %}

Let's build the connector and install it into AnyPoint Studio so we can test it with Mule. The connector project when created added the greet() method to the @Connector class because we checked the box to add an example operation. This operation is shown by the greet() method and annotated with the @Processor notation. When the Mule connector is selected in a Mule Project flow, the operations that have been annotated and defined, are selectable in the IDE to be processed during communications across the Mule flow in production. Select the project ci-wise-rest-connector in the Package explorer and right-mouse. Next select AnyPoint Connector at the bottom and then Build Connector. An Apache Maven build will commence. The project should build successfully.

To install the connector for testing, follow the same steps and when the final pop-up menu is shown, select Install or Update. The IDE will install and accept the connector for use when designing a Mule flow. A dialog should be shown when the installation is complete providing confirmation that the install was successful.

To test the connector using Mule, I created a Mule project called muletest. Open the Mule test project and drag an HTTP connector into the Mule Flow view. Set the properties of the connector to use localhost and port 8081. Next, find CIWiseREST in the Mule palette and drag your new connector into the flow beside te HTTP connector. Select your connector in the flow view and double-click. The connector properties window will be shown. Under Basic Settings/Connector Configuration, select the add or plus icon. Select "CIWiseREST__Configuration". Now select the Operation drop-down and choose "Greet". The greet method required an input string called friend and under General/Friend: you need to give the operation this String. Enter "John". When we run the Mule application and give the browser the URL http://localhost:8081 we should expect to see "Hello John. How are you?". That's it. It doesn't do much, however it demonstrates the @Connector and @Process annotations and how they are used with a Mule Flow. 

Click in the Mule flow view and the little red X in the upper right of your connector should go away as there are no errors now. Press the Save All icon in the IDE. We are now ready to test our connector. Click on the Mule flow XML file in the Package Explorer and right-mouse. Choose Run-As/Mule Application. When Mule says it's deployed and successful, open a browser and enter http://localhost:8081. I see "How are you? John. Hello". I set my configuration backwards. The return statement in the connector
operation method looks like this.

**Listing 2 - Snippet from CIWiseRESTConnector.java**
{% highlight java %}
    @Processor
    public String greet(String friend) {
        /*
         * MESSAGE PROCESSOR CODE GOES HERE
         */
        return config.getGreeting() + " " + friend + ". " + config.getReply();
    }

{% endhighlight %}

And, here's the source for the Configuration.

**Listing 3 - ConnectorConfig.java**
{% highlight java %}
package org.ciwise.modules.ciwiserest.config;

import org.mule.api.annotations.components.Configuration;

@Configuration(friendlyName = "Configuration")
public class ConnectorConfig {

	private final String reply = "Hello";
	private final String greeting = "How are you?";

	public String getReply() {
		return reply;
	}
	public String getGreeting() {
		return greeting;
	}
	
}
{% endhighlight %}

The greeting should be "Hello" and the reply should be "How are you?". Delete the connector from the Mule flow view. Swap the Strings, rebuild the connector, and then try to update the installation. I found that for some reason the Maven clean operation would not work. I'm still trying to figure this out, but you can uninstall the connector, delete the target directory in a terminal or file explorer and then build the connector and install it successfully. Do this and re-test the connector using Mule. I just did and everything is fine now.

Now that I've shown the Mule integration of the generated connector in the flow view, let's add a realistic operation to pull a simple list of accounts from a RESTful service and display the listing in the browser. I needed a simple service with a well documented API. My JHipster application will provide the RESTful service using Spring REST
controllers. It also provides a hosted Swagger API listing with the application. I created a simple entity object called "genaccount" with an id, and a String name. We'll use the JHipster /api/genaccounts call to get our listing.

To start we should create the packages for the client-side, connector entities. These objects will be used when Jackson deserializes the JSON that the JHipster application will send to our connector when we make the api call. Our project package base is org.ciwise.modules.ciwiserest. Create a "base package".entities package. Before we call for the list of accounts, we must obtain a user authentication token. This token is sent
with all requests. We'll get the token and keep it in a private String member within the our Jersey client The token is obtained with the REST call /api/authenticate. The JHipster application returns a JSON object like so.

**Listing 4 - JSON return POST /api/authenticate**
{% highlight javascript %}
{
  "expires": 0,
  "token": "string"
}
{% endhighlight %}

The accounts are just String names and this JSON model schema is like so.

**Listing 5 - JSON return GET /api/genaccounts**
{% highlight javascript %}
[
  {
    "acctno": 0,
    "accttype": "string",
    "id": 0,
    "name": "string"
  }
]
{% endhighlight %}

Notice the square-brackets around the JSON object. This means that JHipster will be sending an array of JSON objects like the one defined in the curly-braces. I showed both API returns first because I wanted to show the simple data that the JHipster application is sending. On our end, within the connector, we don't generally use JSON. We want to be able to serialize and deserialize matching Java objects or Plain Old Java Objects (POJOs). And, looking at the JSON above, these Java classes will be very simple. And, why wouldn't we just create the classes, add the private members, and use Eclipse to create the get() and set() methods now? We're not going to do that because we need to also annotate the simple Java classes with Jackson 1.x annotations. And, we're going to use an online utility to generate the classes and annotate them from our JSON schema.

I wasn't familiar with the [Jackson][] annotations but I knew what the process did. And, there's a website called [JSONSchema2POJO][] that will accept JSON schema or JSON and create the POJO classes we need. I'm not going to discuss the operation of the site or how I created the files but I'll list them here for you to review.

**Listing 6 - Token.java**
{% highlight java %}
package org.ciwise.modules.ciwiserest.entity;

import java.util.HashMap;
import java.util.Map;
import javax.annotation.Generated;
import org.codehaus.jackson.annotate.JsonAnyGetter;
import org.codehaus.jackson.annotate.JsonAnySetter;
import org.codehaus.jackson.annotate.JsonIgnore;
import org.codehaus.jackson.annotate.JsonProperty;
import org.codehaus.jackson.annotate.JsonPropertyOrder;
import org.codehaus.jackson.map.annotate.JsonSerialize;

/**
 * POJO for mapping with JSON response api/authenticate
 * 
 * @author David L. Whitehurst
 *
 */
@JsonSerialize(include = JsonSerialize.Inclusion.NON_NULL)
@Generated("org.jsonschema2pojo")
@JsonPropertyOrder({ "expires", "token" })
public class Token {

	@JsonProperty("expires")
	private Object expires;
	@JsonProperty("token")
	private Object token;
	@JsonIgnore
	private Map<String, Object> additionalProperties = new HashMap<String, Object>();

	/**
	 * 
	 * @return The expires
	 */
	@JsonProperty("expires")
	public Object getExpires() {
		return expires;
	}

	/**
	 * 
	 * @param expires
	 *            The expires
	 */
	@JsonProperty("expires")
	public void setExpires(Object expires) {
		this.expires = expires;
	}

	/**
	 * 
	 * @return The token
	 */
	@JsonProperty("token")
	public Object getToken() {
		return token;
	}

	/**
	 * 
	 * @param token
	 *            The token
	 */
	@JsonProperty("token")
	public void setToken(Object token) {
		this.token = token;
	}

	@JsonAnyGetter
	public Map<String, Object> getAdditionalProperties() {
		return this.additionalProperties;
	}

	@JsonAnySetter
	public void setAdditionalProperty(String name, Object value) {
		this.additionalProperties.put(name, value);
	}

}
{% endhighlight %}

**Listing 7 - Account.java**
{% highlight java %}
package org.ciwise.modules.ciwiserest.entity;

import java.util.HashMap;
import java.util.Map;
import javax.annotation.Generated;
import org.codehaus.jackson.annotate.JsonAnyGetter;
import org.codehaus.jackson.annotate.JsonAnySetter;
import org.codehaus.jackson.annotate.JsonIgnore;
import org.codehaus.jackson.annotate.JsonProperty;
import org.codehaus.jackson.annotate.JsonPropertyOrder;
import org.codehaus.jackson.map.annotate.JsonSerialize;

@JsonSerialize(include = JsonSerialize.Inclusion.NON_NULL)
@Generated("org.jsonschema2pojo")
@JsonPropertyOrder({
"acctno",
"id",
"name"
})
public class Account {

@JsonProperty("acctno")
private int acctno;
@JsonProperty("id")
private int id;
@JsonProperty("name")
private String name;
@JsonIgnore
private Map<String, Object> additionalProperties = new HashMap<String, Object>();

/**
* 
* @return
* The acctno
*/
@JsonProperty("acctno")
public int getAcctno() {
return acctno;
}

/**
* 
* @param acctno
* The acctno
*/
@JsonProperty("acctno")
public void setAcctno(int acctno) {
this.acctno = acctno;
}

/**
* 
* @return
* The id
*/
@JsonProperty("id")
public int getId() {
return id;
}

/**
* 
* @param id
* The id
*/
@JsonProperty("id")
public void setId(int id) {
this.id = id;
}

/**
* 
* @return
* The name
*/
@JsonProperty("name")
public String getName() {
return name;
}

/**
* 
* @param name
* The name
*/
@JsonProperty("name")
public void setName(String name) {
this.name = name;
}

@JsonAnyGetter
public Map<String, Object> getAdditionalProperties() {
return this.additionalProperties;
}

@JsonAnySetter
public void setAdditionalProperty(String name, Object value) {
this.additionalProperties.put(name, value);
}

}
{% endhighlight %}

The next step is to create a package for any exceptions our client class might encounter. Create a package "base-package".exception and the following classes to extend Exception:
 
 * CIWiseRESTConnectorException
 * CIWiseRESTConnectorTokenExpiredException
 
I'm going to assume you don't need to see these, but you will need them. Now, we'll create the real nuts and bolts of the connector, the client class.

Create a package "base-package".client and then create a class called RestClient. This class will implement the Jersey-client. I'll discuss the pieces of the class and share snippets as we go.

The connector class needs two public methods. These are:

**Listing 8 - RestClient (public methods signatures) **
{% highlight java %}
	
	public Token getXAuthToken() throws CIWiseRESTConnectorException,
		CIWiseRESTConnectorTokenExpiredException {
		WebResource webResource = getApiResource().path("api").path(
				"authenticate");
		return executeAuthenticationPost(webResource, Token.class);
	}
	
	public Account[] getAccounts() throws CIWiseRESTConnectorException,
	CIWiseRESTConnectorTokenExpiredException {

		WebResource webResource = getApiResource().path("api").path(
		"genaccounts");
		return execute(webResource, "GET", Account[].class);
	}

{% endhighlight %}

We should talk about the connector class now and how it uses these methods to accomplish its goals. I'll address the connector lifecycle and how it relates to the Java annotations. Ultimately our goal is to run the Mule connector operation "Get Accounts". We select that operation in the properties dialog for the custom connector in our Mule flow. This operation is defined here in the connector class.

**Listing 9 - CIWiseRESTConnector (key process method)**
{% highlight java %}

    @Processor(friendlyName="Get Accounts")
    public String getAccounts() throws CIWiseRESTConnectorException, CIWiseRESTConnectorTokenExpiredException {
    	Account[] array = getClient().getAccounts();
    	String retStr = "";
    	for(int i=0; i < array.length; i++)
    	{
    	    retStr = retStr + array[i].getId() + "," + array[i].getName() + "," + array[i].getAcctno() + "\n";
    	    
    	}
    	
    	return retStr;
    }

{% endhighlight %}

Before the String listing of account names can be returned, the connector needs to be instantiated and then the lifecycle process begins. When the connector class is instantiated, a configuration class is also created and the configuration data is then available to the connector for whatever it needs. Next, the @Start annotation is inspected and the initialization method is processed.

**Listing 10 - CIWiseRESTConnector (@Start event)**
{% highlight java %}

    @Start
    public void init() {
        setClient(new RestClient(this));
    }    

{% endhighlight %}
  
When the start event is processed, our client class gets created and the connector instance is passed in, giving access to the configuration data to the client. The Mule documentation calls is wrapping the client, but not exactly. It's not the Decorator pattern where the wrapper class provides a different class signature but with very little functionality. After the @Start annotation, the @Process operation that was selected in the connector's property dialog will be executed. I created an @Process method for getting the X-Auth token but only used it for testing. The next method called after @Start is getAccounts() (annotated with @Process).

**Listing 11 - CIWiseRESTConnector (@Process event)**
{% highlight java %}

    @Processor(friendlyName="Get Accounts")
    public String getAccounts() throws CIWiseRESTConnectorException, CIWiseRESTConnectorTokenExpiredException {
    	Account[] array = getClient().getAccounts();
    	String retStr = "";
    	for(int i=0; i < array.length; i++)
    	{
    	    retStr = retStr + array[i].getId() + "," + array[i].getName() + "," + array[i].getAcctno() + "\n";
    	    
    	}
    	
    	return retStr;
    }

{% endhighlight %} 

The connector calls getAccounts(), the client calls its getAccounts() and our client gets an array of Account Java objects. We can iterate over them and return a textual list of accounts to our browser when our connector is in use. The real work happens in the RestClient class. I'll describe things there now.

I'll start with the getAccounts() method in the client class. It calls upon the private method execute(). I'll provide the listing here.

**Listing 12 - RestClient (execute())**
{% highlight java %}

	/**
	 * Executes an API request
	 *
	 */
	private <T> T execute(WebResource webResource, String method,
			Class<T> returnClass)
			throws CIWiseRESTConnectorTokenExpiredException,
			CIWiseRESTConnectorException {

		/**
		 * Call HTTP Method
		 */
		if (connector.getToken() == null) {
			Token token = getXAuthToken();
			if (token != null) {
				connector.setToken((String) token.getToken()); 
			}
		}
		ClientResponse clientResponse = webResource.accept(
				MediaType.APPLICATION_JSON).header("X-Auth-Token", connector.getToken())
				.method(method, ClientResponse.class);
		if (clientResponse.getStatus() == 200) {
			return clientResponse.getEntity(returnClass);
		} else if (clientResponse.getStatus() == 401) {
			throw new CIWiseRESTConnectorTokenExpiredException(
					"The access token has expired; "
							+ clientResponse.getEntity(String.class));
		} else {
			throw new CIWiseRESTConnectorException(String.format(
					"ERROR - statusCode: %d - message: %s",
					clientResponse.getStatus(),
					clientResponse.getEntity(String.class)));
		}
	}

{% endhighlight %} 

You will notice that the initial logic is to check the connector for a not-null token. And, since we're calling the getAccounts() method, we won't have one. So we must get the token. The token comes by way of the executeAuthenticationPost() method that's somewhat similar to the execute() method. Its listing is here.

**Listing 13 - RestClient (executeAuthenticationPost())**
{% highlight java %}

	/**
	 * Executes an authentication request via HTTP POST
	 *
	 */
	
	private <T> T executeAuthenticationPost(WebResource webResource,
			Class<T> returnClass)
			throws CIWiseRESTConnectorTokenExpiredException,
			CIWiseRESTConnectorException {

		// Map query params for POST operation MultivaluedMap, MultivaluedMapImpl
		
		MultivaluedMap<String,String> queryParams = new MultivaluedMapImpl();
		queryParams.add("username",
				getConnector().getConfig().getUsername());
		queryParams.add("password",
				getConnector().getConfig().getPassword());

		/**
		 * Call HTTP POST
		 */
		ClientResponse clientResponse = webResource.queryParams(queryParams)
				.accept(MediaType.APPLICATION_JSON).post(ClientResponse.class);

		if (clientResponse.getStatus() == 200) {
			return clientResponse.getEntity(returnClass);
		} else if (clientResponse.getStatus() == 401) {
			throw new CIWiseRESTConnectorTokenExpiredException(
					"The access token has expired; "
							+ clientResponse.getEntity(String.class));
		} else {
			throw new CIWiseRESTConnectorException(String.format(
					"ERROR - statusCode: %d - message: %s",
					clientResponse.getStatus(),
					clientResponse.getEntity(String.class)));
		}
	}

{% endhighlight %} 

In this method we send the username and password on an HTTP POST request, in query parameters and the client response object populates the returnClass (Token.java). We never see the JSON deserialization because of the Jackson 1.x annotations on our entity objects. Once we have the token, we save the String token to be part of our /api/genaccounts request next. In listing 12 you will notice that we place the key "X-Auth-Token" and the actual token value on the HTTP header. We make the call and the same JSON deserialization into our Account object(s) occurs. This time the returnClass is Account[].class or an array of Account objects. This is what we needed to display our list of accounts.

You can check out my entire connector project at: [CIWiseRESTConnector][] 

[CIWiseRESTConnector]: https://github.com/ciwise/ci-wise-rest-connector
[JSONSchema2POJO]: http://www.jsonschema2pojo.org/
[JHipster]: http://jhipster.github.io/
[Jackson]: http://wiki.fasterxml.com/JacksonAnnotations
