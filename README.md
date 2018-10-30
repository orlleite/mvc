
# kldit::mvc

## Install
```bash
npm i --save @kldit/mvc
```

## Getting started
Create a .env with the params:
```bash
# Name in case of https: https://HOSTNAMES/PATH_NAMES
# Leave empty if you're not going to use.
# it will run only in 443 port
HOSTNAMES=
PATH_NAMES=

# Name in case of http: http://HOSTNAME/PATH_NAME
# You have to specify a port
HOSTNAME=localhost
PATH_NAME=
PORT=3000

# Environment (development|production)
ENV=development
# Make sure the path exists
LOG_PATH=log
ENCODE=utf8

# If you want to use cluster (multiple instances of your application)
CLUSTER=false
```

You can start your first app using the code below (`app.js`):
```javascript
const mvc = require('@kldit/mvc');

class MainApplication extends mvc.Application
{

}

mvc.server.setApplication( new MainApplication() );

```
### Run it
```bash
$ node app.js
> Setup  9.95 MB
```

### Test it
```bash
$ curl localhost:3000
> Kldit::MVC
```

## Your first service
Create a folder and name it as you want. I'll call `MyService`. Create a struct as below:
```bash
MyService
> controller
> model
> view
```

Inside the controller folder, create a file called `Home.js`
```javascript
const mvc = require.main.require('@Kldit/mvc');

module.exports = class Home extends mvc.BaseController
{
	index(ctx) {
		ctx.body = 'Hello World!';
	}
}
```

Change your class `MainApplication` in the `app.js`:
```javascript
class MainApplication extends mvc.Application
{
	init( koa )
	{
		super.init( koa );
		// Put the name you want and point to the root service folder.
		this.loadService( 'my-service', './MyService' );
	}
}
```

### Test it
```bash
$ curl localhost:3000/my-service
> Hello World!
```
[More about controllers](extra/CONTROLLER.md)

### Default Service
Change `app.js`
```javascript
this.setDefaultService( 'my-service', './MyService' );
```
And test it. Now your app is at the root path.
```bash
$ curl localhost:3000
> Hello World!
```

## Your first Model
You have to choose a database application. I'll use MySQL by installing `ext-mysql` extension.
```bash
$ npm install ext-mysql
```
Add to the method `MainApplication::init` the code:
```javascript
// Require
const MySQL = require('ext-mysql');

MySQL.CREATE_POOL({
	host: process.env.DB_HOSTNAME,
	port: process.env.DB_PORT ? process.env.DB_PORT : 3306,
	user: process.env.DB_USER,
	password: process.env.DB_PASSWORD,
	database: process.env.DB_DATABASE,
	waitForConnections: true,
	connectionLimit: 5,
	queueLimit: 0
});

// It is good to have a log.
MySQL.LOGGER = mvc.log.databaseLog;
``` 

Add this two methods to `MainApplication` class.
```javascript
async getDbConnection()
{
	const conn = new MySQL();
	return await conn.init();
}

releaseDbConnection( conn )
{
	conn.release();
}
```
Set properly all those .env values (`DB_HOSTNAME`, `DB_USER`, `DB_PASSWORD`, `DB_DATABASE`). Now, you are ready to go. Let's create our first model indeed.
```javascript
// ./model/HomeModel.js
const { BaseModel } = require.main.require('@kldit/mvc');
module.exports = class HomeModel extends BaseModel
{
	async version(ctx)
	{
		const conn = await ctx.db();
		let result = null;
		try
		{
			[result, fields] = await conn.execute(
				`SELECT version(), ?`,
				["my value"]);
		}
		catch (err)
		{
			console.log(err);
		}
		
		return result;
	}
}
```

And the `Home.js`
```javascript
const mvc = require.main.require('@Kldit/mvc');

module.exports = class Home extends mvc.BaseController
{
	init()
	{
		// Load the model you want
		this.loadModel('Home');
	}

    async index(ctx)
    {
	    // Call the method and show it in the body of the response.
    	ctx.body = await this.model.home.version(ctx);
    }
}
```

### Test it
```bash
$ curl localhost:3000
> [{"version()":"5.7.23","my value":"my value"}]
```
[More about models](extra/MODEL.md)

## Your first view
**kldit::mvc** supports mustache, ejs and vanilla js, although, it does not require any and you have to install as you wish.
```javascript
// HomeView.js (js version)
module.exports = function( data )
{
	let version = data.version[0];
	return `Version is: ${version['version()']} and val: ${version['my value']}`
}
```
Change `index(ctx)` to:
```javascript
ctx.body = this.renderView( 'Home', { version:await this.model.home.version(ctx) } );
```

### Test it
```bash
$ curl localhost:3000
Version is: 5.7.23 and val: my value
```
### EJS Version
You have to install ejs package in order to use it. `npm i ejs`
```javascript
// HomeView.ejs (ejs version)
Version is: <%= version[0]['version()'] %> and val: <%= version[0]['my value'] %>
```
[More about views](extra/VIEW.md)