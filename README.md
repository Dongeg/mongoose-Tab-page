# mongoose-Tab-page
mongoose分页查询

```js
//../db/dbHelper
var mongoose = require('mongoose');
var Schema = mongoose.Schema;
var async = require('async');

var pageQuery = function (page, pageSize, Model, populate, queryParams, sortParams, callback) {
    var start = (page - 1) * pageSize;
    var $page = {
        pageNumber: page
    };
    async.parallel({
        count: function (done) {  // 查询数量
            Model.count(queryParams).exec(function (err, count) {
                done(err, count);
            });
        },
        records: function (done) {   // 查询一页的记录
            Model.find(queryParams).skip(start).limit(pageSize).populate(populate).sort(sortParams).exec(function (err, doc) {
                done(err, doc);
            });
        }
    }, function (err, results) {
        var count = results.count;
        $page.pageCount = (count - 1) / pageSize + 1;
        $page.results = results.records;
        callback(err, $page);
    });
};

module.exports = {
    pageQuery: pageQuery
};

```
调用
```
var dbHelper = require('../db/dbHelper');

router.get('/', function(req, res, next){
    var page = req.query.page || 1;
    var Article = mongoose.model('Article', {xxx:xxx});
    dbHelper.pageQuery(page, 10, Article, '', {}, {
        created_time: 'desc'
    }, function(error, $page){
        if(error){
            next(error);
        }else{
            res.render('index'{
                records: $page.results,
                pageCount : $page.pageCount,  //共多少页
                pageNumber:$page.pageNumber: 当前第几页（从1 开始算）                    
                results:  $page.results: 当前页的记录
            })
        }
    });
})
```




