---
title: HTTPServer beta
layout: post
permalink: httpserver-beta
published: true
---
When I first started work on HTTPServer it was meant to be something awesome, but over the course of development many of the features were sacrificed. This pained me greatly as HTTPServer wasn't what I wanted it to be. As a result, I decided to start work on a full rewrite of the software to be more powerful and more developer-friendly.

### What's changed?
##### Short Answer
Everything.
##### Long Answer
###### Threading.
HTTPServer is now better at threading. Every requst is run by a worker in a pool. This allows multiple requests to be processed in parallel. 
###### Templating
HTTPServer no longer uses its own template system. Now Handlerbars is used.
###### New API
The API has been entirely rewritten to be more fun to use. It is much more logical and powerful.

### The New API
The API is still tentitive and might undergo some large structural changes before its release, so don't get too attached to it. The API centralizes on WebsiteData objects which are magical objects which allow interaction with the server.

#### Getting API access
##### Anonymous
This mode of access is **not** recomended . It allows direct access to the API without any logging or monitoring.
```php
	$data = new \httpserver\WebsiteData();
```
##### Identified 
In an optimal setting you should identify yourself to HTTPServer. This will allow HTTPServer to create logs of your API usage. In order to identify yourself, you will need to pass a PluginBase object to HTTPServer. You can construct a MonitoredWebsiteData directly, but this might not be supported in future versions.
```php
	$data = new \httpserver\MonitoredWebsiteData($this);
```
#### Setting and getting values
Once you have a WebsiteData object, you get a link to the global scope of handlebars variables.
```php
	$data = new \httpserver\WebsiteData(); //We are using anon
    $data["foo"] = ["1", "2", "3"];
    var_dump($data["foo"]); //["1", "2", "3"]
```

#### Changing the scope
Obviously some variables should only be available to /foo/* and some other should only be accessible to /foo/bar/*

**Note** Variables in higher scopes won't be available to lower scope. So if I am in $data->1->2, I won't be able to see $data->1 variables.

```php
	$data = new \httpserver\WebsiteData(); //We are using anon
    $foo = $data->foo; //Switch scope to /foo/*
    $data["thestuff"] = "You are a /foo/* but not a /foo/bar/*";
    $foobar = $foo->bar; //Switch scope to /foo/bar/*
    var_dump($foobar["thestuff"]); //null
    $data["thestuff"] = "You are a /foo/bar/*";
```

#### Dynamic Page Registration
To ease plugin installation, pages can be dynamically registered and stored in memory. This is done by using the magic template variable `_content`. If no page is found and the magic variable is set then the page will be set to its value.

**Note**  Dynamic page registration implementation details are still a work in progress. Dynamic pages are generally bad pratice and so this feature is a low priority.

**Note 2** If a page exists in the HTTPServer folder then it can't be registered  or modified dynamically. But no error will be thrown.

```php
     $data = new \httpserver\WebsiteData(); //We are using anon
     $data->example["_content"] = "Hello {{text}}!"; //Set /example by setting {{_content}} 
     $data->example["text"] = "World"; //Set {{text}} in /example/*
```
### About Custom Scripts
Custom scripts are not yet finalized but likely will support PHP, Python, Node.js and Ruby. These scripts will be injected with global scope variables using loader scripts or data will be writen to the STDIN depending on the language. HTTPServer will listen to the output buffer and foward all data to the client.

### Getting a copy
The beta is not yet ready for public testing. When it is  you can go to https://github.com/Falkirks/HTTPServer/releases to get it.

