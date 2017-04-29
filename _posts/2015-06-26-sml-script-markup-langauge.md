---
title: 'SML: Script Markup Language'
layout: post
permalink: sml-script-markup-langauge
published: false
---
*SML* is a markup language I have developed for an upcoming web renderer for Shakespeare's plays. The language is a distinct and compliant subset of *XML*. *SML* allows for natural representation of a play script. It is readable by humans and machines, the perfect combo. 


## Anatomy 
* We start off an *SML* file with an *XML* declaration. This lets the *XML* parser knoew what it is handling.

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

* Next we open a `<play>` tag. Everything we need will be contained inside this tag.
* At the top of the play we have a header which carries metadata for the compiler. This metadata includes the title, a short name (for storage), tags and more. The metadata is very loosely enforced, only the title parameter is required by SML. Specific renderers may require additional metadata.

```xml
<head>
        <title>Comedy of Errors</title>
        <note>The Comedy of Errors tells the story of two sets of identical twins that were accidentally separated at birth.</note>
        <shortName>comedy-of-errors</shortName>
        <tag>comedy</tag>
        <tag>short</tag>
        <tag>identity</tag>
        <tag>twins</tag>
        <tag>early</tag>
        <author>Noah Heyl</author>
        <image>https://upload.wikimedia.org/wikipedia/commons/thumb/f/f9/Comedy_of_Errors-Dromios.pdf/page1-639px-Comedy_of_Errors-Dromios.pdf.jpg</image>
    </head>
```

* Now we open the ` <text>` object. This object contains the body of the play.


