因为使用这个项目，自己的翻译分享出来，分享给大家，如果错误和遗漏，希望大家在Issues提醒指正。🙏
原文档地址：https://github.com/louischatriot/nedb

<img src="http://i.imgur.com/9O1xHFb.png" style="width: 25%; height: 25%; float: left;">

## The JavaScript Database

**Embedded persistent or in memory database for Node.js, nw.js, Electron and browsers, 100% JavaScript, no binary dependency**. API is a subset of MongoDB's and it's <a href="#speed">plenty fast</a>.

为Node.js、nw.js、Electron、浏览器设计，100%的Javascript程序可使用的嵌入式或内存数据库，不依赖二进制库。数据库API是MongoDB的一个子集，运行速度非常快。

**IMPORTANT NOTE**: Please don't submit issues for questions regarding your code. Only actual bugs or feature requests will be answered, all others will be closed without comment. Also, please follow the <a href="#bug-reporting-guidelines">bug reporting guidelines</a> and check the <a href="https://github.com/louischatriot/nedb/wiki/Change-log" target="_blank">change log</a> before submitting an already fixed bug :)

重要提示：不要提交使用过程中的代码问题，只有数据库Bug或希望增加的功能才会回复。所有其他的评论都将关闭。另外提交bug前先看已经提交过的bug，然后再提交。

## Support NeDB development（赞助NeDB开发）

<img src="http://i.imgur.com/mpwi4lf.jpg">

No time to <a href="#pull-requests">help out</a>? You can support NeDB development by sending money or bitcoins!

Money: [![Donate to author](https://www.paypalobjects.com/en_US/i/btn/btn_donate_SM.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=louis%2echatriot%40gmail%2ecom&lc=US&currency_code=EUR&bn=PP%2dDonationsBF%3abtn_donate_LG%2egif%3aNonHostedGuest)

Bitcoin address: 1dDZLnWpBbodPiN8sizzYrgaz5iahFyb1


## Installation, tests
Module name on npm and bower is `nedb`.

在npm上的模块名字是nedb

```
npm install nedb --save    # Put latest version in your package.json
npm test                   # You'll need the dev dependencies to launch tests
bower install nedb         # For the browser versions, which will be in browser-version/out
```

## API
It is a subset of MongoDB's API (the most used operations).

这是MongoDB的API子集，最常用的一部分操作

* <a href="#creatingloading-a-database">Creating/loading a database</a>
* <a href="#persistence">Persistence</a>
* <a href="#inserting-documents">Inserting documents</a>
* <a href="#finding-documents">Finding documents</a>
  * <a href="#basic-querying">Basic Querying</a>
  * <a href="#operators-lt-lte-gt-gte-in-nin-ne-exists-regex">Operators ($lt, $lte, $gt, $gte, $in, $nin, $ne, $exists, $regex)</a>
  * <a href="#array-fields">Array fields</a>
  * <a href="#logical-operators-or-and-not-where">Logical operators $or, $and, $not, $where</a>
  * <a href="#sorting-and-paginating">Sorting and paginating</a>
  * <a href="#projections">Projections</a>
* <a href="#counting-documents">Counting documents</a>
* <a href="#updating-documents">Updating documents</a>
* <a href="#removing-documents">Removing documents</a>
* <a href="#indexing">Indexing</a>
* <a href="#browser-version">Browser version</a>

### Creating/loading a database
You can use NeDB as an in-memory only datastore or as a persistent datastore. One datastore is the equivalent of a MongoDB collection. The constructor is used as follows `new Datastore(options)` where `options` is an object with the following fields:

可以吧NeDB作为内存数据库，当然也可以持久化存储。一个数据存储区相当于MongoDB的一个集合（collection）。

* `filename` (optional): path to the file where the data is persisted. If left blank, the datastore is automatically considered in-memory only. It cannot end with a `~` which is used in the temporary files NeDB uses to perform crash-safe writes.
* `filename`（可选项），数据持久化存储的文件路径。如果为空就只是会作为内存数据库使用。不要吧这个文件当做程序崩溃向NeDB写入的临时文件。
* `inMemoryOnly` (optional, defaults to `false`): as the name implies.
* `inMemoryOnly`（缺省为false）：顾名思义，是否只作为内存数据库使用。
* `timestampData` (optional, defaults to `false`): timestamp the insertion and last update of all documents, with the fields `createdAt` and `updatedAt`. User-specified values override automatic generation, usually useful for testing.
* `timestampData`（缺省false）插入和更新时间戳到所有文档，属性`createdAt`和`updatedAt`，如果用指定了值，将会自动被覆盖，这个功能一般用于测试。
* `autoload` (optional, defaults to `false`): if used, the database will automatically be loaded from the datafile upon creation (you don't need to call `loadDatabase`). Any command issued before load is finished is buffered and will be executed when load is done.
* `autoload`（缺省false）：如果使用这个参数，数据库将自动从数据库文件加载数据（不需要调用`loadDatebase`函数）。将在任何命令发出前进行加载。
* `onload` (optional): if you use autoloading, this is the handler called after the `loadDatabase`. It takes one `error` argument. If you use autoloading without specifying this handler, and an error happens during load, an error will be thrown.
* `onload`：如果使用了自动加载，`loadDatabase`之后会被调用。需要参数`error`，如果不指定这个回调函数，自动加载过程中发生错误就会抛出异常。
* `afterSerialization` (optional): hook you can use to transform data after it was serialized and before it is written to disk. Can be used for example to encrypt data before writing database to disk. This function takes a string as parameter (one line of an NeDB data file) and outputs the transformed string, **which must absolutely not contain a `\n` character** (or data will be lost).
* `afterSerialization`：可以使用这个钩子将数据序列化并写入磁盘。可以在写入磁盘之前将数据加密。此函数将一个字符串作为参数（NeDB数据文件的一行），并且需要返回一个字符串，**字符串结尾不要包含`\n`字符**（否则数据会丢失）。
* `beforeDeserialization` (optional): inverse of `afterSerialization`. Make sure to include both and not just one or you risk data loss. For the same reason, make sure both functions are inverses of one another. Some failsafe mechanisms are in place to prevent data loss if you misuse the serialization hooks: NeDB checks that never one is declared without the other, and checks that they are reverse of one another by testing on random strings of various lengths. In addition, if too much data is detected as corrupt, NeDB will refuse to start as it could mean you're not using the deserialization hook corresponding to the serialization hook used before (see below).
* `beforeDeserialization`：`afterSerialization`的逆向操作，要确保该操作没有任何风险。出于安全考虑，要确保两个函数操作是逆向的。为了防止数据丢失有必要使用一些安全机制，不要滥用这个钩子函数。NeDB从来不会用随机字符串测试你的算法是否是逆向的。另外，NeDB如果检测到坏数据将会拒绝启动，因为有可能您无法逆向以前的加密数据（见下文）。
* `corruptAlertThreshold` (optional): between 0 and 1, defaults to 10%. NeDB will refuse to start if more than this percentage of the datafile is corrupt. 0 means you don't tolerate any corruption, 1 means you don't care.
* `corruptAlertThreshold`：介于0和1之间，缺省为10%。如果超过这个比例的数据被损坏，NeDB将会拒绝启动。0意味着你不允许任何坏数据，1意味着你不介意任何坏数据。
* `compareStrings` (optional): function compareStrings(a, b) compares strings a and b and return -1, 0 or 1. If specified, it overrides default string comparison which is not well adapted to non-US characters in particular accented letters. Native `localCompare` will most of the
time be the right choice
* `compareStrings`：函数compareStrings(a, b)将会比较字符串a和b并且返回-1、0、1。如果指定这个参数，他将会覆盖默认的字符串比较。默认的比较不太适合非英文字符。尽量使用适合自己语言的字符串比较函数。
* `nodeWebkitAppName` (optional, **DEPRECATED**): if you are using NeDB from whithin a Node Webkit app, specify its name (the same one you use in the `package.json`) in this field and the `filename` will be relative to the directory Node Webkit uses to store the rest of the application's data (local storage etc.). It works on Linux, OS X and Windows. Now that you can use `require('nw.gui').App.dataPath` in Node Webkit to get the path to the data directory for your application, you should not use this option anymore and it will be removed.
* `nodeWebkitAppName`（该参数已弃用）

If you use a persistent datastore without the `autoload` option, you need to call `loadDatabase` manually.
This function fetches the data from datafile and prepares the database. **Don't forget it!** If you use a
persistent datastore, no command (insert, find, update, remove) will be executed before `loadDatabase`
is called, so make sure to call it yourself or use the `autoload` option.

如果不使用`autoload`选项，需要手动调用`loadDatabase`函数来实现持久化存储。这个函数将数据写入数据库文件。**切记切记！**如果不调用这个函数，数据的任何操作（插入、查找、更新、删除）都不会保存，所以一定要确保使用了`autoload`选项选项或者手动调用了`loadDatabase`函数。

Also, if `loadDatabase` fails, all commands registered to the executor afterwards will not be executed. They will be registered and executed, in sequence, only after a successful `loadDatabase`.

另外，如果`loadDatabase`函数调用失败，那么后面的命令都不会被执行。只有在成功调用`loadDatabase`函数之后才可以执行。

```javascript
// Type 1: In-memory only datastore (no need to load the database)
// 第一种类型，内存性数据库（不需要加载数据库）
var Datastore = require('nedb')
  , db = new Datastore();


// Type 2: Persistent datastore with manual loading
// 第二种类型，手动加载持久化数据库
var Datastore = require('nedb')
  , db = new Datastore({ filename: 'path/to/datafile' });
db.loadDatabase(function (err) {    // Callback is optional 回调函数是可选的
  // Now commands will be executed
  // 数据库写入后执行这里
});


// Type 3: Persistent datastore with automatic loading
// 第三种类型，自动加载的持久化存储数据库
var Datastore = require('nedb')
  , db = new Datastore({ filename: 'path/to/datafile', autoload: true });
// You can issue commands right away
// 这里可以立即执行数据库操作


// Type 4: Persistent datastore for a Node Webkit app called 'nwtest'
// For example on Linux, the datafile will be ~/.config/nwtest/nedb-data/something.db
// 第四种类型：Node Webkit的起名为‘nwtest’的持久化应用
// 比如在Linux上，数据库文件将会写入 ~/.config/nwtest/nedb-data/something.db
var Datastore = require('nedb')
  , path = require('path')
  , db = new Datastore({ filename: path.join(require('nw.gui').App.dataPath, 'something.db') });


// Of course you can create multiple datastores if you need several
// collections. In this case it's usually a good idea to use autoload for all collections.
// 如果需要，可以创建多个数据存储集合，这种情况下建议使用自动保存
db = {};
db.users = new Datastore('path/to/users.db');
db.robots = new Datastore('path/to/robots.db');

// You need to load each database (here we do it asynchronously)
// 这里需要加载每一个数据库（这里是异步方式）
db.users.loadDatabase();
db.robots.loadDatabase();
```

### Persistence （续篇）
Under the hood, NeDB's persistence uses an append-only format, meaning that all updates and deletes actually result in lines added at the end of the datafile, for performance reasons. The database is automatically compacted (i.e. put back in the one-line-per-document format) every time you load each database within your application.

该存储引擎处于性能考虑，NeDB的持久化存储仅使用追加方式，这样会导致所有的更新和删除实际上只是在末尾添加行。每次应用程序加载数据库时，数据库将自动压缩（单行文档格式）。

You can manually call the compaction function with `yourDatabase.persistence.compactDatafile` which takes no argument. It queues a compaction of the datafile in the executor, to be executed sequentially after all pending operations. The datastore will fire a `compaction.done` event once compaction is finished.

可以不带参数的调用`yourDatabase.persistence.compactDatafile`压缩数据库。他将在执行器对数据文件的压缩进行排队，等所有操作处理完再执行。如果压缩完，将会出发数据库的`compaction.done`事件。

You can also set automatic compaction at regular intervals with `yourDatabase.persistence.setAutocompactionInterval(interval)`, `interval` in milliseconds (a minimum of 5s is enforced), and stop automatic compaction with `yourDatabase.persistence.stopAutocompaction()`.

可以设置`yourDatabase.persistence.setAutocompactionInterval(interval)`定时自动压缩，`interval`以毫秒为单位（最短5秒调用一次），并且要停止使用`yourDatabase.persistence.stopAutocompaction()`的自动压缩。

Keep in mind that compaction takes a bit of time (not too much: 130ms for 50k records on a typical development machine) and no other operation can happen when it does, so most projects actually don't need to use it.

切忌压缩是需要时间（一般的开发机无其他干扰，50k数据需要130毫秒）的，这时间内不可以执行其他操作，大部分项目不需要用着个功能。

Compaction will also immediately remove any documents whose data line has become corrupted, assuming that the total percentage of all corrupted documents in that database still falls below the specified `corruptAlertThreshold` option's value.

压缩还将删除任何坏文档，假如数据库的坏文档比例低于`corruptAlertThreshold`选项的值。

Durability works similarly to major databases: compaction forces the OS to physically flush data to disk, while appends to the data file do not (the OS is responsible for flushing the data). That guarantees that a server crash can never cause complete data loss, while preserving performance. The worst that can happen is a crash between two syncs, causing a loss of all data between the two syncs. Usually syncs are 30 seconds appart so that's at most 30 seconds of data. <a href="http://oldblog.antirez.com/post/redis-persistence-demystified.html" target="_blank">This post by Antirez on Redis persistence</a> explains this in more details, NeDB being very close to Redis AOF persistence with `appendfsync` option set to `no`.

持久性和其他主流数据库类似，压缩操作强制操作系统将数据保存到物理磁盘，而追加数据不会如此（操作系统负责刷新数据）。这样既能保证数据的完整性，又能保持较高性能。可能发生的最坏事情是两次同步之间发生崩溃，造成两次同步之间的数据操作丢失。通常30秒同步一次数据，最差丢30秒的数据。Antirez在<a href="http://oldblog.antirez.com/post/redis-persistence-demystified.html" target="_blank">Redis数据持久化存储</a>文章里详细的解释了这一点。NeDB非常接近Redis的AOF持久化选项`appendfsync`设置为`no`。


### Inserting documents（添加文档）
The native types are `String`, `Number`, `Boolean`, `Date` and `null`. You can also use
arrays and subdocuments (objects). If a field is `undefined`, it will not be saved (this is different from 
MongoDB which transforms `undefined` in `null`, something I find counter-intuitive).

支持的类型是`String`、`Number`、`Boolean`、`Date`和`null`。还可以试用数组和子文档，如果字段未定义不会保存（这一点与MongoDB是不同的）。

If the document does not contain an `_id` field, NeDB will automatically generated one for you (a 16-characters alphanumerical string). The `_id` of a document, once set, cannot be modified.

如果文档没有设置`_id`，NeDB将会自动生成（长16的字符串）。文档`_id`一旦设置无法修改。

Field names cannot begin by '$' or contain a '.'.

字段名不可以'$'开头，也不可以包含'.'。

```javascript
var doc = { hello: 'world'
               , n: 5
               , today: new Date()
               , nedbIsAwesome: true
               , notthere: null
               , notToBeSaved: undefined  // Will not be saved
               , fruits: [ 'apple', 'orange', 'pear' ]
               , infos: { name: 'nedb' }
               };

db.insert(doc, function (err, newDoc) {   // Callback is optional
  // newDoc is the newly inserted document, including its _id
  // newDoc has no key called notToBeSaved since its value was undefined
});
```

You can also bulk-insert an array of documents. This operation is atomic, meaning that if one insert fails due to a unique constraint being violated, all changes are rolled back.

可以将数组批量插入数据库，这样操作是原子操作，如果有一个不合规范，其他操作都将回滚。

```javascript
db.insert([{ a: 5 }, { a: 42 }], function (err, newDocs) {
  // Two documents were inserted in the database
  // newDocs is an array with these documents, augmented with their _id
});

// If there is a unique constraint on field 'a', this will fail
db.insert([{ a: 5 }, { a: 42 }, { a: 5 }], function (err) {
  // err is a 'uniqueViolated' error
  // The database was not modified
});
```

### Finding documents （查找文档）
Use `find` to look for multiple documents matching you query, or `findOne` to look for one specific document. You can select documents based on field equality or use comparison operators (`$lt`, `$lte`, `$gt`, `$gte`, `$in`, `$nin`, `$ne`). You can also use logical operators `$or`, `$and`, `$not` and `$where`. See below for the syntax.

使用`find`查找和匹配多个文档，或者用`findOne`读取一个文档。可以用比较符号筛选文档(`$lt`、`$lte`、`$gt`、`$gte`、`$in`、`$nin`、`$ne`)。还可以用逻辑运算符`$or`、`$and`、`$not`和`$where`，请看下面的语法。

You can use regular expressions in two ways: in basic querying in place of a string, or with the `$regex` operator.

两种方式可以使用正则，基本查询中或`$regex`操作符中。

You can sort and paginate results using the cursor API (see below).

可以使用游标API对结果进行排序和分页，详见下文。

You can use standard projections to restrict the fields to appear in the results (see below).

可以使用投影的方式限制字段出现在结果中，详见下文。

#### Basic querying （基本查询）
Basic querying means are looking for documents whose fields match the ones you specify. You can use regular expression to match strings.
You can use the dot notation to navigate inside nested documents, arrays, arrays of subdocuments and to match a specific element of an array.

基本查询是查找和条件匹配的文档。可以用正则来匹配字符串。可以用`.`号来引用嵌套文档、数组、子文档数组，并匹配指定的元素。

```javascript
// Let's say our datastore contains the following collection
// 假设我们的数据库包含下面的数据集合
// { _id: 'id1', planet: 'Mars', system: 'solar', inhabited: false, satellites: ['Phobos', 'Deimos'] }
// { _id: 'id2', planet: 'Earth', system: 'solar', inhabited: true, humans: { genders: 2, eyes: true } }
// { _id: 'id3', planet: 'Jupiter', system: 'solar', inhabited: false }
// { _id: 'id4', planet: 'Omicron Persei 8', system: 'futurama', inhabited: true, humans: { genders: 7 } }
// { _id: 'id5', completeData: { planets: [ { name: 'Earth', number: 3 }, { name: 'Mars', number: 2 }, { name: 'Pluton', number: 9 } ] } }

// Finding all planets in the solar system
// 查找太阳系中的所有行星
db.find({ system: 'solar' }, function (err, docs) {
  // docs is an array containing documents Mars, Earth, Jupiter
  // If no document is found, docs is equal to []
});

// Finding all planets whose name contain the substring 'ar' using a regular expression
// 使用正则查找名称中包含“ar”的所有行星
db.find({ planet: /ar/ }, function (err, docs) {
  // docs contains Mars and Earth
});

// Finding all inhabited planets in the solar system
// 查找太阳系中所有有人的行星
db.find({ system: 'solar', inhabited: true }, function (err, docs) {
  // docs is an array containing document Earth only
});

// Use the dot-notation to match fields in subdocuments
// 使用`.`符号匹配子文档中的字段
db.find({ "humans.genders": 2 }, function (err, docs) {
  // docs contains Earth
});

// Use the dot-notation to navigate arrays of subdocuments
// 使用`.`符号浏览子文档中的数组
db.find({ "completeData.planets.name": "Mars" }, function (err, docs) {
  // docs contains document 5
});

db.find({ "completeData.planets.name": "Jupiter" }, function (err, docs) {
  // docs is empty
});

db.find({ "completeData.planets.0.name": "Earth" }, function (err, docs) {
  // docs contains document 5
  // If we had tested against "Mars" docs would be empty because we are matching against a specific array element
});


// You can also deep-compare objects. Don't confuse this with dot-notation!
// 也可以深入的比较对象，不要和`.`符号混淆
db.find({ humans: { genders: 2 } }, function (err, docs) {
  // docs is empty, because { genders: 2 } is not equal to { genders: 2, eyes: true }
});

// Find all documents in the collection
db.find({}, function (err, docs) {
});

// The same rules apply when you want to only find one document
db.findOne({ _id: 'id1' }, function (err, doc) {
  // doc is the document Mars
  // If no document is found, doc is null
});
```

#### Operators ($lt, $lte, $gt, $gte, $in, $nin, $ne, $exists, $regex)（操作符）
The syntax is `{ field: { $op: value } }` where `$op` is any comparison operator:  

* `$lt`, `$lte`: less than, less than or equal
* `$lt`, `$lte`: 小于、小于等于
* `$gt`, `$gte`: greater than, greater than or equal
* `$gt`, `$gte`: 大于、大于等于
* `$in`: member of. `value` must be an array of values
* `$in`: 成员。`value`必须是数组
* `$ne`, `$nin`: not equal, not a member of
* `$ne`, `$nin`: 不等于、不是成员
* `$exists`: checks whether the document posses the property `field`. `value` should be true or false
* `$exists`: 检查文档是否有指定的字段属性`field`.`value`，返回true或false
* `$regex`: checks whether a string is matched by the regular expression. Contrary to MongoDB, the use of `$options` with `$regex` is not supported, because it doesn't give you more power than regex flags. Basic queries are more readable so only use the `$regex` operator when you need to use another operator with it (see example below)
* `$regex`: 检查字符串是否与正则表达式匹配，与MongoDB相反，不支持`$options`，因为它不会比正则表达式更强。基本查询可读性更好，最好在确实需要正则是时候才使用（详情如下）

```javascript
// $lt, $lte, $gt and $gte work on numbers and strings
db.find({ "humans.genders": { $gt: 5 } }, function (err, docs) {
  // docs contains Omicron Persei 8, whose humans have more than 5 genders (7).
});

// When used with strings, lexicographical order is used
db.find({ planet: { $gt: 'Mercury' }}, function (err, docs) {
  // docs contains Omicron Persei 8
})

// Using $in. $nin is used in the same way
db.find({ planet: { $in: ['Earth', 'Jupiter'] }}, function (err, docs) {
  // docs contains Earth and Jupiter
});

// Using $exists
db.find({ satellites: { $exists: true } }, function (err, docs) {
  // docs contains only Mars
});

// Using $regex with another operator
db.find({ planet: { $regex: /ar/, $nin: ['Jupiter', 'Earth'] } }, function (err, docs) {
  // docs only contains Mars because Earth was excluded from the match by $nin
});
```

#### Array fields
When a field in a document is an array, NeDB first tries to see if the query value is an array to perform an exact match, then whether there is an array-specific comparison function (for now there is only `$size` and `$elemMatch`) being used. If not, the query is treated as a query on every element and there is a match if at least one element matches.  

当一个文档中的字段是一个数组时，NeDB首先尝试查看值是否是一个数组来执行完全匹配，然后查看是否使用数组特定的比较函数（目前只支持`$size`和`$elemMatch`）。如果没有使用，会查询每一个数组元素，只要有一个数组向匹配，该文档就满足条件。

* `$size`: match on the size of the array
* `$elemMatch`: matches if at least one array element matches the query entirely

```javascript
// Exact match
db.find({ satellites: ['Phobos', 'Deimos'] }, function (err, docs) {
  // docs contains Mars
})
db.find({ satellites: ['Deimos', 'Phobos'] }, function (err, docs) {
  // docs is empty
})

// Using an array-specific comparison function
// $elemMatch operator will provide match for a document, if an element from the array field satisfies all the conditions specified with the `$elemMatch` operator
db.find({ completeData: { planets: { $elemMatch: { name: 'Earth', number: 3 } } } }, function (err, docs) {
  // docs contains documents with id 5 (completeData)
});

db.find({ completeData: { planets: { $elemMatch: { name: 'Earth', number: 5 } } } }, function (err, docs) {
  // docs is empty
});

// You can use inside #elemMatch query any known document query operator
db.find({ completeData: { planets: { $elemMatch: { name: 'Earth', number: { $gt: 2 } } } } }, function (err, docs) {
  // docs contains documents with id 5 (completeData)
});

// Note: you can't use nested comparison functions, e.g. { $size: { $lt: 5 } } will throw an error
db.find({ satellites: { $size: 2 } }, function (err, docs) {
  // docs contains Mars
});

db.find({ satellites: { $size: 1 } }, function (err, docs) {
  // docs is empty
});

// If a document's field is an array, matching it means matching any element of the array
db.find({ satellites: 'Phobos' }, function (err, docs) {
  // docs contains Mars. Result would have been the same if query had been { satellites: 'Deimos' }
});

// This also works for queries that use comparison operators
db.find({ satellites: { $lt: 'Amos' } }, function (err, docs) {
  // docs is empty since Phobos and Deimos are after Amos in lexicographical order
});

// This also works with the $in and $nin operator
db.find({ satellites: { $in: ['Moon', 'Deimos'] } }, function (err, docs) {
  // docs contains Mars (the Earth document is not complete!)
});
```

#### Logical operators $or, $and, $not, $where
You can combine queries using logical operators:  

* For `$or` and `$and`, the syntax is `{ $op: [query1, query2, ...] }`.
* For `$not`, the syntax is `{ $not: query }`
* For `$where`, the syntax is `{ $where: function () { /* object is "this", return a boolean */ } }`

```javascript
db.find({ $or: [{ planet: 'Earth' }, { planet: 'Mars' }] }, function (err, docs) {
  // docs contains Earth and Mars
});

db.find({ $not: { planet: 'Earth' } }, function (err, docs) {
  // docs contains Mars, Jupiter, Omicron Persei 8
});

db.find({ $where: function () { return Object.keys(this) > 6; } }, function (err, docs) {
  // docs with more than 6 properties
});

// You can mix normal queries, comparison queries and logical operators
db.find({ $or: [{ planet: 'Earth' }, { planet: 'Mars' }], inhabited: true }, function (err, docs) {
  // docs contains Earth
});

```

#### Sorting and paginating
If you don't specify a callback to `find`, `findOne` or `count`, a `Cursor` object is returned. You can modify the cursor with `sort`, `skip` and `limit` and then execute it with `exec(callback)`.

```javascript
// Let's say the database contains these 4 documents
// doc1 = { _id: 'id1', planet: 'Mars', system: 'solar', inhabited: false, satellites: ['Phobos', 'Deimos'] }
// doc2 = { _id: 'id2', planet: 'Earth', system: 'solar', inhabited: true, humans: { genders: 2, eyes: true } }
// doc3 = { _id: 'id3', planet: 'Jupiter', system: 'solar', inhabited: false }
// doc4 = { _id: 'id4', planet: 'Omicron Persei 8', system: 'futurama', inhabited: true, humans: { genders: 7 } }

// No query used means all results are returned (before the Cursor modifiers)
db.find({}).sort({ planet: 1 }).skip(1).limit(2).exec(function (err, docs) {
  // docs is [doc3, doc1]
});

// You can sort in reverse order like this
db.find({ system: 'solar' }).sort({ planet: -1 }).exec(function (err, docs) {
  // docs is [doc1, doc3, doc2]
});

// You can sort on one field, then another, and so on like this:
db.find({}).sort({ firstField: 1, secondField: -1 }) ...   // You understand how this works!
```

#### Projections
You can give `find` and `findOne` an optional second argument, `projections`. The syntax is the same as MongoDB: `{ a: 1, b: 1 }` to return only the `a` and `b` fields, `{ a: 0, b: 0 }` to omit these two fields. You cannot use both modes at the time, except for `_id` which is by default always returned and which you can choose to omit. You can project on nested documents.

```javascript
// Same database as above

// Keeping only the given fields
db.find({ planet: 'Mars' }, { planet: 1, system: 1 }, function (err, docs) {
  // docs is [{ planet: 'Mars', system: 'solar', _id: 'id1' }]
});

// Keeping only the given fields but removing _id
db.find({ planet: 'Mars' }, { planet: 1, system: 1, _id: 0 }, function (err, docs) {
  // docs is [{ planet: 'Mars', system: 'solar' }]
});

// Omitting only the given fields and removing _id
db.find({ planet: 'Mars' }, { planet: 0, system: 0, _id: 0 }, function (err, docs) {
  // docs is [{ inhabited: false, satellites: ['Phobos', 'Deimos'] }]
});

// Failure: using both modes at the same time
db.find({ planet: 'Mars' }, { planet: 0, system: 1 }, function (err, docs) {
  // err is the error message, docs is undefined
});

// You can also use it in a Cursor way but this syntax is not compatible with MongoDB
db.find({ planet: 'Mars' }).projection({ planet: 1, system: 1 }).exec(function (err, docs) {
  // docs is [{ planet: 'Mars', system: 'solar', _id: 'id1' }]
});

// Project on a nested document
db.findOne({ planet: 'Earth' }).projection({ planet: 1, 'humans.genders': 1 }).exec(function (err, doc) {
  // doc is { planet: 'Earth', _id: 'id2', humans: { genders: 2 } }
});
```



### Counting documents
You can use `count` to count documents. It has the same syntax as `find`. For example:

```javascript
// Count all planets in the solar system
db.count({ system: 'solar' }, function (err, count) {
  // count equals to 3
});

// Count all documents in the datastore
db.count({}, function (err, count) {
  // count equals to 4
});
```


### Updating documents
`db.update(query, update, options, callback)` will update all documents matching `query` according to the `update` rules:  
* `query` is the same kind of finding query you use with `find` and `findOne`
* `update` specifies how the documents should be modified. It is either a new document or a set of modifiers (you cannot use both together, it doesn't make sense!)
  * A new document will replace the matched docs
  * The modifiers create the fields they need to modify if they don't exist, and you can apply them to subdocs. Available field modifiers are `$set` to change a field's value, `$unset` to delete a field, `$inc` to increment a field's value and `$min`/`$max` to change field's value, only if provided value is less/greater than current value. To work on arrays, you have `$push`, `$pop`, `$addToSet`, `$pull`, and the special `$each` and `$slice`. See examples below for the syntax.
* `options` is an object with two possible parameters
  * `multi` (defaults to `false`) which allows the modification of several documents if set to true
  * `upsert` (defaults to `false`) if you want to insert a new document corresponding to the `update` rules if your `query` doesn't match anything. If your `update` is a simple object with no modifiers, it is the inserted document. In the other case, the `query` is stripped from all operator recursively, and the `update` is applied to it.
  * `returnUpdatedDocs` (defaults to `false`, not MongoDB-compatible) if set to true and update is not an upsert, will return the array of documents matched by the find query and updated. Updated documents will be returned even if the update did not actually modify them.
* `callback` (optional) signature: `(err, numAffected, affectedDocuments, upsert)`. **Warning**: the API was changed between v1.7.4 and v1.8. Please refer to the <a href="https://github.com/louischatriot/nedb/wiki/Change-log" target="_blank">change log</a> to see the change.
  * For an upsert, `affectedDocuments` contains the inserted document and the `upsert` flag is set to `true`.
  * For a standard update with `returnUpdatedDocs` flag set to `false`, `affectedDocuments` is not set.
  * For a standard update with `returnUpdatedDocs` flag set to `true` and `multi` to `false`, `affectedDocuments` is the updated document.
  * For a standard update with `returnUpdatedDocs` flag set to `true` and `multi` to `true`, `affectedDocuments` is the array of updated documents.

**Note**: you can't change a document's _id.

```javascript
// Let's use the same example collection as in the "finding document" part
// { _id: 'id1', planet: 'Mars', system: 'solar', inhabited: false }
// { _id: 'id2', planet: 'Earth', system: 'solar', inhabited: true }
// { _id: 'id3', planet: 'Jupiter', system: 'solar', inhabited: false }
// { _id: 'id4', planet: 'Omicron Persia 8', system: 'futurama', inhabited: true }

// Replace a document by another
db.update({ planet: 'Jupiter' }, { planet: 'Pluton'}, {}, function (err, numReplaced) {
  // numReplaced = 1
  // The doc #3 has been replaced by { _id: 'id3', planet: 'Pluton' }
  // Note that the _id is kept unchanged, and the document has been replaced
  // (the 'system' and inhabited fields are not here anymore)
});

// Set an existing field's value
db.update({ system: 'solar' }, { $set: { system: 'solar system' } }, { multi: true }, function (err, numReplaced) {
  // numReplaced = 3
  // Field 'system' on Mars, Earth, Jupiter now has value 'solar system'
});

// Setting the value of a non-existing field in a subdocument by using the dot-notation
db.update({ planet: 'Mars' }, { $set: { "data.satellites": 2, "data.red": true } }, {}, function () {
  // Mars document now is { _id: 'id1', system: 'solar', inhabited: false
  //                      , data: { satellites: 2, red: true }
  //                      }
  // Not that to set fields in subdocuments, you HAVE to use dot-notation
  // Using object-notation will just replace the top-level field
  db.update({ planet: 'Mars' }, { $set: { data: { satellites: 3 } } }, {}, function () {
    // Mars document now is { _id: 'id1', system: 'solar', inhabited: false
    //                      , data: { satellites: 3 }
    //                      }
    // You lost the "data.red" field which is probably not the intended behavior
  });
});

// Deleting a field
db.update({ planet: 'Mars' }, { $unset: { planet: true } }, {}, function () {
  // Now the document for Mars doesn't contain the planet field
  // You can unset nested fields with the dot notation of course
});

// Upserting a document
db.update({ planet: 'Pluton' }, { planet: 'Pluton', inhabited: false }, { upsert: true }, function (err, numReplaced, upsert) {
  // numReplaced = 1, upsert = { _id: 'id5', planet: 'Pluton', inhabited: false }
  // A new document { _id: 'id5', planet: 'Pluton', inhabited: false } has been added to the collection
});

// If you upsert with a modifier, the upserted doc is the query modified by the modifier
// This is simpler than it sounds :)
db.update({ planet: 'Pluton' }, { $inc: { distance: 38 } }, { upsert: true }, function () {
  // A new document { _id: 'id5', planet: 'Pluton', distance: 38 } has been added to the collection  
});

// If we insert a new document { _id: 'id6', fruits: ['apple', 'orange', 'pear'] } in the collection,
// let's see how we can modify the array field atomically

// $push inserts new elements at the end of the array
db.update({ _id: 'id6' }, { $push: { fruits: 'banana' } }, {}, function () {
  // Now the fruits array is ['apple', 'orange', 'pear', 'banana']
});

// $pop removes an element from the end (if used with 1) or the front (if used with -1) of the array
db.update({ _id: 'id6' }, { $pop: { fruits: 1 } }, {}, function () {
  // Now the fruits array is ['apple', 'orange']
  // With { $pop: { fruits: -1 } }, it would have been ['orange', 'pear']
});

// $addToSet adds an element to an array only if it isn't already in it
// Equality is deep-checked (i.e. $addToSet will not insert an object in an array already containing the same object)
// Note that it doesn't check whether the array contained duplicates before or not
db.update({ _id: 'id6' }, { $addToSet: { fruits: 'apple' } }, {}, function () {
  // The fruits array didn't change
  // If we had used a fruit not in the array, e.g. 'banana', it would have been added to the array
});

// $pull removes all values matching a value or even any NeDB query from the array
db.update({ _id: 'id6' }, { $pull: { fruits: 'apple' } }, {}, function () {
  // Now the fruits array is ['orange', 'pear']
});
db.update({ _id: 'id6' }, { $pull: { fruits: $in: ['apple', 'pear'] } }, {}, function () {
  // Now the fruits array is ['orange']
});

// $each can be used to $push or $addToSet multiple values at once
// This example works the same way with $addToSet
db.update({ _id: 'id6' }, { $push: { fruits: { $each: ['banana', 'orange'] } } }, {}, function () {
  // Now the fruits array is ['apple', 'orange', 'pear', 'banana', 'orange']
});

// $slice can be used in cunjunction with $push and $each to limit the size of the resulting array.
// A value of 0 will update the array to an empty array. A positive value n will keep only the n first elements
// A negative value -n will keep only the last n elements.
// If $slice is specified but not $each, $each is set to []
db.update({ _id: 'id6' }, { $push: { fruits: { $each: ['banana'], $slice: 2 } } }, {}, function () {
  // Now the fruits array is ['apple', 'orange']
});

// $min/$max to update only if provided value is less/greater than current value
// Let's say the database contains this document
// doc = { _id: 'id', name: 'Name', value: 5 }
db.update({ _id: 'id1' }, { $min: { value: 2 } }, {}, function () {
  // The document will be updated to { _id: 'id', name: 'Name', value: 2 }
});

db.update({ _id: 'id1' }, { $min: { value: 8 } }, {}, function () {
  // The document will not be modified
});
```

### Removing documents
`db.remove(query, options, callback)` will remove all documents matching `query` according to `options`  
* `query` is the same as the ones used for finding and updating
* `options` only one option for now: `multi` which allows the removal of multiple documents if set to true. Default is false
* `callback` is optional, signature: err, numRemoved

```javascript
// Let's use the same example collection as in the "finding document" part
// { _id: 'id1', planet: 'Mars', system: 'solar', inhabited: false }
// { _id: 'id2', planet: 'Earth', system: 'solar', inhabited: true }
// { _id: 'id3', planet: 'Jupiter', system: 'solar', inhabited: false }
// { _id: 'id4', planet: 'Omicron Persia 8', system: 'futurama', inhabited: true }

// Remove one document from the collection
// options set to {} since the default for multi is false
db.remove({ _id: 'id2' }, {}, function (err, numRemoved) {
  // numRemoved = 1
});

// Remove multiple documents
db.remove({ system: 'solar' }, { multi: true }, function (err, numRemoved) {
  // numRemoved = 3
  // All planets from the solar system were removed
});

// Removing all documents with the 'match-all' query
db.remove({}, { multi: true }, function (err, numRemoved) {
});
```

### Indexing
NeDB supports indexing. It gives a very nice speed boost and can be used to enforce a unique constraint on a field. You can index any field, including fields in nested documents using the dot notation. For now, indexes are only used to speed up basic queries and queries using `$in`, `$lt`, `$lte`, `$gt` and `$gte`. The indexed values cannot be of type array of object.

To create an index, use `datastore.ensureIndex(options, cb)`, where callback is optional and get passed an error if any (usually a unique constraint that was violated). `ensureIndex` can be called when you want, even after some data was inserted, though it's best to call it at application startup. The options are:  

* **fieldName** (required): name of the field to index. Use the dot notation to index a field in a nested document.
* **unique** (optional, defaults to `false`): enforce field uniqueness. Note that a unique index will raise an error if you try to index two documents for which the field is not defined.
* **sparse** (optional, defaults to `false`): don't index documents for which the field is not defined. Use this option along with "unique" if you want to accept multiple documents for which it is not defined.
* **expireAfterSeconds** (number of seconds, optional): if set, the created index is a TTL (time to live) index, that will automatically remove documents when the system date becomes larger than the date on the indexed field plus `expireAfterSeconds`. Documents where the indexed field is not specified or not a `Date` object are ignored

Note: the `_id` is automatically indexed with a unique constraint, no need to call `ensureIndex` on it.

You can remove a previously created index with `datastore.removeIndex(fieldName, cb)`.

If your datastore is persistent, the indexes you created are persisted in the datafile, when you load the database a second time they are automatically created for you. No need to remove any `ensureIndex` though, if it is called on a database that already has the index, nothing happens.

```javascript
db.ensureIndex({ fieldName: 'somefield' }, function (err) {
  // If there was an error, err is not null
});

// Using a unique constraint with the index
db.ensureIndex({ fieldName: 'somefield', unique: true }, function (err) {
});

// Using a sparse unique index
db.ensureIndex({ fieldName: 'somefield', unique: true, sparse: true }, function (err) {
});


// Format of the error message when the unique constraint is not met
db.insert({ somefield: 'nedb' }, function (err) {
  // err is null
  db.insert({ somefield: 'nedb' }, function (err) {
    // err is { errorType: 'uniqueViolated'
    //        , key: 'name'
    //        , message: 'Unique constraint violated for key name' }
  });
});

// Remove index on field somefield
db.removeIndex('somefield', function (err) {
});

// Example of using expireAfterSeconds to remove documents 1 hour
// after their creation (db's timestampData option is true here)
db.ensureIndex({ fieldName: 'createdAt', expireAfterSeconds: 3600 }, function (err) {
});

// You can also use the option to set an expiration date like so
db.ensureIndex({ fieldName: 'expirationDate', expireAfterSeconds: 0 }, function (err) {
  // Now all documents will expire when system time reaches the date in their
  // expirationDate field
});

```

**Note:** the `ensureIndex` function creates the index synchronously, so it's best to use it at application startup. It's quite fast so it doesn't increase startup time much (35 ms for a collection containing 10,000 documents).


## Browser version
The browser version and its minified counterpart are in the `browser-version/out` directory. You only need to require `nedb.js` or `nedb.min.js` in your HTML file and the global object `Nedb` can be used right away, with the same API as the server version:

```
<script src="nedb.min.js"></script>
<script>
  var db = new Nedb();   // Create an in-memory only datastore
  
  db.insert({ planet: 'Earth' }, function (err) {
   db.find({}, function (err, docs) {
     // docs contains the two planets Earth and Mars
   });
  });
</script>
```

If you specify a `filename`, the database will be persistent, and automatically select the best storage method available (IndexedDB, WebSQL or localStorage) depending on the browser. In most cases that means a lot of data can be stored, typically in hundreds of MB. **WARNING**: the storage system changed between v1.3 and v1.4 and is NOT back-compatible! Your application needs to resync client-side when you upgrade NeDB.

NeDB is compatible with all major browsers: Chrome, Safari, Firefox, IE9+. Tests are in the `browser-version/test` directory (files `index.html` and `testPersistence.html`).

If you fork and modify nedb, you can build the browser version from the sources, the build script is `browser-version/build.js`.


## Performance
### Speed
NeDB is not intended to be a replacement of large-scale databases such as MongoDB, and as such was not designed for speed. That said, it is still pretty fast on the expected datasets, especially if you use indexing. On a typical, not-so-fast dev machine, for a collection containing 10,000 documents, with indexing:  
* Insert: **10,680 ops/s**
* Find: **43,290 ops/s**
* Update: **8,000 ops/s**
* Remove: **11,750 ops/s**  

You can run these simple benchmarks by executing the scripts in the `benchmarks` folder. Run them with the `--help` flag to see how they work.

### Memory footprint
A copy of the whole database is kept in memory. This is not much on the
expected kind of datasets (20MB for 10,000 2KB documents).

## Use in other services
* <a href="https://github.com/louischatriot/connect-nedb-session"
  target="_blank">connect-nedb-session</a> is a session store for
Connect and Express, backed by nedb
* If you mostly use NeDB for logging purposes and don't want the memory footprint of your application to grow too large, you can use <a href="https://github.com/louischatriot/nedb-logger" target="_blank">NeDB Logger</a> to insert documents in a NeDB-readable database
* If you've outgrown NeDB, switching to MongoDB won't be too hard as it is the same API. Use <a href="https://github.com/louischatriot/nedb-to-mongodb" target="_blank">this utility</a> to transfer the data from a NeDB database to a MongoDB collection
* An ODM for NeDB: <a href="https://github.com/scottwrobinson/camo" target="_blank">Camo</a>

## Pull requests
If you submit a pull request, thanks! There are a couple rules to follow though to make it manageable:
* The pull request should be atomic, i.e. contain only one feature. If it contains more, please submit multiple pull requests. Reviewing massive, 1000 loc+ pull requests is extremely hard.
* Likewise, if for one unique feature the pull request grows too large (more than 200 loc tests not included), please get in touch first.
* Please stick to the current coding style. It's important that the code uses a coherent style for readability.
* Do not include sylistic improvements ("housekeeping"). If you think one part deserves lots of housekeeping, use a separate pull request so as not to pollute the code.
* Don't forget tests for your new feature. Also don't forget to run the whole test suite before submitting to make sure you didn't introduce regressions.
* Do not build the browser version in your branch, I'll take care of it once the code is merged.
* Update the readme accordingly.
* Last but not least: keep in mind what NeDB's mindset is! The goal is not to be a replacement for MongoDB, but to have a pure JS database, easy to use, cross platform, fast and expressive enough for the target projects (small and self contained apps on server/desktop/browser/mobile). Sometimes it's better to shoot for simplicity than for API completeness with regards to MongoDB.

## Bug reporting guidelines
If you report a bug, thank you! That said for the process to be manageable please strictly adhere to the following guidelines. I'll not be able to handle bug reports that don't:
* Your bug report should be a self-containing gist complete with a package.json for any dependencies you need. I need to run through a simple `git clone gist; npm install; node bugreport.js`, nothing more.
* It should use assertions to showcase the expected vs actual behavior and be hysteresis-proof. It's quite simple in fact, see this example: https://gist.github.com/louischatriot/220cf6bd29c7de06a486
* Simplify as much as you can. Strip all your application-specific code. Most of the time you will see that there is no bug but an error in your code :)
* 50 lines max. If you need more, read the above point and rework your bug report. If you're **really** convinced you need more, please explain precisely in the issue.
* The code should be Javascript, not Coffeescript.

### Bitcoins
You don't have time? You can support NeDB by sending bitcoins to this address: 1dDZLnWpBbodPiN8sizzYrgaz5iahFyb1


## License 

See [License](LICENSE)


