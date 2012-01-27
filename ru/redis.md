\thispagestyle{empty}
\changepage{}{}{}{-0.5cm}{}{2cm}{}{}{}
![The Little Redis Book, By Karl Seguin](title.png)\ 

\clearpage
\changepage{}{}{}{0.5cm}{}{-2cm}{}{}{}

## Об этой книге

### Лицензия

The Little Redis Book (Маленькая книга о Redis) распространяется под лицензией Attribution-NonCommercial 3.0 Unported. Вы не обязаны платить за эту книгу.

Вы можете свободно копировать, распространять, изменять и публиковать книгу. Тем не менее, я прошу, чтобы вы всегда указывали мое, Карла Сегуина (Karl Seguin), авторство и не использовали книгу в коммерческих целях.

Полный текст лицензии вы можете найти по ссылке:

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>

### Об авторе

Карл Сегуин - разработчик, имеющий опыт работы в разных областях и с разными технологями. Активный участник проектов Открытого/Свободного ПО, технический писатель и участник различных конференций. Является автором различных статей о Redis и нескольких утилит для работы с этой системой. На Redis работает система ранжирования и статистики на его бесплатном сервисе для разработчиков казуальных игр - [mogade.com](http://mogade.com/).

Карл написал [The Little MongoDB Book](http://openmymind.net/2011/3/28/The-Little-MongoDB-Book/), популярную бесплатную книгу о MongoDB.

Вы можете найти его блог по адресу <http://openmymind.net>, а также следить за ним в Twitter: [@karlseguin](http://twitter.com/karlseguin)

### Благодарности

Особая благодарность выражается  Перри Нилу ([Perry Neal](https://twitter.com/perryneal)) за его зоркий взгляд, ум и страсть. Ты оказал мне неоценимую помощь. Спасибо.

### Актуальная версия

Актуальная версия книги доступна по адресу:
<http://github.com/karlseguin/the-little-redis-book>

\clearpage

## Введение

За последние пару лет технологии и средства хранения и доступа к данным развивались невероятными темпами. Можно смело сказать, что реляционные базы данных не собираются исчезать, но в то же время мы видим, что экосистема вокруг данных уже никогда не будет прежней.

Из всех новых инструментов и решений, лично для меня, Redis оказался наиболее интересным. Почему? Во-первых, из-за невероятной простоты изучения. Час является самой подходящей единицей измерения, когда мы говорим о времени, необходимом и достаточном для ознакомления с Redis. Во-вторых, эта система решает специфический класс задач, будучи в то же время достаточно универсальной. Что это значит? Redis не пытается охватить весь спект задач, касающийся работы с данными. По мере знакомства с Redis будет все более очевидно, что может и чего не может эта система. И, если она может что-либо, для вас, как для разработчика, это будет приятным опытом.

Вы можете построить целую систему, используя только Redis. Однако, я думаю, что большинство людей обнаружит, что Redis дополняет их классические решение для работы с даными, будь то реляционная СУБД, документо-ориентированная система или что-то еще. Это тот тип систем, которые вы используете для решения специфических задач. В этом смысле Redis близка к индексирующему движку. Вы не будете писать ваше приложение полностью на Lucene, но если вам нужна хорошая система поиска, она подарит вам полезный опыт. Конечно, сходства между Redis и поисковыми движками на этом заканчиваются.

Цель этой книги состоит в описании базиса, необходимого для работы с Redis. Мы сфокусируем усилия на изучении пяти структур данных Redis и рассмотрим различные подходы к моделированию. Также, будут затронуты некоторые ключевые вопросы администрирования и отладки.

## Начало работы

Мы все учимся по-разному: кто-то предпочитает собственный опыт, кто-то смотрит видео, кто-то читает. Ничто не поможет вам понять Redis лучше, чем реальная работа. Redis легко устанавливается. В комплект установки входит простая командная оболочка, предоставляющая все необходимые возможности. Давайте потратим пару минут на то, чтобы установить и запустить Redis у себя на компьютере.

### Windows

Redis официально не поддерживает Windows, но есть возможности запустить ее на этой системе. Вы вряд ли станете так делать на рабочем сервере, но во время разработки я не сталкивался с какими-либо ограничениями.

Сначала перейдите на <https://github.com/dmajkic/redis/downloads> и скачайте последнюю версию (вверху списка).

Распакуйте архив и, в зависимости от архитектуры вашего компьютера, откройте директорию `64bit` или `32bit`.

### *nix и MacOSX

Для пользователей *nix и Mac, сборка Redis из исходников является лучшим вариантом. Инструкции, вместе с номером последней версии, доступны по адресу <http://redis.io/download>. На момент написания этих строк последней версией является 2.4.6. Для установки выполните:

	wget http://redis.googlecode.com/files/redis-2.4.6.tar.gz
	tar xzf redis-2.4.6.tar.gz
	cd redis-2.4.6
	make

(В качестве альтернативы, Redis доступна через различные мнеджеры пакетов. Например, пользователи MacOSX с установленным Homebrew могут просто выполнить команду `brew install redis`.) (*Установка через менеджер пакетов может оказаться гораздо более удачным решением с точки зрения сохранения правильной конфигурации вашей системы - Прим. перев.*)

Если вы собрали систему из исходных текстов, исполняемые файлы можно найти в директории `src`. перейдите в эту директорию, введя команду `cd src`.

### Запуск и подключение к Redis

Если все прошло успешно, исполняемые файлы Redis должны быть в вашем распоряжении. Redis имеет множество исполняемых файлов. Мы сосредоточимся на сервере и командной оболочке (DOS-подобный клиент). Давайте начнем с сервера. В Windows, двойным кликом запустите `redis-server`. В *nix/MacOSX выполните команду `./redis-server` в терминале.

Если вы прочитаете стартовое сообщение, вы увидите предупреждение о том, что файл `redis.conf` не обнаружен. Виесто этого Redis воспользуется настройками по умолчанию, что вполне подойдет для наших целей.

Далее, запустите командную оболочку Redis двойным кликом на `redis-cli` (Windows) или командой `./redis-cli` (*nix/MacOSX). Таким образом создастся подключение к локальному серверу на 6389-м порту по умолчанию.

Вы можете убедиться, что все работает, введя команду `info` в командную строку. Вы должны увидеть множество пар ключ-значение, которые предоставляют большое количество информации о состоянии сервера.

Если у вас возникли проблемы с выполнением описанных выше действий, я рекомендую поискать помощь на [официальных форумах поддержки Redis](https://groups.google.com/forum/#!forum/redis-db).

## Драйверы Redis

Как вы скоро узнаете, API Redis лучше всего описать как явный набор функций в простом процедурном стиле. Это значит, что, используете ли вы командную оболочку или драйвер вашего любимого языка программирования, все работает одинаковым образом. Следовательно, у вас не должно возникнуть проблем, если вы решите работать посредством языка программирования.

\clearpage

## Раздел 1 - Основы

Почему же Redis такой особенный? Какие задачи он решает? На что следует обращать внимание разработчикам? Перед тем, как ответить на все эти вопросы, мы должны понять, что же все-таки это такое - Redis. 

Очень часто Redis описывают как хранилище данных типа ключ-значение, которое постоянно находится в памяти с гарантией сохранности данных. Я не думаю, что это совсем точное определение. Redis, действительно, держит все данные в памяти (позже мы вернемся к этому), и он сбрасывает данные на диск для устойчивости к сбоям. Но он не просто хранилище данных типа ключ-значение. Очень важно разобраться в этом неточном определении. В противном случае, мы не сможем раскрыть всех возможностей Redis.

Суть в том, что Redis предоставляет пять разных структур данных, только одна из которых собственно и есть структура типа ключ-значение. Понимание этих пяти структур данных, как они работают, какие методы они представляют для взаимодействия, и что вы сможете сделать с их помощью, как раз и являются путем к пониманию Redis. Но сначала давайте рассмотрим что же все-таки значит - представление структуры данных.

Если мы применим эту концепцию к реляционному миру, мы можем сказать, что базы данных предоставляют один тип структур данных - таблицы. Таблицы, по сути, сложные и гибкие. Существует очень мало задач, которые нельзя решить с помощью таблиц. Тем не менее, они не идеальны. А именно, они не на столько простые, или не на столько быстрые, на сколько могли бы быть. Что, если вместо универсальной структуры мы бы использовали специализированные структуры? В этом, наверное, найдутся проблемы, которые мы не сможем решить (или, по крайней мере, это решение будет спорным). Однако, в любом случае, мы выиграем на простоте и скорости.

Использование специфичных структур данных для специфичных задач? Разве это не тот подход, который мы постоянно используем при программировании? Вы не используете хэш-таблицы для всех данных программы, ???? тоже самое и с скалярными переменными ????. Я считаю, в этом и состоит подход Redis. Если вы работаете со скалярами, списками, или множествами, почему бы не хранить их как скаляры, списки, хэши и множества. Почему проверка на существование, должна быть сложнее чем просто вызов `exists(key)` или медленнее чем 0(1) (постоянное время выполнения выражения, которое не зависит от количества записей в хранилище)?


## Составные Кирпичики

### Базы Данных

Redis использует знакомую всем концепцию базы данных. База данных содержит набор данных. Типичное предназначение базы данных - это группирование всей информации определенного приложения в месте и изоляция ее от других приложений. 

В Redis база данных идентифицируется просто числом, которое по умолчанию равняется `0`. Если вы хотите сменить базу данных, то вы можете сделать это командой `select`. В командной строке просто введите `select 1`. Redis должен ответить сообщением `OK` и в терминале вы должны увидеть что-то типа `redis 127.0.0.1:6379[1]>`. Если вы хотите переключиться обратно на базу по умолчанию, просто введите в командной строке `select 0`.


### Команды, Ключи и Значения

Несмотря на то, что Redis больше, чем просто хранилище типа ключ-значение, в его основе каждая из пяти используемых структур данных имеет, как минимум, ключ и значение. Очень важно разобраться в том, что такое ключи и что такое значения, перед тем как двигаться дальше.

Ключ - это то, чем мы помечаем части информации. Мы будем пользоваться ключами часто, но пока достаточно знать, что ключ может выглядеть вот так `users:leto`. Под таким ключем можно ожидать информацию о пользователе под именем `leto`. Двоеточие не имеет никакого особого значения, но использование подобных разделителей является хорошим тоном.

Значение - это данные, которые ассоциированы с ключем. Это может быть что угодно. Иногда это строки, иногда числа, а иногда там хранятся сериализованные объекты (в виде JSON, XML или любых других форматов). В основном, Redis рассматривает значение как массив байт и не интересуется о том, что они собой представляют. Обратите внимание, что разные драйверы производят сериализацию по-разному. Поэтому, здесь мы будем говорить только о строках (string), целых числах (integer) и JSON.

Давайте запачкаем немного руки. И выполним следующие команды:

	set users:leto "{name: leto, planet: dune, likes: [spice]}"

Почти все команды Redis выглядят подобным образом. Сначала идет сама команда, в нашем случае `set`, а потом параметры. Команда `set` принимает два параметра: ключ, который мы сохраняем и значение, которое связано с ним. Многие, но не все, команды принимают ключ (это обычно первый параметр). А теперь угадайте, как получить это значение? Надеюсь, что вы догадались (не расстраивайтесь, если это не так):

	get users:leto

Попробуйте самостоятельно поиграть с подобными комбинациями. Ключ и значение - это основная концепция, и  простейший способ работать с ней - это команды `get` и `set`. Создайте других пользователей, попробуйте разные виды ключей и значений.

### Запросы

По мере нашего продвижения вперед, две вещи станет ясно. Что ключи это - все, а значения - ничто. Или, другими словами, Redis не позволяет использовать в запросах значения объектов. С вышесказанного выплывает что мы не сможем найти пользователя который живет на планете `dune`. 

Для многих это может вызвать некоторые беспокойства. Мы живем в мире где запросы к базам данных на столько гибкие и мощные, что подход Redis' кажется примитивным и не практичным. Не дайте завести себя в заблуждение. Redis не является универсальным решением на все случаи жизни. Здесь есть вещи которые просто не решаются с помощью Redis (из-за ограничений в запросах). Также примите к сведению, что в некоторых случаях, вы найдете новые способы создания модели данных.

Мы россмотрим более конкретные примеры по мере продвижения, но очень важно чтобы мы понимали эти базовые принципы Redis. Это позволит понять, как значение может быть важным - в то время как Redis не нужно читать или понимать его. Также это поможет нам научится как строить модель данных в этом новом мире.

### Memory and Persistence

We mentioned before that Redis is an in-memory persistent store. With respect to persistence, by default, Redis snapshots the database to disk based on how many keys have changed. You configure it so that if X number of keys change, then save the database every Y seconds. By default, Redis will save the database every 60 seconds if 1000 or more keys have changed all the way to 15 minutes if only a 9 keys has changed.

Alternatively (or in addition to snapshotting), Redis can run in append mode. Any time a key changes, an append-only file is updated on disk. In some cases it's acceptable to lose 60 seconds worth of data, in exchange for performance, should there be some hardware or software failure. In some cases such a loss is not acceptable. Redis gives you the option. In chapter 5 we'll see a third option, which is offloading persistence to a slave.

With respect to memory, Redis keeps all your data in memory. The obvious implication of this is the cost of running Redis: RAM is still the most expensive part of server hardware. 

I do feel that some developers have lost touch with how little space data can take. The Complete Works of William Shakespeare takes roughly 5.5MB of storage. As for scaling, other solutions tend to be IO- or CPU-bound. Which limitation (RAM or IO) will require you to scale out to more machines really depends on the type of data and how you are storing and querying it. Unless you're storing large multimedia files in Redis, the in-memory aspect is probably a non-issue. For apps where it is an issue you'll likely be trading being IO-bound for being memory bound.

Redis did add support for virtual memory. However, this feature has been seen as a failure (by Redis' own developers) and its use has been deprecated. 

(On a side note, that 5.5MB file of Shakespeare's complete works can be compressed down to roughly 2MB. Redis doesn't do auto-compression but, since it treats values as bytes, there's no reason you can't trade processing time for RAM by compressing/decompressing the data yourself.

### Putting It Together

We've touched on a number of high level topics. The last thing I want to do before diving into Redis is bring some of those topics together. Specifically, query limitations, data structures and Redis' way to store data in memory.

When you add those three things together you end up with something wonderful: speed. Some people think "Of course Redis is fast, everything's in memory." But that's only part of it. The real reason Redis shines versus other solutions is its specialized data structures. 

How fast? It depends on a lot of things - which commands you are using, the type of data, and so on. But Redis' performance tends to be measured in tens of thousands, or hundreds of thousands of operations **per second**. You can run `redis-benchmark` (which is in the same folder as the `redis-server` and `redis-cli`) to test it out yourself. 

I once changed code which used a traditional model to using Redis. A load test I wrote took over 5 minutes to finish using the relational model. It took about 150ms to complete in Redis. You won't always get that sort of massive gain, but it hopefully gives you an idea of what we are talking about

It's important to understand this aspect of Redis because it impacts how you interact with it. Developers with an SQL background often work at minimizing the number of round trips they make to the database. That's good advice for any system, including Redis. However, given that we are dealing with simpler data structures, we'll sometimes need to hit the Redis server multiple times to achieve our goal. Such data access patterns can feel unnatural at first, but in reality it tends to be an insignificant cost compared to the raw performance we gain.

### In This Chapter

Although we barely got to play with Redis, we did cover a wide range of topics. Don't worry if something isn't crystal clear - like querying. In the next chapter we'll go hands-on and any questions you have will hopefully answer themselves. 

The important takeaway from this chapters are:

* Keys are strings which identify pieces of data (values)

* Values are arbitrary byte arrays that Redis doesn't care about

* Redis exposes (and is implemented as) five specialized data structures

* Combined, the above make Redis fast and easy to use, but not suitable for every scenario

\clearpage

## Chapter 2 - The Data Structures

It's time to look at Redis' five data structures. We'll explain what each data structure is, what methods are available and what type of feature/data you'd use it for.

The only Redis constructs we've seen so far are commands, keys and values. So far, nothing about data structures has been concrete. When we used the `set` command, how did Redis know what data structure to use? It turns out that every command is specific to a data structure. For example when you use `set` you are storing the value in a string data structure. When you use `hset` you are storing it in a hash. Given the small size of Redis' vocabulary, it's quite manageable.

**[Redis' website](http://redis.io/commands) has great reference documentation. There's no point in repeating the work they've already done. We'll only cover the most important commands needed to understand the purpose of a data structure.**

There's nothing more important than having fun and trying things out. You can always erase all the values in your database by entering `flushdb`, so don't be shy and try doing crazy things!

### Strings

Strings are the most basic data structures available in Redis. When you think of a key-value pair, you are thinking of strings. Don't get mixed up by the name, as always, your value can be anything. I prefer to call them "scalars", but maybe that's just me.

We already saw a common use-case for strings, storing instances of objects by key. This is something that you'll make heavy use of:

	set users:leto "{name: leto, planet: dune, likes: [spice]}"

Additionally, Redis lets you do some common operations. For example `strlen <key>` can be used to get the length of a key's value; `getrange <key> <start> <end>` returns the specified range of a value; `append <key> <value>` appends the value to the existing value (or creates it if it doesn't exist already). Go ahead and try those out. This is what I get:
	
	> strlen users:leto
	(integer) 42
	
	> getrange users:leto 27 40
	"likes: [spice]"
	
	> append users:leto " OVER 9000!!"
	(integer) 54

Now, you might be thinking, that's great, but it doesn't make sense. You can't meaningfully pull a range out of JSON or append a value. You are right, the lesson here is that some of the commands, especially with the string data structure, only make sense given specific type of data. 

Earlier we learnt that Redis doesn't care about your values. Most of the time that's true. However, a few string commands are specific to some types or structure of values. As a vague example, I could see the above `append` and `getrange` commands being useful in some custom space-efficient serialization. As a more concrete example I give you the `incr`, `incrby`, `decr` and `decrby` commands. These increment or decrement the value of a string:

	> incr stats:page:about
	(integer) 1
	> incr stats:page:about
	(integer) 2
	
	> incrby ratings:video:12333 5
	(integer) 5
	> incrby ratings:video:12333 3
	(integer) 8

As you can imagine, Redis strings are great for analytics. Try incrementing `users:leto` (a non-integer value) and see what happens (you should get an error).

A more advanced example is the `setbit` and `getbit` commands. There's a [wonderful post](http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps/) on how Spool uses these two commands to efficiently answer the question "how many unique visitors did we have today". For 128 million users a laptop generates the answer in less than 50ms and takes only 16MB of memory. 

It isn't important that you understand how bitmaps work, or how Spool uses them, but rather to understand that Redis strings are more powerful than they initially seem. Still, the most common cases are the ones we gave above: storing objects (complex or not) and counters. Also, since getting a value by key is so fast, strings are often used to cache data.

### Hashes

Hashes are a good example of why calling Redis a key-value store isn't quite accurate. You see, in a lot of ways, hashes are like strings. The important difference is that they provide an extra level of indirection: a field. Therefore, the hash equivalents of `set` and `get` are:

	hset users:goku powerlevel 9000
	hget users:goku powerlevel

We can also set multiple fields at once, get multiple fields at once, get all fields and values, list all the fields or delete a specific field:

	hmset users:goku race saiyan age 737
	hmget users:goku race powerlevel
	hgetall users:goku
	hkeys users:goku
	hdel users:goku age

As you can see, hashes give us a bit more control over plain strings. Rather than storing a user as a single serialized value, we could use a hash to get a more accurate representation. The benefit would be the ability to pull and update/delete specific pieces of data, without having to get or write the entire value. 

Looking at hashes from the perspective of a well-defined object, such as a user, is key to understanding how they work. And it's true that, for performance reasons, more granular control might be useful. However, in the next chapter we'll look at how hashes can be used to organize your data and make querying more practical. In my opinion, this is where hashes really shine.

### Lists

Lists let you store and manipulate an array of values for a given key. You can add values to the list, get the first or last value and manipulate values at a given index. Lists maintain their order and have efficient index-based operations. We could have a `newusers` list which tracks the newest registered users to our site:

	rpush newusers goku
	ltrim newusers 0 50

First we push a new user at the front of the list, then we trim it so that it only contains the last 50 users. This is a common pattern. `ltrim` is an O(N) operation, where N is the number of values we are removing. In this case, where we always trim after a single insert, it'll actually have a constant performance of O(1) (because N will always be equal to 1).

This is also the first time that we are seeing a value in one key referencing a value in another. If we wanted to get the details of the last 10 users, we'd do the following combination:

	keys = redis.lrange('newusers', 0, 10)
	redis.mget(*keys.map {|u| "users:#{u}"})

The above is a bit of Ruby which shows the type of multiple roundtrips we talked about before.

Of course, lists aren't only good for storing references to other keys. The values can be anything. You could use lists to store logs or track the path a user is taking through a site. If you were building a game, you might use it to track a queued user actions.

### Sets

Set are used to store unique values and provide a number of set-based operations, like unions. Sets aren't ordered but they provide efficient value-based operations. A friend's list is the classic example of using a set:

	sadd friends:leto ghanima paul chani jessica
	sadd friends:duncan paul jessica alia

Regardless of how many friends a user has, we can efficiently tell (O(1)) whether userX is a friend of userY or not

	sismember friends:leto jessica
	sismember friends:leto vladimir

Furthermore we can see what two or more people share the same friends:

	sinter friends:leto friends:duncan

and even store the result at a new key:

	sinterstore friends:leto_duncan friends:leto friends:duncan

Sets are great for tagging or tracking any other properties of a value for which duplicates don't make any sense (or where we want to apply set operations such as intersections and unions).

### Sorted Sets

The last and most powerful data structure are sorted sets. If hashes are like strings but with fields, then sorted sets are like sets but with a score. The score provides sorting and ranking capabilities. If we wanted a ranked list of friends, we might do:

	zadd friends:leto 100 ghanima 95 paul 95 chani 75 jessica 1 vladimir

Want to find out how many friends `leto` has with a rank of 90 or over?

	zcount friends:leto 90 100

How about figuring out `chani`'s rank?

	zrevrank friends:leto chani

We use `zrevrank` instead of `zrank` since Redis' default sort is from low to high (but in this case we are ranking from high to low). The most obvious use-case for sorted sets is a leaderboard system. In reality though, anything you want sorted by an some integer, and be able to efficiently manipulate based on that score, might be a good fit for a sorted set. 

In the next chapter we'll look at how sorted sets can be used for tracking events which are time-based (where time is the score); which is another common use-case.

### In This Chapter

That's a high level overview of Redis' five data structures. One of the neat things about Redis is that you can often do more than you first realize. There are probably ways to use string and sorted sets that no one has thought of yet. As long as you understand the normal use-case though, you'll find Redis ideal for all types of problems. Also, just because Redis exposes five data structures and various methods, don't think you need to use all of them. It isn't uncommon to build a feature while only using a handful of commands.

\clearpage

## Chapter 3 - Leveraging Data Structures

In the previous chapter we talked about the five data structures and gave some examples of what problems they might solve. Now it's time to look at a few more advanced, yet common, topics and design patterns.

### Big O Notation

Throughout this book we've made references to the Big O notation in the form of O(n) or O(1). Big O notation is used to explain how something behaves given a certain number of elements. In Redis, it's used to tell us how fast a command is based on the number of items we are dealing with.

Redis documentation tells us the Big O notation for each of its commands. It also tells us what the factors are that influence the performance. Let's look at some examples.

The fastest anything can be is O(1) which is a constant. Whether we are dealing with 5 items or 5 million, you'll get the same performance. The `sismember` command, which tells us if a value belongs to a set, is O(1). `sismember` is a powerful command, and its performance characteristics are a big reason for that. A number of Redis commands are O(1).

Logarithmic, or O(log(N)), is the next fastest possibility because it needs to scan through smaller and smaller partitions. Using this type of divide and conquer approach, a very large number of items quickly gets broken down in a few iterations. `zadd` is a O(log(N)) command, where N is the number of elements already in the set.

Next we have linear commands, or O(N). Looking for a non-indexed row in a table is an O(N) operation. So is using the `ltrim` command. However, in the case of `ltrim`, N isn't the number of elements in the list, but rather the elements being removed. Using `ltrim` to remove 1 item from a list of millions will be faster than using `ltrim` to remove 10 items from a list of thousands. (Though they'll probably both be so fast that you wouldn't be able to time it.)

`zremrangebyscore` which removes elements from a sorted set with a score between a minimum and a maximum value has a complexity of O(log(N)+M). This makes it a mix. By reading the documentation we see that N is the number of total elements in the set and M is the number of elements to be removed. In other words, the number of elements that'll get removed is probably going to be more significant, in terms of performance, than the total number of elements in the list.

The `sort` command, which we'll discuss in greater detail in the next chapter has a complexity of O(N+M*log(M)). From its performance characteristic, you can probably tell that this is one of Redis' most complex commands.

There are a number of other complexities, the two remaining common ones are O(N^2) and O(C^N). The larger N is, the worse these perform relative to a smaller N. None of Redis' commands have this type of complexity.

It's worth pointing out that the Big O notation deals with the worst case. When we say that something takes O(N), we might actually find it right away or it might be the last possible element.


### Pseudo Multi Key Queries

A common situation you'll run into is wanting to query the same value by different keys. For example, you might want to get a user by email (for when they first log in) and also by id (after they've logged in). One horrible solution is to duplicate your user object into two string values:

	set users:leto@dune.gov "{id: 9001, email: 'leto@dune.gov', ...}"
	set users:9001 "{id: 9001, email: 'leto@dune.gov', ...}"

This is bad because it's a nightmare to manage and it takes twice the amount of memory.

It would be nice if Redis let you link one key to another, but it doesn't (and it probably never will). A major driver in Redis' development is to keep the code and API clean and simple. The internal implementation of linking keys (there's a lot we can do with keys that we haven't talked about yet) isn't worth it when you consider that Redis already provides a solution: hashes.

Using a hash, we can can remove the need for duplication:
	
	set users:9001 "{id: 9001, email: leto@dune.gov, ...}"
	hset users:lookup:email leto@dune.gov 9001

What we are doing is using the field as a pseudo secondary index and referencing the single user object. To get a user by id, we issue a normal `get`:

	get users:9001

To get a user by email, we issue an `hget` followed by a `get` (in Ruby):

	id = redis.hget('users:lookup:email', 'leto@dune.gov')
	user = redis.get("users:#{id}")

This is something that you'll likely end up doing often. To me, this is where hashes really shine, but it isn't an obvious use-case until you see it.

### References and Indexes

We've seen a couple examples of having one value reference another. We saw it when we looked at our list example, and we saw it in the section above when using hashes to make querying a little easier. What this comes down to is essentially having to manually manage your indexes and references between values. Being honest, I think we can say that's a bit of a downer, especially when you consider having to manage/update/delete these references manually. There is no magic solution to solving this problem in Redis. 

We already saw how sets are often used to implement this type of manual index:

	sadd friends:leto ghanima paul chani jessica

Each member of this set is a reference to a Redis string value containing details on the actual user. What if `chani` changes her name, or deletes her account? Maybe it would make sense to also track the inverse relationships:

	sadd friends_of:chani leto paul

Maintenance cost aside, if you are anything like me, you might cringe at the processing and memory cost of having these extra indexed values. In the next section we'll talk about ways to reduce the performance cost of having to do extra round trips (we briefly talked about it in the first chapter). 

If you actually think about it though, relational databases have the same overhead. Indexes take memory, must be scanned or ideally seeked and then the corresponding records must be looked up. The overhead is neatly abstracted away (and they  do a lot of optimizations in terms of the processing to make it very efficient). 

Again, having to manually deal with references in Redis in unfortunate. But any initial concerns you have about the performance or memory implications of this should be tested. I think you'll find it a non-issue.

### Round Trips and Pipelining 

We already mentioned that making frequent trips to the server is a common pattern in Redis. Since it is something you'll do often, it's worth taking a closer look at what features we can leverage to get the most out of it.

First, many commands either accept one or more set of parameters or have a sister-command which takes multiple parameters. We saw `mget` earlier, which takes multiple keys and returns the values: 

	keys = redis.lrange('newusers', 0, 10)
	redis.mget(*keys.map {|u| "users:#{u}"})

Or the `sadd` command which adds 1 or more members to a set:

	sadd friends:vladimir piter
	sadd friends:paul jessica leto "leto II" chani

Redis also supports pipelining. Normally when a client sends a request to Redis it waits for the reply before sending the next request. With pipelining you can send a number of requests without waiting for their responses. This reduces the networking overhead and can result in significant performance gains. 

It's worth noting that Redis will use memory to queue up the commands, so it's a good idea to batch them. How large a batch you use will depend on what commands you are using, and more specifically, how large the parameters are. But, if you are issuing commands against ~50 character keys, you can probably batch them in thousands or tens of thousands.

Exactly how you execute commands within a pipeline will vary from driver to driver. In Ruby you pass a block to the `pipelined` method:

	redis.pipelined do
	  9001.times do
		redis.incr('powerlevel')
	  end
	end

As you can probably guess, pipelining can really speed up a batch import!

### Transactions

Every Redis command is atomic, including the ones that do multiple things. Additionally, Redis has support for transactions when using multiple commands.

You might not know it, but Redis is actually single-threaded, which is how every command is guaranteed to be atomic. While one command is executing, no other command will run. (We'll briefly talk about scaling in a later chapter.) This is particularly useful when you consider that some commands do multiple things. For example:

`incr` is essentially a `get` followed by a `set`

`getset` sets a new value and returns the original

`setnx` first checks if the key exists, and only sets the value if it does not

Although these commands are useful, you'll inevitably need to run multiple commands as an atomic group. You do so by first issuing the `multi` command, followed by all the commands you want to execute as part of the transaction, and finally executing `exec` to actually execute the commands or `discard` to throw away, and not execute the commands. What guarantee does Redis make about transactions? 

* The commands will be executed in order

* The commands will be executed as a single atomic operation (without another client's command being executed halfway through)

* That either all or none of the commands in the transaction will be executed

You can, and should, test this in the command line interface. Also note that there's no reason why you can't combine pipelining and transactions.

	multi
	hincrby groups:1percent balance -9000000000
	hincrby groups:99percent balance 9000000000
	exec

Finally, Redis lets you specify a key (or keys) to watch and conditionally apply a transaction if the key(s) changed. This is used when you need to get values and execute code based on those values, all in a transaction. With the code above, we wouldn't be able to implement our own `incr` command since they are all executed together once `exec` is called. From code, we can't do:

	redis.multi()
	current = redis.get('powerlevel)
	redis.set('powerlevel', current + 1)
	redis.exec()

That isn't how Redis transactions work. But, if we add a `watch` to `powerlevel`, we can do:

	redis.watch('powerlevel')
	current = redis.get('powerlevel')
	redis.multi()
	redis.set('powerlevel', current + 1)
	redis.exec()

If another client changes the value of `powerlevel` after we've called `watch` on it, our transaction will fail. If no client changes the value, the set will work. We can execute this code in a loop until it works.

### Time Values

A slightly less common pattern that I'm fond of is using sorted sets to track time value. In Redis' documentation the sorting value is called a *score*, which might limit some people's imagination of different ways they can put it to use. 

Let's say we want to track the stock prices for a symbol. Our key would be the symbol, our *score* would be the timestamp and our value the price:

	redis.zadd('GOOG', Time.now.utc.to_i-100, 625.03)
	redis.zadd('GOOG', Time.now.utc.to_i-95, 623.01)
	redis.zadd('GOOG', Time.now.utc.to_i-95, 625.02)
	redis.zadd('GOOG', Time.now.utc.to_i-92, 624.98)

By using `zrangebyscore` we can get a range of values by a score. In our case that means getting values for a specific date range. If we wanted to get the last 5 seconds worth of prices, we could do:

	redis.zrangebyscore('GOOG', (Time.now.utc - 5).to_i, Time.now.utc.to_i)


### Keys Anti-Pattern

In the next chapter we'll talk about commands that aren't specifically related to data structures. Some of these are administrative or debugging tools. But there's one I'd like to talk about in particular: the `keys` command. This command takes a pattern and finds all the matching keys. This command seems like it's well suited for a number of tasks, but it should never be used in production code. Why? Because it does a linear scan through all the keys looking for matches. Or, put simply, it's slow.

How do people try and use it? Say you are building a hosted bug tracking service. Each account will have an `id` and you might decide to store each bug into a string value with a key that looks like `bug:account_id:bug_id`. If you ever need to find all of an account's bugs (to display them, or maybe delete them if they delete their account), you might be tempted (as I was!) to use the `keys` command:

	keys bug:1233:*

The better solution is to use a hash. Much like we can use hashes to provide a way to expose secondary indexes, so too can we use them to organize our data:

	hset bugs:1233 1 "{id:1, account: 1233, subject: '...'}"
	hset bugs:1233 2 "{id:2, account: 1233, subject: '...'}"

To get all the bug ids for an account we simply call `hkeys bugs:1233`. To delete a specific bug we can do `hdel bugs:1233 2` and to delete an account we can delete the key via `del bugs:1233`.


### In This Chapter	

This chapter, combined with the previous one, has hopefully given you some insight on how to use Redis to power real features. There are a number of other patterns you can use to build all types of things, but the real key is to understand the fundamental data structures and to get a sense for how they can be used to achieve things beyond your initial perspective.

\clearpage

## Chapter 4 - Beyond The Data Structures

While the five data structures form the foundation of Redis, there are other commands which aren't data structure specific. We've already seen a handful of these: `info`, `select`, `flushdb`, `multi`, `exec`, `discard`, `watch` and `keys`. This chapter will look at some of the other important ones.

### Expiration

Redis allows you to mark a key for expiration. You can give it an absolute time in the form of a Unix timestamp (seconds since January 1, 1970) or a time to live in seconds. This is a key-based command, so it doesn't matter what type of data structure the key represents. 

	expire pages:about 30
	expireat pages:about 1356933600

The first command will delete the key (and associated value) after 30 seconds. The second will do the same at 12:00 a.m. December 31st, 2012.

This makes Redis an ideal caching engine. You can can find out how long an item has to live until via the `ttl` command and you can remove the expiration on a key via the `persist` command:

	ttl pages:about
	persist pages:about

Finally, there's a special string command, `setex` which lets you set a string and specify a time to live in a single atomic command (this is more for convenience than anything else):

	setex pages:about 30 '<h1>about us</h1>....'

### Publication and Subscriptions

Redis lists have an `blpop` and `brpop` command which returns and removes the first (or last) element from the list or blocks until one is available. These can be used to power a simple queue.

Beyond this, Redis has first-class support for publishing messages and subscribing to channels. You can try this out yourself by opening a second `redis-cli` window. In the first window subscribe to a channel (we'll call it `warnings`):

	subscribe warnings

The reply is the information of your subscription. Now, in the other window, publish a message to the `warnings` channel:

	publish warnings "it's over 9000!"

If you go back to your first window you should have received the message to the `warnings` channel.

You can subscribe to multiple channels (`subscribe channel1 channel2 ...`), subscribe to a pattern of channels (`psubscribe warnings:*`) and use the `unsubscribe` and `punsubscribe` commands to stop listening to one or more channels, or a channel pattern.

Finally, notice that the `publish` command returned the value 1. This indicates the number of clients that received the message.


### Monitor and Slow Log

The `monitor` command lets you see what Redis is up to. It's a great debugging tool that gives you insight into how your application is interacting with Redis. In one of your two redis-cli windows (if one is still subscribed, you can either use the `unsubscribe` command or close the window down and re-open a new one) enter the `monitor` command. In the other, execute any other type of command (like `get` or `set`). You should see those commands, along with their parameters, in the first window.

You should be wary of running monitor in production, it really is a debugging and development tool. Aside from that, there isn't much more to say about it. It's just a really useful tool.

Along with `monitor`, Redis has a `slowlog` which acts as a great profiling tool. It logs any command which takes longer than a specified number of **micro**seconds. In the next section we'll briefly cover how to configure Redis, for now you can configure Redis to log all commands by entering:

	config set slowlog-log-slower-than 0

Next, issue a few commands. Finally you can retrieve all of the logs, or the most recent logs via:

	slowlog get
	slowlog get 10

You can also get the number of items in the slow log by entering `slowlog len`

For each command you entered you should see four parameters:

* An auto-incrementing id

* A Unix timestamp for when the command happened

* The time, in microseconds, it took to run the command

* The command and its parameters

The slow log is maintained in memory, so running it in production, even with a low threshold, shouldn't be a problem. By default it will track the last 1024 logs. 

### Sort

One of Redis' most powerful commands is `sort`. It lets you sort the values within a list, set or sorted set (sorted sets are ordered by score, not the members within the set). In its simplest form, it allows us to do:

	rpush users:leto:guesses 5 9 10 2 4 10 19 2
	sort users:leto:guesses

Which will return the values sorted from lowest to highest. Here's a more advanced example:

	sadd friends:ghanima leto paul chani jessica alia duncan
	sort friends:ghanima limit 0 3 desc alpha

The above command shows us how to page the sorted records (via `limit`), how to return the results in descending order (via `desc`) and how to sort lexicographically instead of numerically (via `alpha`).

The real power of `sort` is its ability to sort based on a referenced object. Earlier we showed how lists, sets and sorted sets are often used to reference other Redis objects. The `sort` command can dereference those relations and sort by the underlying value. For example, say we have a bug tracker which lets users watch issues. We might use a set to track the issues being watched:

	sadd watch:leto 12339 1382 338 9338

It might make perfect sense to sort these by id (which the default sort will do), but we'd also like to have these sorted by severity. To do so, we tell Redis what pattern to sort by. First, let's add some more data so we can actually see a meaningful result:

	set severity:12339 3
	set severity:1382 2
	set severity:338 5
	set severity:9338 4

To sort the bugs by severity, from highest to lowest, you'd do:

	sort watch:leto by severity:* desc

Redis will substitute the `*` in our pattern (identified via `by`) with the values in our list/set/sorted set. This will create the key name that Redis will query for the actual values to sort by.

Although you can have millions of keys within Redis, I think the above can get a little messy. Thankfully sort can also work on hashes and their fields. Instead of having a bunch of top-level keys you can leverage hashes:

	hset bug:12339 severity 3
	hset bug:12339 priority 1
	hset bug:12339 details "{id: 12339, ....}"

	hset bug:1382 severity 2
	hset bug:1382 priority 2
	hset bug:1382 details "{id: 1382, ....}"

	hset bug:338 severity 5
	hset bug:338 priority 3
	hset bug:338 details "{id: 338, ....}"

	hset bug:9338 severity 4
	hset bug:9338 priority 2
	hset bug:9338 details "{id: 9338, ....}"

Not only is everything better organized, and we can sort by `severity` or `priority`, but we can also tell `sort` what field to retrieve:

	sort watch:leto by bug:*->priority get bug:*->details

The same value substitution occurs, but Redis also recognizes the `->` sequence and uses it to look into the specified field of our hash. We've also included the `get` parameter, which also does the substitution and field lookup, to retrieve bug details.

Over large sets, `sort` can be slow. The good news is that the output of a `sort` can be stored:

	sort watch:leto by bug:*->priority get bug:*->details store watch_by_priority:leto

Combining the `store` capabilities of `sort` with the expiration commands we've already seen makes for a nice combo.


### In This Chapter 

This chapter focused on non-data structure-specific commands. Like everything else, their use is situational. It isn't uncommon to build an app or feature that won't make use of expiration, publication/subscription and/or sorting. But it's good to know that they are there. Also, we only touched on some of the commands. There are more, and once you've digested the material in this book it's worth going through the [full list](http://redis.io/commands).

\clearpage

## Chapter 5 - Administration

Our last chapter is dedicated to some of the administrative aspects of running Redis. In no way is this a comprehensive guide on Redis administration. At best we'll answer some of the more basic questions new users to Redis are most likely to have.

### Configuration

When you first launched the Redis server, it warned you that the `redis.conf` file could not be found. This file can be used to configure various aspects of Redis. A well documented `redis.conf` file is available for each release of Redis. The sample file contains the default configuration options, so it's useful to both understand what the settings do and what their defaults are. You can find it at <https://github.com/antirez/redis/raw/2.4.6/redis.conf>.

**This is the config file for Redis 2.4.6. You should replace "2.4.6" in the above URL with your version. You can find your version by running the `info` command and looking at the first value.**

Since the file is well-documented, we won't be going over the settings.

In addition to configuring Redis via the `redis.conf` file, the `config set` command can be used to set individual values. In fact, we already used it when setting the `slowlog-log-slower-than` setting to 0. 

There's also the `config get` command which displays the value of a setting. This command supports pattern matching. So if we want to display everything related to logging, we can do:

	config get *log*

### Authentication

Redis can be configured to require a password. This is done via the `requirepass` setting (set through either the `redis.conf` file or the `config set` command). When `requirepass` is set to a value (which is the password to use), clients will need to issue an `auth password` command.

Once a client is authenticated, they can issue any command against any database. This includes the `flushall` command which erases every key from every database. Through the configuration, you can rename commands to achieve some security through obfuscation:

	rename-command CONFIG 5ec4db169f9d4dddacbfb0c26ea7e5ef	
	rename-command FLUSHALL 1041285018a942a4922cbf76623b741e

Or you can disable a command by setting the new name to an empty string.

### Size Limitations

As you start using Redis, you might wonder "how many keys can I have?". You might also wonder how many fields can a hash have (especially when you use it to organize your data), or how many elements can lists and sets have? Per instance, the practical limits for all of these is in the hundreds of millions.


### Replication

Redis supports replication, which means that as you write to one Redis instance (the master), one or more other instances (the slaves) are kept up-to-date by the master. To configure a slave you use either the `slaveof` configuration setting or the `slaveof` command (instances running without this configuration are or can be masters).

Replication helps protect your data by copying to different servers. Replication can also be used to improve performance since reads can be sent to slaves. They might respond with slightly out of date data, but for most apps that's a worthwhile tradeoff.

Unfortunately, Redis replication doesn't yet provide automated failover. If the master dies, a slave needs to be manually promoted. Traditional high-availability tools that use heartbeat monitoring and scripts to automate the switch are currently a necessary headache if you want to achieve some sort of high availability with Redis.

### Backups

Backing up Redis is simply a matter of copying Redis' snapshot to whatever location you want (S3, FTP, ...). By default Redis saves its snapshop to a file named `dump.rdb`. At any point in time, you can simply `scp`, `ftp` or `cp` (or anything else) this file.

It isn't uncommon to disable both snapshotting and the append-only file (aof) on the master and let a slave take care of this. This helps reduce the load on the master and lets you set more aggressive saving parameters on the slave without hurting overall system responsiveness.

### Scaling and Redis Cluster

### Масштабирование и Redis Cluster

Replication is the first tool a growing site can leverage. Some commands are more expensive than others (`sort` for example) and offloading their execution to a slave can keep the overall system responsive to incoming queries.

Репликация является первым инструментом, который используют при росте нагрузке на систему. Некоторые команды являются более дорогостоящими, чем другие (например `sort`). Такие запросы предпочтительно выполнять не на master-узлах, а на slave. В этом случае удастся сохранять общую отзывчивость системы.

Beyond this, truly scaling Redis comes down to distributing your keys across multiple Redis instances (which could be running on the same box, remember, Redis is single-threaded). For the time being, this is something you'll need to take care of (although a number of Redis drivers do provide consistent-hashing algorithms). Thinking about your data in terms of horizontal distribution isn't something we can cover in this book. It's also something you probably won't have to worry about for a while, but it's something you'll need to be aware of regardless of what solution you use.

Помимо этого, по-настоящему масштабируемый Redis использует распределение ключей между несколькими узлами Redis (которые могут быть запущены в той же системе - как мы помним, Redis однопоточный). В настоящее время, это то, над чем приходится заботится нам (хотя ряд драйверов предоставляют ???? consistent-hashing algorithms ????). Мы не может покрыть в этой книге вопросы, касающиеся работы с данными в ракурсе горизонтального масштабирования. Тем более, вам скорее всего не придется беспокоится о подобных вещах, однако стоит быть вкурсе независимо от того, какое решение вы используете.

The good news is that work is under way on Redis Cluster. Not only will this offer horizontal scaling, including rebalancing, but it'll also provide automated failover for high availability.

Хорошей новостью является то, что ведется работа над Redis Cluster. Это решение предоставит не только возможность горизонтального масштабирования, включая автоматическую балансировку нагрузки, но также решения в плане отказоустойчивости.

High availability and scaling is something that can be achieved today, as long as you are willing to put the time and effort into it. Moving forward, Redis Cluster should make things much easier.

Высокая отказоустойчивость и масштабируемость может быть достигнута уже сегодня, но вам придется вложить некоторое количество времени и усилий в это. В конечном счете, Redis Cluster должен сделать эти вещи гораздо более простыми.

### In This Chapter

### Обобщение

Given the number of projects and sites using Redis already, there can be no doubt that Redis is production-ready, and has been for a while. However, some of the tooling, especially around security and availability is still young. Redis Cluster, which we'll hopefully see soon, should help address some of the current management challenges.

Учитывая количество проектов, уже использующих Redis, не может быть никаких сомнений, что эта система является ???? production-ready ????. Однако, некоторые из инструментов, особенно касающиеся безопасности и отказоустойчивости все еще требуют доработки. Redis Cluster, который мы надеемся увидеть в ближайшее время, должен помочь решить некоторые из существующих проблем администрирования.

\clearpage

## Conclusion

## Заключение

In a lot of ways, Redis represents a simplification in the way we deal with data. It peels away much of the complexity and abstraction available in other systems. In many cases this makes Redis the wrong choice. In others it can feel like Redis was custom-built for your data.

Redis использует множество способов облегчить наше взаимодействие с данными. Эта система убирает большую часть сложности и абстракции, присущих другим системам. Во многих случаях это делает Redis неудачным выбором. Но в других случаях может показаться, что Redis был написан для решения вашей конкретной проблемы.

Ultimately it comes back to something I said at the very start: Redis is easy to learn. There are many new technologies and it can be hard to figure out what's worth investing time into learning. When you consider the real benefits Redis has to offer with its simplicity, I sincerely believe that it's one of the best investments, in terms of learning, that you and your team can make.

В конечном итоге мы вернулись к тому, что я говорил в самом начале: Redis прост в освоении. Когда вокруг так много новых технологий, не просто понять, что стоит времени на обучение. Когда вы ощутите реальное преимущество, которое Redis может предложить, я уверен, что у вас не останется вопросов о том, уделять ли время на изучение Redis.
