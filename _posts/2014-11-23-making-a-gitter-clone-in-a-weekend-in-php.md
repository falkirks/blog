---
title: Making a PHP Gitter clone in a weekend
layout: post
permalink: making-a-gitter-clone-in-a-weekend-in-php
published: true
---
Last weekend a group of developers from the PocketMine community, myself included, set out to make an ugly and featureless clone of Gitter written in PHP. We established our goals early on:

* Repo based chat system (no history)
* WebSockets browser interface
* Bridge to IRC
* Some method to store chat history on browser

With these goals in mind we decided to use MongoDB for the backend despite our lack of experience with it. Using this as a data structure provided much more freedom them a typical SQL database would. The development was greatly catalyzed by this tool and it allowed us to quickly add features without rewriting the databse setup. 

It was evident that we would require a template system and for this we chose to go with {{ mustache }} because it is easy to use and makes logical templates.

---

### Sessions

A major stumbling block for us was sessions. It held up the project for a large chunk of the Saturday. We settled on using `$_SESSION` to store session information. For the format we  used `[user]]\\md5(time() . $ip)` for simplicity. The session data was stored in the database to allow the easy termination of sessions in the settings pane.

---

### WebSockets 
To use WebSockets we implemented a custom protocol for communication. The packets included a JSON object with a type and a payload. For example the follow is used to autheticate:
```javascript
{ 
	type: "auth",
    payload: {
 		key: sessionkey   
    }
}
```
The protcol contains the types (**bold** indicates a two way packet with different payload formats):

* auth
* authreply
* **message**
* channel
	* add
    * remove
    * error
    
---
### LocalStorage of chat history
Storing chat history and current channels in local storage seemed like an awesome idea. We loaded up **basil.js** to make things easier. It was a breeze to implement but we soon realized that it was riddled with concurrent modification issues regarding duplicate messages and channels not adding or removing properly. We solved this in two less than pretty ways:

1. To remove the duplicate messages we used some ugly logic to sort messages and hopefully prevent messages from other tabs while still allowing messages from other computers logged into the same account.
2.  To combat the issue were channels didn't get removed or added to other tabs we used a simple `window.setTimeout()` are reloaded the channels every second or so. 

---

### An IRC Bridge
The IRC protocol is painful and annoying to implement in a compliant way. Still we did our best to get our IRC to be standards compliant and working with all modern IRC clients. For authentication we decided against a NickServ type approach and we used the server password to store the user password. So when a user joins they need to specify a username and set the server password to the corresponding password. 
