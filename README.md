node-dom
===

Javascript fast DOM generation

## Purpose:

Optimization to generate the W3C DOM for a web page as fast as possible.

It is  a mix of W3C / WHATWG and specific adaptations for server side and to increase speed, following more what browsers are doing than W3C (ie not being really internally compliant but compliant externally at the end)

It does implement what is mostly used and returns an acceptable result for what is rarely used.

It does handle scripts, styles, links, images, xhr and cookies.

It does not handle virtual rendering, stylesheets (but can load links) and events (except onload), but does not crash if not implemented features are used.

## Install :

    npm install node-dom

or

    git clone http://github.com/Nais/node-dom.git
    cd node-dom
    npm link .
	
Complementary modules :
	 [Nais/node-cookies](https://github.com/Nais/node-cookies) 
	 [driverdan/node-XMLHttpRequest](https://github.com/driverdan/node-XMLHttpRequest)
	 [rsms/node-imagemagick](https://github.com/rsms/node-imagemagick)

## Simple case :

	simple.js :
	
````
	var request = require('request'),
		dom = require('node-dom').dom,
		fs = require('fs'),   
		URL = require('url');
		
	var	args = require('tav').set({
					url:{
					note:'URL of the page to parse'
					}
				},'node-dom for node.js',true);

	var url = URL.parse(args.url);

	var req = {uri:url.href};

	request(req,function (error, response, page) {
	
		if (!error && response.statusCode == 200) {
			
			var options =	{	url:url,
								features: {
											FetchExternalResources  : {script:'', img:'', input:'', link:''},
											ProcessExternalResources: {script:'',img:'',link:'',input:''},
											removeScript: true //Remove scripts for innerHTML and outerHTML output
								}
			};
			
			window=dom(page,null,options); //global
			
			document=window.document; //global
			
			document.onload=function() {
			//Warning : you are not in the window context here (ie you can not access window's global var as global variables directly)
			//Context are explained here https://github.com/joyent/node/issues/1674
					
				fs.writeFile('./outer.html', document.html.outerHTML, function (err) {});
				//check the result if you want in created outer.html file
				//if you want to test it in a browser, don't forget to put the base tag after <head> with correct href	
			};
		};
	});
````

#### Example : node simple.js --url=http://www.example.com

## Medium case :

	medium.js :
	
````
	var request = require('request'),
		dom = require('node-dom').dom,
		fs = require('fs'),   
		URL = require('url');
		
	var	args = require('tav').set({
					url:{
						note:'URL of the page to parse'
					},
					post:{
						value:'',
						note:'Post parameters'
					}
				},'dom-node for node.js',true);

	var url = URL.parse(args.url);

	var req = {uri:url.href};

	if (args.post) {
		req['method'] = 'post';
		req['body'] = args.post;
	};

	request(req,function (error, response, page) {
	
		if (!error && response.statusCode == 200) {
			
			var options =	{	url:url,
								features: {
											FetchExternalResources  : {script:'', img:'', input:'', link:''},
											ProcessExternalResources: {script:'',img:'',link:'',input:''},
											removeScript: true, //Remove scripts for innerHTML and outerHTML output
								}
			};
			
			window=dom(page,null,options); //global
			
			document=window.document; //global
			
			document.onload=function() {
			//Warning : you are not in the window context here (ie you can not access window global var as global variables directly)
			//Contexts are explained here https://github.com/joyent/node/issues/1674
			//Add your code and do what you have to do with the DOM
				
				fs.writeFile('./outer.html', document.html.outerHTML, function (err) {});
				//check the result if you want in created outer.html file
				//if you want to test it in a browser, don't forget to put the base tag after <head> with correct href
				
				//Example, add your class
				//var MyClass = new require('MyClass').MyClass();
				//window.$=window.jQuery=require('jQuery').jQuery (invention here, let's say you want to use jQuery)
				
				//Do what you have to do in the DOM
				//MyClass.do_some_stuff_in_the_page();
				//Use jQuery
		
			};
		};
	});
````

#### Example : node medium.js --url=http://www.example.com

## Complete case :

	complete.js :
	
````
	var request = require('request'),
		dom = require('node-dom').dom,
		fs = require('fs'),   
		URL = require('url'),
		cookies = require('node-cookies'),
		cookieJar = cookies.cookieJar;
		
	var	args = require('tav').set({
					url:{
						note:'URL of the page to parse'
					},
					cookies:{
						value:'/tmp/cookies.txt',
						note:'Where to store cookies'
					},
					post:{
						value:'',
						note:'Post parameters'
					}
				},'dom-node for node.js',true);

	var url = URL.parse(args.url);

	var req = {uri:url.href};

	//check if the cookie file exists - create it if not
	try {fs.statSync(args.cookies);} catch(ee) {fs.writeFile(args.cookies,'',function(err) {});}
	var fullJar = new cookieJar(args.cookies);
	var jar = fullJar.extractCookiesForUrl(url.href);
	var cookiestr = jar.cookiesForUrl();
	req['headers'] = { Cookies:cookiestr };

	if (args.post!=''){
		req['method'] = 'post';
		req['body'] = args.post;
	};

	request(req,function (error, response, page) {

			if (!error && response.statusCode == 200) {
			
				// update cookies sent by the server

				if (response.headers['set-cookie']){
					jar.setCookiesForUrl(response.headers['set-cookie']);
				};
				
				var options =	{	url:url,							
									features: {
												FetchExternalResources  : {script:'', img:'', input:'', link:''},
												ProcessExternalResources: {script:'',img:'',link:'',input:''},
												removeScript: true, //Remove scripts for innerHTML and outerHTML output
									},
									cookie: jar
				};
				
				window=dom(page,null,options); //global
				
				document=window.document; //global
				
				document.onload=function() {
				//Warning : you are not in the window context here (ie you can not access window global var as global variables directly)
				//Contexts are explained here https://github.com/joyent/node/issues/1674
				//Add your code and do what you have to do with the DOM
					
					fs.writeFile('./outer.html', document.html.outerHTML, function (err) {});
					//check the result if you want in created outer.html file
					//if you want to test it in a browser, don't forget to put the base tag after <head> with correct href
					
					//Example, add your class
					//var MyClass = new require('MyClass').MyClass();
					//window.$=window.jQuery=require('jQuery').jQuery (invention here, let's say you want to use jQuery)
					
					//Do what you have to do in the DOM
					//MyClass.do_some_stuff_in_the_page();
					//Use jQuery
					
					//save cookies
					cookies.mergeJar(fullJar,document._cookie).saveToFile(args.cookies);
				};
			};
	});
````
#### Example : node complete.js --url=http://www.example.com --cookies=./cookies.txt

## Options and arguments :

dom(page,level,options) : level is not used

	options =	{	
								url:url,							
								features: {
											FetchExternalResources  : {script:'', img:'', input:'', link:''},
											ProcessExternalResources: {script:'',img:'',link:'',input:''},
											removeScript: true, //Remove scripts for innerHTML and outerHTML output
								},
								cookie: jar
	};

Remove from features if you don't want to use it :

- script : load and execute scripts
- link : load links
- img/input : load images and background images, retrieve the size, assign width/height to the objects
- removeScript : it's generally useless to keep scripts in inner/outerHTML output, so default should be true. Whatever is the setting, scripts present in innerHTML or outerHTML will not execute as it is specified in standards.
		

## Resources sequencing :

Unlike browsers scripts execution is a little defer here, so it can not block the construction of the DOM, scripts are loaded and queued (inline and outside), then executed in the right order on document.close() (which queue document) and onload is fired.

Since scripts can add others, queue can extend after document (then their execution can continue after readyState "complete").

When queue is empty, scripts are executed right away and onload fired.

Scripts that fail will be re-executed after a certain delay, see comments in the code why it can happen

Same happens for links.

Images are loaded asynchronously, same image is loaded just once, then width/height are set to objects that are using this image.
	
## Tests :

	See test/tests.txt
	
Tested on google, yahoo, msn, usual js frameworks (jQuery, YUI, prototype, mootools,...) and unbelievable usual web sites.
