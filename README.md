hack-karma-https
================

Hacking karma to launch https webserver

* Why?
As of v9.x, Karma supports running the application under test over https.
The ```proxyValidateSSL: false``` configuration option allows the browser ssl warning to be ignored when a self signed certificate is used.

However, the web server started by Karma itself is still running over http. Therefore if the server is configured to send secure cookies (eg: session, csrf, etc), then they will not be sent to the browser running Karma. 
Therefore any end-to-end tests that modify state (http POST, PUT, etc.) or otherwise depend on cookies will fail
because server does not receive the expected cookie.

The code here is a hack/proof-of-concept to see if Karma could be modified to run under https. A proper solution would involve modifying and reading from the config to support this as an option.

* Files Modified

	* karma/lib/launcher.js
	```
	// var url = 'http://' + hostname + ':' + port + urlRoot;
	var url = 'https://' + hostname + ':' + port + urlRoot;
	```
