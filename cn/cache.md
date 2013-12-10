# 缓存

- [配置](#configuration)
- [缓存用法](#cache-usage)
- [增加 & 减少](#increments-and-decrements)
- [Cache Tags](#cache-tags)
- [数据库缓存](#database-cache)

<a name="configuration"></a>
## 配置

Laravel 对不同的缓存机制提供了一套统一的API。缓存配置信息存放于`app/config/cache.php`文件。在该配置文件中，你可以指定整个应用程序所使用的缓存驱动器。Laravel自身支持大多数流行的缓存服务器，例如[Memcached](http://memcached.org)和[Redis](http://redis.io)。

缓存配置文件还包含了其他配置项，文件里都有详细说明，因此，请务必查看这些配置项和其描述信息。默认情况下，Laravel被配置为使用`file`缓存驱动，它将数据序列化，并存放于文件系统中。在大型应用中，强烈建议使用基于内存的缓存系统，例如Memcached或APC。

<a name="cache-usage"></a>
## 缓存用法

**将某一数据存入缓存**

	Cache::put('key', 'value', $minutes);

**Using Carbon Objects To Set Expire Time**

	$expiresAt = Carbon::now()->addMinutes(10);

	Cache::put('key', 'value', $expiresAt);

**当某一数据不在缓存中是才将其保存**

	Cache::add('key', 'value', $minutes);

**检查缓存中是否有某个key对应的数据**

	if (Cache::has('key'))
	{
		//
	}

**从缓存中取得数据**

	$value = Cache::get('key');

**从缓存中取得数据，如果数据不存，则返回指定的默认值**

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

**将数据永久地存于缓存中**

	Cache::forever('key', 'value');

有时你可能想从缓存中取得某项数据，但是还希望在数据不存在时存储一项默认值。那就可以通过 `Cache::remember`方法实现：

	$value = Cache::remember('users', $minutes, function()
	{
		return DB::table('users')->get();
	});

还可以将`remember`和`forever`方法结合使用：

	$value = Cache::rememberForever('users', function()
	{
		return DB::table('users')->get();
	});

注意：所有存在于缓存中的数据都是经过序列化的，因此，你可以存储任何类型的数据。

**从缓存中删除某项数据**

	Cache::forget('key');

<a name="increments-and-decrements"></a>
## 增加 & 减少

除了`文件`和`数据库`驱动器，其他驱动器都支持`增加`和`减少`操作：

**让某个值增加**

	Cache::increment('key');

	Cache::increment('key', $amount);

**让某个值减少**

	Cache::decrement('key');

	Cache::decrement('key', $amount);

<a name="cache-tags"></a>
## Cache Tags

> **Note:** Cache tags are not supported when using the `file` or `database` cache drivers. Furthermore, when using multiple tags with caches that are stored "forever", performance will be best with a driver such as `memcached`, which automatically purges stale records.

Cache tags allow you to tag related items in the cache, and then flush all caches tagged with a given name. To access a tagged cache, use the `tags` method:

**Accessing A Tagged Cache**

You may store a tagged cache by passing in an ordered list of tag names as arguments, or as an ordered array of tag names.

	Cache::tags('people', 'authors')->put('John', $john, $minutes);

	Cache::tags(array('people', 'artists'))->put('Anne', $anne, $minutes);

You may use any cache storage method in combination with tags, including `remember`, `forever`, and `rememberForever`. You may also access cached items from the tagged cache, as well as use the other cache methods such as `increment` and `decrement`:

**Accessing Items In A Tagged Cache**

To access a tagged cache, pass the same ordered list of tags used to save it.

	$anne = Cache::tags('people', 'artists')->get('Anne');

	$john = Cache::tags(array('people', 'authors')->get('John);

You may flush all items tagged with a name or list of names. For example, this statement would remove all caches tagged with either `people`, `authors`, or both. So, both "Anne" and "John" would be removed from the cache:

	Cache::tags('people', 'authors')->flush();

In contrast, this statement would remove only caches tagged with 'author', so "John" would be removed, but not "Anne".

	Cache::tags('authors')->flush();

<a name="database-cache"></a>
## 数据库缓存

当使用`数据库`缓存驱动时，你需要设置一个数据表来缓存数据。以下案例是`Schema`表的定义：

	Schema::create('cache', function($table)
	{
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});

译者：王赛  [（Bootstrap中文网）](http://www.bootcss.com)