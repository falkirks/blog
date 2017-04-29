---
title: Bringing EssentialsPE support to SimpleWarp
layout: post
permalink: essentials-support-in-simplewarp
published: true
---
I released *SimpleWarp* about a year and a half ago. A year after I released it, @LegendsOfMCPE released *EssentialsPE*. *EssentialsPE* included a warp feature which was inspired by *SimpleWarp* (with my permission). This meant that *EssentialsPE* would register the **/warp** and **/delwarp** commands, which would prevent the plugins from working together. I was happy with this as users would either use *SimpleWarp* or *EssentialsPE*, they should never need both. However, recently two things have changed which have made me reconsider that decision.

1. With the release SimpleWarp 2.0, a whole bunch of new feature became available. These features can't be offered to *EssentialsPE* users.
2. I became the maintainer of EssentialsPE, which made me want to ensure some interoperatbility between the plugins. 

Following this, I considered removing warps from *EssentialsPE* or disabling them when *SimpleWarp* was installed. I decided against this because I want to remain true the features that @iksaku has written into *EssentialsPE*. Failing that, I decided to add support for *EssentialsPE* in *SimpleWarp*. 

### Enabling
Their are likely a bunch of ways I could have gone about this, but I settled on a simple and hacky solution. First, the `plugin.yml` is modified so that *SimpleWarp* always loads after *EssentialsPE* (softdepend).

* If a user has enabled `essentials-support` in their config and have *EssentialsPE* installed.
	* Find and save the *EssentialsPE* commands.
    * Unregister them too.
	* Create alternate command objects for **/warp** and **/delwarp**, they will act as routers between the two plugins. They extend their coresponding *SimpleWarp* command. 
    * Register other commands like normal.
* If not
	* Register commands like normal and hope for the best.
    
### Running commands
When a command like **/addwarp** (*SimpleWarp*) or **/setwarp** (*EssentialsPE*) is run, nothing special happens as their is no conflict. However, when we have **/warp** or **/delwarp** their needs to be some magic. They both behave the same, so let's look at **/warp**.

* Player runs **/warp**
* Did they specify a warp name?
	* Yes
    	* Is this a *SimpleWarp* warp?
        	* Yes
            	* Warp them to the *SimpleWarp* warp.
            * No
            	* Is this an *EssentialsPE* warp? (see below)
                	* Yes
                    	* Warp them to the *EssentialsPE* warp.
                    * No
                    	* Tell them the warp doesn't exist.
    * No
    	* Tell them to try again.
* Done.

### Explictly specifying EssentialsPE warps
If both plugins have a warp called **1**, we will always pick the *SimpleWarp* one. This isn't quite fair to *EssentialsPE*. To combat this, *SimpleWarp* will warn the user that there is an *EssentialsPE* warp available. If they want to use it, they can explicitly ask. If it is not available, the command will fail. This is done by prefixing the warp name with **ess:**. If a user attempts to create a *SimpleWarp*  warp that starts with **ess:**, they will get a warning message.

### Conclusion
Finally *EssentialsPE* users will be able to try out all the awesome functionality in *SimpleWarp* without sacraficing their existing warp plugin. I think that plugins should strive to work together, but what I have done here is a bit extreme. When making a plugin it is useful to look around for other plugins that use the same command(s), namespace(s) or plugin name as you. 
