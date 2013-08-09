hack-karma-https
================

Hacking karma to launch https webserver

### Why?
As of v9.x, Karma supports running the application under test over https. The ```proxyValidateSSL: false``` configuration option allows the browser ssl warning to be ignored when a self signed certificate is used, without any manual intervention.

However, the web server started by Karma itself is still running over http. So if the server is configured to send secure cookies (eg: session, csrf, etc), then they will not be sent to the browser running Karma. 
Therefore any end-to-end tests that modify state (http POST, PUT, etc.) or otherwise depend on cookies will fail.

The code here is a copy of the karma code, plus a hack/proof-of-concept to see if Karma could be modified to run under https. A proper solution would involve modifying and reading from the config to support this as an option.

### Files Modified

* karma/lib/launcher.js

	```
	// var url = 'http://' + hostname + ':' + port + urlRoot;
	var url = 'https://' + hostname + ':' + port + urlRoot;
	```

* karma/lib/server.js

	Not sure if its really required to modify io.listen

	```
	// Added to read and resolve cert and key files
	var fs = require('fs');
	var path = require('path');

	// Added secure, key, and cert
	var server = io.listen(webServer, {
    	secure: true,
    	key: fs.readFileSync(path.resolve(__dirname, '../../../server/cert/key.pem')),
    	cert: fs.readFileSync(path.resolve(__dirname, '../../../server/cert/cert.pem')),
    	logger: logger.create('socket.io', constant.LOG_ERROR),
    	resource: config.urlRoot + 'socket.io',
    	transports: config.transports
  	});
  	```

* karma/lib/web-server.js

	```
	var https = require('https');

	// Proper soln would have the key and cert file locations specified in config
	var options = {
    	key: fs.readFileSync(path.resolve(__dirname, '../../../server/cert/key.pem')),
    	cert: fs.readFileSync(path.resolve(__dirname, '../../../server/cert/cert.pem'))
  	};
  
  	// return http.createServer(handler);
  	return https.createServer(options, handler);

* karma/static/karma.js

	```
	//var socket = io.connect('http://' + location.host, {
	var socket = io.connect('https://' + location.host, {

* karma-chrome-launcher/index.js

	For Chrome, prevent the ssl warning by adding command line switches ```ignore-certificate-errors``` and ```ignore-urlfetcher-cert-requests```. Otherwise user would have to manually intervene to ignore the error when the Chrome browser is launched. It's possible that only the first one is required. The equivalent controls would have to be determined for all other browsers to be used for e2e tests.

	```
	return [
      '--ignore-certificate-errors', 
      '--ignore-urlfetcher-cert-requests',
      '--user-data-dir=' + this._tempDir,
      '--no-default-browser-check',
      '--no-first-run',
      '--disable-default-apps',
      '--start-maximized'
	```