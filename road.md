# 死月自用的项目基础之撸

米娜桑请轻拍。(✪ω✪)

# Version 1 - SevenzJS

https://github.com/XadillaX/SevenzJS

在不会 Node.js 的情况下写的失败的框架。

## Directories

https://github.com/XadillaX/SevenzJS

```
.
├── README.md
├── actions
│   └── index.js
├── index.js
├── package.json
├── settings.js
├── sevenz
│   ├── SevenzJS.js
│   ├── sAction.js
│   ├── sMongoSync.js
│   ├── sMysql.js
│   ├── sRequest.js
│   ├── sRouter.js
│   └── utils
│       ├── json2.js
│       ├── log.js
│       └── mime.js
└── statics
```

## How to Implement a Router

```javascript
exports.action = function(action) {
    var self = { };
    var helper = action;

    /**
     * empty路由
     */
    self["_empty"] = function() {
        helper.write("你访问了未定义的路由：" + helper.pathinfo[1]);
    }

    /**
     * demo首页
     */
    self["index"] = function() {
        helper.write("Hello SevenzJS!");
    }

    /**
     * 查看tables
     */
    self["showtables"] = function() {
        if(!helper.mysql.connect()) return;

        helper.write("<pre>");
        var tables = helper.mysql.query("SHOW TABLES;");
        for(var i = 0; i < tables.length; i++)
        {
            helper.write(JSON.stringify(tables[i]));
            helper.write("\n");
        }
        helper.write("</pre>");
    }

    return self;
};
```

## Who Reads the Routers

```javascript
exports.route = function(pathinfo, postData, request, response) {
    /** 去除后缀 */
    if(pathinfo.length > 0)
    {
        var lastParam = pathinfo[pathinfo.length - 1];
        var suffCount = this.conf["server"]["suffix"].length;
        var suffs = this.conf["server"]["suffix"];

        for(var i = 0; i < suffCount; i++)
        {
            if(lastParam.substr(lastParam.length - suffs[i].length, suffs[i].length).toLowerCase() == suffs[i].toLowerCase())
            {
                lastParam = lastParam.substring(0, lastParam.length - suffs[i].length);
                pathinfo[pathinfo.length - 1] = lastParam;
                break;
            }
        }
    }

    /** 解析Action以及其func */
    var act = pathinfo[0];
    var func = pathinfo[1];
    if(act === undefined || act === "") act = "index";
    if(func === undefined || func === "") func = "index";

    this._view(act.toLowerCase(), func.toLowerCase(), request, response, pathinfo, postData);
}
```

### this._view

```javascript
exports._view = function(cls, func, request, response, pathinfo, postData) {
    this.logger.info("Routing to '" + cls + "' - '" + func + "'...");

    /** 实例化action对象 */
    var rAction = require("./sAction");
    var action = new rAction.sAction(request, response, pathinfo, this.conf, this.logger, postData);

    /** 判断Action合法性 */
    if(this.isValidClassString(cls))
    {
        var filename = "../actions/" + cls;
        var actionObj = null;

        /** 载入Action模块 */
        try {
            var actionObj = require(filename);
        }
        catch(e) {
            this.logger.error(e);
            if(this.conf["server"]["debug"] === true) action.set404(e);
            else action.set404();
            return;
        }

        /** Action错误 */
        if(actionObj === undefined || typeof actionObj.action !== "function")
        {
            this.logger.error("Broken action object '" + cls + "'.");

            if(this.conf["server"]["debug"] === true) action.set404("Broken action object '" + cls + "'.");
            else action.set404();
            return;
        }

        /** Action路由错误 */
        var routeTable = actionObj.action(action);
        if(routeTable === undefined || typeof routeTable !== "object")
        {
            this.logger.error("Broken action router '" + cls + "'.");

            if(this.conf["server"]["debug"] === true) action.set404("Broken action router '" + cls + "'.");
            else action.set404();
            return;
        }

        /** Empty */
        if(func.toLowerCase() === "_empty")
        {
            action.set404();
        }

        /** 操作名错误 */
        if(routeTable[func] === undefined || typeof routeTable[func] !== "function")
        {
            /** 但是如果有empty的话 */
            if(routeTable["_empty"] !== undefined && typeof routeTable["_empty"] === "function")
            {
                func = "_empty";
            }
            /** 连empty都没的话 */
            else
            {
                this.logger.error("No suct action function '" + func + "' @ '" + cls + "'.");

                if(this.conf["server"]["debug"] === true) action.set404("No suct action function '" + func + "' @ '" + cls + "'.");
                else action.set404();
                return;
            }
        }

        /** 执行函数 */
        var Fiber = require('fibers');

        Fiber(function(){
            routeTable[func]();
            action._render();
        }).run();
    }
    else
    {
        this.logger.error("Error action name '" + cls + "'");
        if(this.conf["server"]["debug"] === true) action.set404("Error action name '" + cls + "'");
        else action.set404();
    }
}
```

## Features && Shortages

### Features

+ 直接往 `actions` 添加文件，按格式添加函数就能导航到相应路由。
+ 同步模式（使用了 Fibers）。

### Shortages

+ 同步模式（根本没必要，实现了也没用）。
+ 一次请求一次连接数据库什么的。（当时受 PHP 毒害太深）
+ 这代码写的什么鸟玩意？
+ 这代码写的什么鸟玩意？ x 2
+ 这代码写的什么鸟玩意？ x 3
+ 这代码写的什么鸟玩意这代码写的什么鸟玩意... ***N Combos!***

# You can't use a car as a boat. If you want a boat, use a boat.

http://xcoder.in/2013/03/29/nodejs-mongodb-sync/

# Version 2 - XPlan

> Nov 17, 2013 - Jan 8, 2014

https://github.com/XadillaX/xplan-backend

## Directories

```
.
├── README.md
├── action
│   ├── backend
│   │   ├── classAction.js
│   │   ├── feedbackAction.js
│   │   ├── funnyAction.js
│   │   ├── indexAction.js
│   │   ├── newsAction.js
│   │   ├── userAction.js
│   │   └── waterpitAction.js
│   └── resource
│       ├── avatarAction.js
│       └── imageAction.js
├── attache_image
│   └── deleteme
├── avatar
│   ├── Z134325055
│   ├── Z134325059
│   └── deleteme
├── config
│   └── config.js
├── data
├── logs
├── model
│   ├── base
│   │   └── model.js
│   ├── classModel.js
│   ├── dummySessionModel.js
│   ├── imageModel.js
│   ├── newsModel.js
│   ├── shopCommentModel.js
│   ├── shopModel.js
│   ├── userModel.js
│   ├── waterpitModel.js
│   └── waterpitReplyModel.js
├── package.json
├── router
│   ├── index.js
│   ├── main-rule
│   │   ├── class.js
│   │   ├── feedback.js
│   │   ├── funny.js
│   │   ├── index.js
│   │   ├── news.js
│   │   ├── user.js
│   │   └── waterpit.js
│   └── xrs-rule
│       ├── avatar.js
│       └── image.js
├── statics
├── tmp
├── util
│   └── functions.js
├── xplan-resource-server.js
└── xplan.js
```

## How to Implement a Router

### action/backend/classAction.js

```javascript
var ClassModel = require("../../model/classModel");

/**
 * Class list
 * @param req
 * @param resp
 */
exports.list = function(req, resp) {
    var classModel = new ClassModel();
    classModel.getAllClass(function(err, list) {
        if(err) {
            var result = {
                status: false,
                msg: err.message
            };
            return resp.send(200, JSON.stringify(result));
        }

        var result = {
            status: true,
            result: {
                count: list.length,
                classes: list
            }
        };

        resp.send(200, JSON.stringify(result));
    });
};
```

### router/main-rule/class.js

```javascript
var index = require("../../action/backend/indexAction");
var classAction = require("../../action/backend/classAction");

exports.post = {
};

exports.get = {
    "/class/list"       : [ index._pretreatment, classAction.list ]
};
```

## Who Reads the Routers

### router/index.js

```javascript
var util = require("util");
var config = require("../config/config");

/**
 * initialize the router.
 * @param app
 * @param callback
 */
exports.initRouter = function(app, dir, callback) {
    var walk = require("walk");
    var walker = walk.walk(__dirname + "/" + dir);

    config.logger.info("Initializing the router...");
    walker.on("file", function(root, fileStats, next) {
        var router = require("./" + dir + "/" + fileStats.name);
        var postRouter = router.post;
        var getRouter = router.get;

        // proceed the post router
        for(var key in postRouter) {
            config.logger.trace("Adding `" + key + "` to the post router...");
            if(typeof(postRouter[key]) === "function") {
                app.post(key, postRouter[key]);
            } else if(util.isArray(postRouter[key])) {
                for(var i = 0; i < postRouter[key].length; i++) {
                    app.post(key, postRouter[key][i]);
                }
            }
            config.logger.trace("Added.");
        }

        // proceed the get router
        for(var key in getRouter) {
            config.logger.trace("Adding `" + key + "` to the get router...");
            if(typeof(getRouter[key]) === "function") {
                app.get(key, getRouter[key]);
            } else if(util.isArray(getRouter[key])) {
                for(var i = 0; i < getRouter[key].length; i++) {
                    app.get(key, getRouter[key][i]);
                }
            }
            config.logger.trace("Added.");
        }

        next();
    });

    walker.on("end", function () {
        config.logger.info("Initialized.");
        callback();
    });
};
```

### xplan.js

```javascript
...

// Set the router and start up the server.
router.initRouter(app, "main-rule", function() {
    // Connect to the database
    baseModel.connect();

    // Start up the server
    http.createServer(app).listen(app.get("port"), function() {
        config.logger.info("X-Plan backend server listening on 0.0.0.0:" + app.get("port") + ".");
    });
});
```

## Features && Shortages

### Features

+ 更方便、强大、智能的路由管理功能。
+ 支持 Express 式的“中间件”机制——数组申明的链式处理路由。
+ 目录结构更清晰。

### Shortages

+ 管理方式单一。
+ 代码简陋、风格难看等。

# Version 3 - Ex(f)r(am)ess

https://github.com/XadillaX/exframess

## Directories

```
.
├── LICENSE
├── README.md
├── app.js
├── config
│   ├── db
│   │   └── mongodb.js
│   ├── index.js
│   ├── log4js.js
│   ├── renderData.js
│   ├── router.js
│   ├── secret.js
│   ├── server.js
│   ├── upload.js
│   └── useDB.js
├── controller
│   ├── commonController.js
│   └── indexController.js
├── helper
│   ├── common.js
│   ├── index.js
│   └── string.js
├── logs
│   └── deleteme
├── model
│   ├── adminModel.js
│   ├── base
│   │   ├── index.js
│   │   └── mongodb.js
│   └── exampleModel.js
├── package.json
├── router
│   └── index.js
├── statics
│   └── deleteme
├── template
│   └── deleteme
└── uploadTemp
    └── deleteme
```

## How to Implement a Router

```javascript
var helper = require("../helper");
var indexController = helper.common.getController("index");

exports.get = {
    "/"         :   indexController.index
};
```

## Who Reads the Routers

> Reference to [XPlan](https://github.com/XadillaX/xplan-backend/blob/master/router/index.js).

## Useful Helpers (under helper/*.js)

Eg.

```javascript
exports.getModel = function(modelName) {
    if(undefined === modelCache[modelName]) {
        var filename = "../model/" + modelName + "Model";
        var model = null;

        try {
            model = require(filename);
        } catch (e) {
            console.log(e);
            model = null;
            return null;
        }

        modelCache[modelName] = model;
    }

    return new modelCache[modelName]();
};
```

## Actual Examples

+ [Yogar Life](http://git.oschina.net/xadillax/Yogar-Life-Main-Server)
+ [Feime](https://coding.net/u/xadillax/p/feime/git)

## Features && Shortages

### Features

+ 框架抽离出来，可复用。
+ 框架添加了有用的函数们，并可扩展。

### Shortages

+ 只能 `clone` 框架或者复制粘贴或者解压框架。
+ 从不是很好的代码里面抽离出来，所以还是有一定的代码烂的问题。

# Version 4 - Gensokyo

http://gitlab.widget-inc.com/huaban/gensokyo

## One Possible Directories

```
.
├── app.js
├── config
│   ├── config.server.js
│   ├── config.zookeeper.js
│   └── index.js
├── controller
│   └── echoController.js
├── dummyClient
│   └── dummy.js
├── filter
│   └── echoFilter.js
├── log
├── model
└── router
    └── echoRouter.js
```

## How to Implement a Router

http://gitlab.widget-inc.com/huaban/gensokyo/tree/master#router-chain

### node_modules/gensokyo/_template/router/echoRouter.js

```javascript
/**
 * each router example:
 *
 *   + single controller
 *     - foo : "bar"
 *     - foo : [ "bar" ]
 *     - foo : { ctrller: "bar" }
 *
 *   + multi controller
 *     - foo : [ [ "bar1", "bar2" ] ]
 *     - foo : { ctrller: [ "bar1", "bar2" ] }
 *
 *   + with single/multiple `before filter`
 *     - foo : [ "before", "bar" ]                      ///< with only 2 elements in the array
 *     - foo : [ "before", [ "bar1", "bar2" ] ]         ///< with only 2 elements in the array
 *     - foo : [ [ "before1", "before2" ], [ "bar" ] ]  ///< with only 2 elements in the array
 *     - foo : { before: "before", ctrller: "bar" }
 *     - foo : { before: [ "before1", "before2" ], ctrller: "bar" }
 *
 *   + with single/multiple `after filter`
 *     - foo : { ctrller: ..., after: [ ... ] }
 *     - foo : { ctrller: ..., after: "bar" }
 *
 *   + with both `before` and `after` filters
 *     - foo : [ "before", "bar", "after" ]             ///< must with 3 elements in the array
 *     - foo : [ [ ... ], [ ... ], [ ... ] ]            ///< must with 3 elements in the array
 *     - foo : { before: "before" / [ ... ], ctrller: "bar" / [ ... ], after: "after" / [ ... ]}
 */
module.exports = {
    echo1           : [ "before1", "echo2" ],
    echo2           : [ "before1", [ "echo1", "echo2" ], "after1" ],
    echo3           : "echo1",
    echo4           : { before: [ "before1" ], ctrller: "echo1" }
};
```

### .../controller/echoController.js

```javascript
/**
 * echo controller
 * @param gensokyo
 * @constructor
 */
var EchoController = function(gensokyo) {
    Controller.call(this, gensokyo);
};

util.inherits(EchoController, Controller);

EchoController.prototype.echo1 = function(req, resp, next) {
    this.logger.info("echo1");
    next();
};

EchoController.prototype.echo2 = function(req, resp, next) {
    this.logger.info("echo2");
    resp.message.ok = true;

    // 测试人品
    if(Number.random(1, 100) < 50) {
        resp.error("You're unlucky!");
        return;
    }

    next();
};

module.exports = EchoController;
```

### .../filter/echoFilter.js

```javascript
var util = require("util");
var Filter = require("gensokyo").Filter;

/**
 * echo filter
 * @param gensokyo
 * @constructor
 */
var EchoFilter = function(gensokyo) {
    Filter.call(this, gensokyo);
};

util.inherits(EchoFilter, Filter);

EchoFilter.prototype.before1 = function(req, resp, next) {
    this.logger.info("before1");
    next();
};

EchoFilter.prototype.before2 = function(req, resp, next) {
    this.logger.info("before2");
    next();
};

EchoFilter.prototype.after1 = function(req, resp, next) {
    this.logger.info("after1");
    next();
};

EchoFilter.prototype.after2 = function(req, resp, next) {
    this.logger.info("after2");
    next();
};

module.exports = EchoFilter;
```

## Who Reads the Router

### _initControllers

```javascript
Gensokyo.prototype._initControllers = function() {
    var self = this;
    walk.walkSync(this.paths.controller, {
        listeners   : {
            file    : function(root, fileStat, next) {
                // if not root directory.
                if(path.normalize(root) !== path.normalize(self.paths.controller)) return next();

                // check filename.
                var filename = fileStat.name;
                if(!filename.endsWith("Controller.js")) return next();

                // parse controller name.
                var controllerName = filename.first(filename.length - "Controller.js".length);

                // require this controller.
                var Controller = require(root + filename);
                self.controllers[controllerName] = new Controller(self);

                next();
            }
        }
    });
};
```

### _initRouters

```javascript
Gensokyo.prototype._initRouters = function() {
    var self = this;
    walk.walkSync(this.paths.router , {
        listeners   : {
            file    : function(root, fileStat, next) {
                // if not root directory.
                if(path.normalize(root) !== path.normalize(self.paths.router)) return next();

                // check filename.
                var filename = fileStat.name;
                if(!filename.endsWith("Router.js")) return next();

                // parse router name.
                var routerName = filename.first(filename.length - "Router.js".length);

                // require this router and integrate them.
                var router = require(root + filename);
                self.routers[routerName] = integrateController.integrate(self.filters[routerName], self.controllers[routerName], router, routerName);

                next();
            },

            nodeError: function(root, nodeStatsArray, next) {
                self.logger.error(nodeStatsArray.error.message + " (" + nodeStatsArray.name + ")");
            }
        }
    });
};
```

## integrateController

### integrate

```javascript
exports.integrate = function(filter, controller, router, routerName) {
    var wrapper = {};

    // wrong router
    if(typeof router !== "object" || !router) {
        app.logger.error(
            "You should read `services/⑨/router/echoRouter.js` to know how to write router file `{routername}`.".
                assign({ routername: routerName })
        );

        process.exit(0);
    }

    // fill the controller container and filter container
    for(var key in router) {
        var beforeFunc = [], controllerFunc = [], afterFunc = [];

        fillControllersAndFilters({
            before: beforeFunc,
            controller: controllerFunc,
            after: afterFunc
        }, filter, controller, router[key], routerName);

        var funcSequence = makeSequence(beforeFunc, controllerFunc, afterFunc, routerName, key);
        wrapper[key] = funcSequence;
    }

    return wrapper;
};
```

### fillControllersAndFilters

http://gitlab.widget-inc.com/huaban/gensokyo/blob/master/lib/base/lib/integrateController.js#L51

### makeSequence

```javascript
function makeSequence(before, controller, after, routerName, keyName) {
    var sequence = [];
    sequence.add(before);
    sequence.add(controller);
    sequence.add(after);

    var func = function(req, resp, callback) {
        var funcIdx = 0;
        var funcCount = sequence.length;

        app.logger.info("Received a request \"{router}.{action}\".".assign({
            router      : routerName,
            action      : keyName
        }));

        // do the sequence
        async.whilst(
            function() {
                return funcIdx < funcCount;
            },
            function(next) {
                var func = sequence[funcIdx];
                func(req, resp, function() {
                    funcIdx++;
                    next();
                });
            },
            function(err) {
                if(err) {
                    app.logger.err("An error occurred in `{router}` -> `{action}`: {error}".assign({
                        router      : routerName,
                        action      : keyName,
                        error       : err.message
                    }));
                    return;
                }

                callback();
            }
        );
    };

    return func;
}
```

## Features && Shortages

### Features

+ 更智能、语义化的路由管理方案。
+ 更规范化、语义化的目录结构。
+ 能通过 `$ npm install -g` 安装框架，就跟 Express 一样。

### Shortages

+ 不是通用的 Web 框架。
+ 没有较好的三方函数支持及扩展（需要开发者自行自觉添加）。
+ 尚未经过真实项目考验。

# Idealized Scheme

+ 有一套强大、智能的路由管理方案。
+ 规范化、语义化、模块化或者[组件化](http://gitlab.widget-inc.com/huaban/pinit/tree/develop/lib/components)的目录结构。
+ 如模型层能兼容或者处理多套 ORM。
+ 持续维护，持续完善，通过 `npm` 或者 `cnpm` 进行管理，框架升级只需修改版本号并重新安装即可。
+ 向下兼容。
+ ...

# FAQ

轻拍。٩(｡・ω・｡)﻿و

