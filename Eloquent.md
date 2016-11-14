# Laravel Eloquent使用小记

今天因为开发数据库业务中间层需要，开始研究Laravel Eloquent，因为刚开始使用laravel框架的时候，都是使用query，查询构建器来写sql类似于

```
DB::connection('mydb')->table('mylove')
						->where( 'name', 'guowan' )
						->get();
```

复杂一点的sql使用db::raw

```
DB::connection('mydb')->table('mylove')->select( DB::RAW( 'count("name") as mylovecount' ) )
                        ->where( 'name', 'guowan' )
                        ->get();
```

本着在工作中学习的态度开始研究Eloquent，对着laravel中文文档，开始设计Eloquent Model。这里给出表大概字段（因兼容老系统要求，表字段设计与当前业务不相符，这里不与讨论～）

### 表结构

```
CREATE TABLE `user_ext` (
  `user_id` 	int(10) 			NOT NULL,
  `realname` 	varchar(255) 		DEFAULT NULL,
  `gender` 		int(11) 			NOT NULL DEFAULT '0',
  `birthday` 	datetime 			DEFAULT NULL,
  `comefrom` 	varchar(255) 		DEFAULT NULL,
  `qq` 			varchar(255) 		DEFAULT NULL,
  `weibo` 		varchar(255) 		DEFAULT NULL,
  `blog` 		varchar(255) 		DEFAULT NULL,
  `mobile` 		varchar(255) 		DEFAULT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8



CREATE TABLE `user` (
  `user_id` 	int(10) NOT NULL AUTO_INCREMENT,
  `username` 	varchar(100) 	DEFAULT NULL,
  `email` 		varchar(255) 	DEFAULT NULL,
  `user_img` 	varchar(255) 	DEFAULT NULL,
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

### 创建Eloqueue Model

- user

```
<?php

namespace App\Http\Models\Eloquent;

use Illuminate\Database\Eloquent\Model;

class CUser extends Model
{
	/**
	 * 与模型关联的数据表。
	 *
	 * @var string
	 */
    protected $table = 'user';

	/*
	 * 数据库表主键
	 *
	 * @var string
	 */
	protected $primaryKey = 'user_id';

	/*
	 * 取消自动维护create_at,update_at字段
	 *
	 * @var string
	 */
	public $timestamps = false;


	/*
	 * 获取与指定用户相关联的扩展信息记录
	 */
	public function hasOneExt()
	{
		return $this->hasOne( 'App\Http\Models\Eloquent\CUserExt', 'user_id', 'user_id' );
	}
}

```

- user_ext

```
<?php

namespace App\Http\Models\Eloquent;

use Illuminate\Database\Eloquent\Model;

class CUserExt extends Model
{

	/**
	 * 与模型关联的数据表。
	 *
	 * @var string
	 */
	protected $table = 'ac_user_ext';

	/*
	 * 数据库表主键
	 *
	 * @var string
	 */
	protected $primaryKey = 'user_id';

	/*
	 * 取消自动维护create_at,update_at字段
	 *
	 * @var string
	 */
	public $timestamps = false;


	public function acUser()
	{
		return $this->belongsTo( 'App\Http\Models\Eloquent\CUser' );
	}
}

```

user与user_ext表为1对1关系。

### 注意

> user model中的hasOneExt方法，之所以使用hasOneExt方法名，是通过方法命名，在调用方法的时候就可以知道和userExt表的关系。

> hasOne函数，第一个参数是类路径；第二个参数外键，也就是userExt表的主键；第三个参数才是user表主键。自己使用的时候，没有指定第二个和第三个参数，会出现错误


### 问题

下面才是今天记录的主要内容，在使用过程中，出现一些问题，以及问题相应的解决方法，可能有些问题还没有解决或者解决的不好，这里记录一下，增加一下印象，也可以和其他同学一块讨论一下

#### 1. 依赖方法hasOneExt

调用下面方法

```
$oUser = CUser::find( $sUMId )->hasOneExt();
```

结果竟然返回UserExt表中数据。我的本意本来想做相应的关联查询，查出两个表的数据。然后在网上各种搜索Eloquent两表联查，返回两表字段。

最终解决方案如下：

```
$oUser = CAcUser::with( 'hasOneExt' )->find( $sUMId );
```

查询结果：

```
Array
(
    [user_id] => 1
    [username] => admin
    [email] => wanguowan521@163.com
    [user_img] => 201303/26132122j2lg.jpg
    [has_one_ext] => Array
        (
            [user_id] => 1
            [realname] => 瞌睡
            [gender] => 1
            [birthday] =>
            [comefrom] => **,不限
            [qq] =>
            [weibo] =>
            [blog] =>
            [mobile] =>
        )

)
```

这里依赖表数据使用方法名作为key成为返回结果的一部分，这个对于业务接口，需要一维数组的时候还得需要翻译。幸好对于业务层来说，希望屏蔽底层数据层字段细节，本来就需要做一次翻译，所以这里也就不是什么大问题。

这里with语法，是Eloquent中所谓的预加载语法，主要是为了解决ORM(Object Relation Mapping) n+1次查询问题--[详细说明][1]。在网上查询过程中，这里虽然是1对1关系，但是如果这样解决，会将一次join查询，变成两次查询，对于将来高并发场景来说，有点不能接受。但是不这样解决又没有找到其他解决方法。

无奈，尝试打印Eloquent执行的sql，查看具体的sql语句(打印laravel执行sql方法比较多，可以参考[资料][2]），代码如下：

```
DB::enableQueryLog();

$oUser = CUser::with( 'hasOneExt' )->find( $sUMId );

print_r(
	DB::getQueryLog()
);
```

打印结果如下：

```
Array
(
    [0] => Array
        (
            [query] => select * from `user` where `user`.`user_id` = ? limit 1
            [bindings] => Array
                (
                    [0] => 1
                )

            [time] => 0.56
        )

    [1] => Array
        (
            [query] => select * from `user_ext` where `user_ext`.`user_id` in (?)
            [bindings] => Array
                (
                    [0] => 1
                )

            [time] => 0.32
        )

)
```

可以看出，sql先根据user_id查询到主标数据，然后在去依赖表中做in查询，这样确实解决了ORM n+1次查询的问题，但是对于直接使用sql，还是多出一次查询。

这里发现一个比较有趣的事情，log里有一个time值，难道这个是sql执行时间，如果这个是执行时间的话，那就可以简单的验证一下sql执行效率问题了，然后开始查询资料，终于在源码中找到了答案，源码如下：[详细链接][3]

```
	/**
	 * Run a SQL statement and log its execution context.
	 *
	 * @param  string   $query
	 * @param  array    $bindings
	 * @param  Closure  $callback
	 * @return mixed
	 *
	 * @throws QueryException
	 */
	protected function run($query, $bindings, Closure $callback)
	{
		$start = microtime(true);
		// To execute the statement, we'll simply call the callback, which will actually
		// run the SQL against the PDO connection. Then we can calculate the time it
		// took to execute and log the query SQL, bindings and time in our memory.
		try
		{
			$result = $callback($this, $query, $bindings);
		}
		// If an exception occurs when attempting to run a query, we'll format the error
		// message to include the bindings with SQL, which will make this exception a
		// lot more helpful to the developer instead of just the database's errors.
		catch (\Exception $e)
		{
			throw new QueryException($query, $bindings, $e);
		}
		// Once we have run the query we will calculate the time that it took to run and
		// then log the query, bindings, and execution time so we will report them on
		// the event that the developer needs them. We'll log time in milliseconds.
		$time = $this->getElapsedTime($start);
		$this->logQuery($query, $bindings, $time);
		return $result;
	}

	/**
	 * Get the elapsed time since a given starting point.
	 *
	 * @param  int    $start
	 * @return float
	 */
	protected function getElapsedTime($start)
	{
		return round((microtime(true) - $start) * 1000, 2);
	}
```

这里可以看出time就是sql执行时间，而且单位是毫秒.

这里就可以测试单条join和使用eloquent with查询效率对比，代码如下：

```
DB::enableQueryLog();

DB::table(  'user' )
	->leftJoin( 'user_ext as ext', 'user.user_id', '=', 'ext.user_id' )
	->where( 'user.user_id', 1 )
	->get();

$oUser = CUser::with( 'hasOneExt' )->find( $sUMId );

print_r(
	DB::getQueryLog()
);
```

结果如下：

```
Array
(
    [0] => Array
        (
            [query] => select * from `user` as `user` left join `user_ext` as `ext` on `user`.`user_id` = `ext`.`user_id` where `user`.`user_id` = ?
            [bindings] => Array
                (
                    [0] => 1
                )

            [time] => 0.65
        )

    [1] => Array
        (
            [query] => select * from `user` where `user`.`user_id` = ? limit 1
            [bindings] => Array
                (
                    [0] => 1
                )

            [time] => 0.35
        )

    [2] => Array
        (
            [query] => select * from `user_ext` where `user_ext`.`user_id` in (?)
            [bindings] => Array
                (
                    [0] => 1
                )

            [time] => 0.35
        )

)
```

从结果可以看出，执行一条时间相比执行两条时间，差距不是很大，但是客观来说，这说明不了什么问题；首先，测试基于本地数据库，一次请求和两次请求的网络影响及延迟会比线上差距要小很多；其次，本地测试数据库，两个表数据量都在1k，数据量太小，无法反应真实线上数据查询效率。所以这里查询结果仅供参考，后期详细结果，会在本地伪造100w左右数据量进行测试观察，并咨询公司dba，对于大数据量对连表查询效率影响情况。

### 总结

对于今天解决问题的过程，虽然感觉没有得到完美的答案，但是在查询过程中也学习到不少东西，在这里做一下记录。以备后期温故学习。

这里记录一下几个小细节：

>数据库查询过程中，为了节省应用服务器与数据库服务器之间网络流量及数据库服务器的IO，数据库查询原则是只查询返回有用字段，对于无用的大字段，特别是text等，不需要时，尽量不查询。数据库查询尽量不要使用selec *

Eloquent 联合查询指定字段

- 方法1

```
$oUser = CUser::with( [ 'hasOneExt' => function( $query ) {
		$query->select( 'user_id', 'realname', 'gender', 'birthday' );
		} ] )->find( $sUMId, [ 'user_id', 'username', 'email', 'user_img' ] );
		
```
其中$query->select( 'user_id', 'realname', 'gender', 'birthday' )为查询扩展表字段；find( $sUMId, [ 'user_id', 'username', 'email', 'user_img' ] )为查询主表字段

- 方法2

```
public function hasOneExt() {
	return $this->hasOne( 'App\Http\Models\Eloquent\CUserExt', 'user_id', 'user_id' )
	->select( 'user_id', 'realname', 'gender', 'birthday' );
}
	
	
$oUser = CUser::with( 'hasOneExt' )->find( $sUMId, [ 'user_id', 'username', 'email', 'user_img' ] );	
```

执行sql结果：

```
Array
(
	[0] => Array
        (
            [query] => select `user_id`, `username`, `email`, `user_img` from `user` where `user`.`user_id` = ? limit 1
            [bindings] => Array
                (
                    [0] => 1
                )

            [time] => 0.5
        )

    [1] => Array
        (
            [query] => select `user_id`, `realname`, `gender`, `birthday` from `user_ext` where `user_ext`.`user_id` in (?)
            [bindings] => Array
                (
                    [0] => 1
                )

            [time] => 0.33
        )

)
```




---------------
[1]: https://laravel-china.org/docs/5.1/eloquent-relationships#eager-loading
[2]: http://stackoverflow.com/questions/27753868/how-to-get-the-query-executed-in-laravel-5-dbgetquerylog-returning-empty-arr
[3]: https://github.com/laravel/framework/blob/da5bdc94574d796b136d607365619506ca67ac50/src/Illuminate/Database/Connection.php#L592