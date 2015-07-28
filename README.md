## hapi-sequelized - a hapi plugin for the sequelize orm

[![Build Status](https://travis-ci.org/danecando/hapi-sequelized.svg)](https://travis-ci.org/danecando/hapi-sequelized)
[![npm](https://img.shields.io/npm/dm/localeval.svg)](https://www.npmjs.com/package/hapi-sequelized)

* http://hapijs.com/
* http://sequelizejs.com/

### Installation

`npm install hapi-sequelized --save`

### Loading the plugin

See http://hapijs.com/tutorials/plugins

```javascript
server.register(
    [
        {
            register: require('hapi-sequelized'),
            options: {
                database: 'dbName',
                user: 'root',
                pass: 'root',
                dialect: 'mysql',
                port: 8889,
                models: 'models/**/*.js',
                sequelize: {
                    define: {
                        underscoredAll: true
                    }
                }
            }
        },
    ], function(err) {
        if (err) {
            console.error('failed to load plugin');
        }
    }
);
```

### Available Options

```javascript
options: {
    database: 'dbName', // database name
    user: 'root',       // db username
    pass: 'root',       // db password
    dialect: 'mysql',   // database type
    host: 'localhost',  // db host
    port: 8889,         // database port #
    models: ['models/**/*.js', 'other/models/*.js'],   // glob or an array of globs to directories containing your sequelize models
    logging: false      // sql query logging
    sequelize: {       // Options object passed to the Sequelize constructor http://docs.sequelizejs.com/en/latest/api/sequelize/#new-sequelizedatabase-usernamenull-passwordnull-options
        define: {
            timestamps: false
        }
    }
}
```

### Usage

Your models are attached to the sequelize instance and will be available
throughout your application via server.plugins (which is also available
through the request object ie: request.server.plugins)

```javascript
var models = request.server.plugins['hapi-sequelized'].db.sequelize.models;

models.User.create({
    email: 'some@email.com',
    password: 'password123'
});
```

### Model Definitions

Exports a function that returns a model definition 
http://docs.sequelizejs.com/en/latest/docs/models-definition/

```javascript
module.exports = function(sequelize, DataTypes) {
    var StoreOptions = sequelize.define(
        'StoreOptions',
        {
            optionName: {
                type: DataTypes.STRING,
                unique: true,
                allowNull: false
            },
            optionValue: {
                type: DataTypes.TEXT
            }
        },
        {
            tableName: 'store_config',
            timestamps: false
        }
    );

    return StoreOptions;
};
```

### Syncing Models

Sync'ing needs to be done after the plugin has loaded (typically in the
callback function where it was registered). A regular sync will 
 `CREATE TABLE IF NOT EXISTS` , while a sync with the 
`{ force: true }` option passed will drop all of your tables first. 

```javascript
var db = server.plugins['hapi-sequelized'].db;
db.sequelize.sync().then(function() {
  console.log('models synced');
});
```

### Security

This plugin has been updated to use version 3+ of Sequelize which adds many 
preventative measures to help users avoid the vulnerabilities explained
in the article below. Please give the article a good read to ensure that you're
using Sequelize safely!

https://securityblog.redhat.com/2015/05/20/json-homoiconicity-and-database-access/

Also make sure to take a look at the changelog if you're upgrading from a pre 3+ 
version. There are several breaking changes that will need to be addressed.

https://github.com/sequelize/sequelize/blob/master/changelog.md
