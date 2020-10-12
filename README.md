# Interacting with the Webex XML API from NodeJS

In this tutorial we are going to spend some time interacting with the Webex XML API from Node JS. If you are studying for your [DevWBX certification](https://www.cisco.com/c/en/us/training-events/training-certifications/exams/current-list/devwbx-300-920.html), this tutorial may help you along the way. 

## Pre-Work

1. This lab assumes that you already have some familiarity with the [Webex XML API](https://developer.cisco.com/docs/webex-meetings/#!xml-api). Before coding against the API, you can go through [this learning lab](https://developer.cisco.com/learning/lab/collab-webex/step/1) to get some exposure. 
2. Have node.js installed, and have some experience with it. Need an intro to Node.js? See [nodeschool.io](nodeschoo.io). 
3. Have a webex site to test against. While any site can be used, the easiest to test and learn on is one where SSO is NOT enabled. The below assumes you will be using a site with a simple username & password (others can be used, you'll just have to do a little more leg work). If you don't have a site you can always use the [developer sandbox provided on DevNet's page](https://devnetsandbox.cisco.com/RM/Topology?c=1c4570f4-4199-41e4-b677-fb3a6346f345). 

## Create a Meeting and Parse the Results

### 1. Meeting Details XML
The most basic function you might want to invoke with Webex is to create a meeting. This can be done with the [CreateMeeting](https://developer.cisco.com/docs/webex-xml-api-reference-guide/#!createmeeting) command. Assuming you have some exposure to the Webex XML API you know that there are 2 key items you will need in the XML: 

1. The Security Context details. In the most simple cases this will be the sitename, your webexid and your password. For the example below, the username is 'steven', the password is 'testing123!' and the site is 'my-sample-site.webex.com'. 
2. Some basic details of your meeting (e.g. date, time, title, password, etc). In the example below the meeting password is 'mp-123!', the title is 'Sample Meeting' and the start date & time are '12/02/2020 @2:15pm'. 

The above details mapped into the XML schema for creating a meeting would look like the following XML: 

```
<?xml version="1.0" encoding="UTF-8"?> 
<serv:message xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">	
<header>		
  <securityContext> 
    <webExID>steven</webExID> 
    <password>testing123!</password> 
    <siteName>my-sample-site</siteName> 
  </securityContext>	
</header>	
<body>		
	<bodyContent xsi:type="java:com.webex.service.binding.meeting.CreateMeeting">
		<accessControl>				
			<meetingPassword>mp-123!</meetingPassword>			
		</accessControl>			
		<metaData>				
			<confName>Sample Meeting</confName>			
		</metaData>	
		<schedule> 				
			<startDate>12/02/2020 14:15:00</startDate>				
		</schedule>		
	</bodyContent>	
</body>
</serv:message>
```

### 2. Install 2 Useful Packages

Now that we have our XML is time to write our NodeJS code. There are many ways to perform an [HTTP POST request in Node.js](https://nodejs.dev/learn/making-http-requests-with-nodejs), but one of the simplest ways to use the [Axios library](https://github.com/axios/axios). Axios is a Promise based HTTP client for NodeJS. So after creating a directory to work in, install axios. 

```npm install axios```

Not only is the data that we are sending formatted in XML, but so will be the response. Manipulation of that data will be easier if we can parse it into a native object/JSON. Again there are a lot of ways to parse that data but for this example we'll use the ['xml2js' package](https://www.npmjs.com/package/xml2js). 

```npm install xml2js```

### 3. Write the Base Code

The beginning of our code is simple. Import the 2 libraries we are going to use and define the xml.
```
const axios = require('axios');
const parseString = require('xml2js').parseString;

let xml = `
<?xml version="1.0" encoding="UTF-8"?> 
<serv:message xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">	
<header>		
  <securityContext> 
    <webExID>steven</webExID> 
    <password>testing123!</password> 
    <siteName>my-sample-site</siteName> 
  </securityContext>	
</header>	
<body>		
	<bodyContent xsi:type="java:com.webex.service.binding.meeting.CreateMeeting">
		<accessControl>				
			<meetingPassword>mp-123!</meetingPassword>			
		</accessControl>			
		<metaData>				
			<confName>Sample Meeting</confName>			
		</metaData>	
		<schedule> 				
			<startDate>12/02/2020 14:15:00</startDate>				
		</schedule>		
	</bodyContent>	
</body>
</serv:message>
`
```
Now all that needs to be added is the command to send the request (via POST) to the webex api and show us the results. In this case we'll log response.data which is the response that was provided by the server, but the response schema from Axios contains a lot more. 

```
axios.post('https://api.webex.com/WBXService/XMLService', xml)
  .then(function (response) {
      console.log(response.data);
  })
  .catch(function (error) {
    console.log(error);
  });
```

Copy the above two snippets of code into your favorite development friendly editor (replacing the security context with your own) and run the file with node and you should get back a blob of xml like this (note you will get a raw output, I've done some formatting to the response to make it easier to read): 

```
<?xml version="1.0" encoding="UTF-8"?>
<serv:message xmlns:serv="http://www.webex.com/schemas/2002/06/service" xmlns:com="http://www.webex.com/schemas/2002/06/common" xmlns:meet="http://www.webex.com/schemas/2002/06/service/meeting" xmlns:att="http://www.webex.com/schemas/2002/06/service/attendee">
    <serv:header>
        <serv:response>
            <serv:result>SUCCESS</serv:result>
            <serv:gsbStatus>PRIMARY</serv:gsbStatus>
        </serv:response>
    </serv:header>
    <serv:body>
        <serv:bodyContent xsi:type="meet:createMeetingResponse" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <meet:meetingkey>17260117096</meet:meetingkey>
            <meet:meetingUUID>c0aeed23bc2ff2c06dc685d6a4e717e8e</meet:meetingUUID>
            <meet:meetingPassword>mp-123!</meet:meetingPassword>
            <meet:iCalendarURL>
                <serv:host>https://my-sample-site.webex.com/my-sample-site/j.php?MTID=m1531589fcf05ef0936924e81eb663c442</serv:host>
                <serv:attendee>https://my-sample-site.webex.com/my-sample-site/j.php?MTID=m7ed4f21aad1b455398f3e4fcc849d1814</serv:attendee>
            </meet:iCalendarURL>
            <meet:guestToken>562917404d04dcc0433a7c5fd526e7542</meet:guestToken>
        </serv:bodyContent>
    </serv:body>
</serv:message>
```

Note: Each time you run this command you will be creating a new meeting. Depending on your environment, you may want to clean those up (or see later on in this lab). 

### 4. Parse the Response

Pulling a piece of the data out of the above response will be a lot easier if we parse the data into an object. 

Remove or comment out the existing console.log command from your code and replace it with the following to parse the data prior to logging it: 
```
    parseString(response.data, function (err, result) {
        console.log(result)
    });
```
After running the output of which should look like this: 
```
{
  'serv:message': {
    '$': {
      'xmlns:serv': 'http://www.webex.com/schemas/2002/06/service',
      'xmlns:com': 'http://www.webex.com/schemas/2002/06/common',
      'xmlns:meet': 'http://www.webex.com/schemas/2002/06/service/meeting',
      'xmlns:att': 'http://www.webex.com/schemas/2002/06/service/attendee'
    },
    'serv:header': [ [Object] ],
    'serv:body': [ [Object] ]
  }
}
```
While now a native object, there is some extra namespace prefixes that would be nice to remove (e.g. it will be easier to work with if we convert 'serv:body' to 'body'). xml2js has a processor that are will do that for you, 'stripPrefix'. Add one more require module for the processor towards the beginning of your code. 

```
const stripNS = require('xml2js').processors.stripPrefix;
```

Now you can adjust the parse call to include stripping the namespace. 

```
    parseString(response.data, { tagNameProcessors: [stripNS] }, function (err, result) {
        console.log(result);
    });
```

After running you should get the following XML with out the namespace: 

```
{
  message: {
    '$': {
      'xmlns:serv': 'http://www.webex.com/schemas/2002/06/service',
      'xmlns:com': 'http://www.webex.com/schemas/2002/06/common',
      'xmlns:meet': 'http://www.webex.com/schemas/2002/06/service/meeting',
      'xmlns:att': 'http://www.webex.com/schemas/2002/06/service/attendee'
    },
    header: [ [Object] ],
    body: [ [Object] ]
  }
}
```
Most of the data you'd want work with is inside the body (Note some of objects are obscured above because Node uses util.inspect to convert the object into strings and that function stops after depth=2 which is a bit low this & most XML). Now that the XML is in a object and you've stripped off the namespace, you can easily access any of the data inside using simple dot notation. You can step by step dive deeper into the body object, but I can save you some time and simply have you now update your log command to:

```
console.log(result.message.body[0].bodyContent[0]);
```

That should give you and output similar to the following: 
```
{
  '$': {
    'xsi:type': 'meet:createMeetingResponse',
    'xmlns:xsi': 'http://www.w3.org/2001/XMLSchema-instance'
  },
  meetingkey: [ '17268753316' ],
  meetingUUID: [ 'ac94edca8da004f037470c7f36148ce7b' ],
  meetingPassword: [ 'mp-123!' ],
  iCalendarURL: [ { host: [Array], attendee: [Array] } ],
  guestToken: [ 'da6423b7daf5b9491cc81a813b8c2228e' ]
}

```
Great, you've sucessfully completed this section. Feel free to adjust the code above or move on to one of the other tutorials below. You can also challange yourself to go futhre. Maybe add some error handling. How about writing some code that prompts the user for some basic webex info then creates the meeting based on their preferances? Or maybe bulk importing meeting info from a csv file and creating multiple meetings. 

## Bulk Delete Meetings


## Listing Recordings

## License

Provided under Cisco Sample Code License, for details see [LICENSE](./LICENSE.md)

## Code of Conduct

Our code of conduct is available [here](./CODE_OF_CONDUCT.md)

## Contributing

See our contributing guidelines [here](./CONTRIBUTING.md)

