\thispagestyle{empty}
\changepage{}{}{}{-0.5cm}{}{2cm}{}{}{}
![The Little Redis Book, By Karl Seguin](title.png)\

\clearpage
\changepage{}{}{}{0.5cm}{}{-2cm}{}{}{}

## Sobre Este Livro

### Licença

The Little Redis Book é licenciado sob a licença Attribution-NonCommercial 3.0 Unported. Você não deve pagar por este livro.

Você é livre para copiar, distribuir, modificar ou exibir o livro. Entretanto, peço que você sempre atribua este livro a mim, Karl Seguin, e não use-o para propósitos comerciais.

Você pode ver o *texto completo* da **licença em**:

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>

### Sobre o Autor

Karl Seguin é um desenvolvedor com experiência em vários campos e tecnologias. Ele é um contribuidor ativo em projetos de Software Livre, um escritor técnico e palestrante ocasional. Ele escreveu diversos artigos sobre e algumas ferramentas para Redis. Redis é usado no ranking e estatística de seu serviço gratuito para desenvolvedores casuais de jogos: [mogade.com](http://mogade.com/).

Karl escreveu [The Little MongoDB Book](http://openmymind.net/2011/3/28/The-Little-MongoDB-Book/), um livro gratuito e popular sobre MongoDB.

Seu blog pode ser acessado em <http://openmymind.net> e seus tweets via [@karlseguin](http://twitter.com/karlseguin)

### Agradecimentos

Um obrigado especial para [Perry Neal](https://twitter.com/perryneal) por me emprestar seus olhos, mente e paixão. Você me forneceu uma ajuda sem preço.

### Versão Mais Atual

O código mais recente deste livro está disponível em:
<http://github.com/karlseguin/the-little-redis-book>

\clearpage

## Introdução

Nos últimos anos, as técnicas e ferramentas usadas para persistência e busca de dados têm crescido num ritmo impressionante. Ao mesmo tempo em que é seguro dizer que os bancos de dados relacionais não vão a lugar algum, nós também podemos dizer que o ecossistema de dados nunca será o mesmo.

De todas as ferramentas e soluções novas, para mim, Redis tem sido a mais excitante. Porquê? Primeiramente porque é inacreditavelmente fácil de aprender. Pode-se falar em termos de horas para descrever a quantidade de tempo necessária para sentir-se confortável com Redis. Em segundo lugar, ele resolve um conjunto específico de problemas e ao mesmo consegue ser bastante genérico. O que isso significa exatamente? Redis não tenta ser a solução de todos os problemas para qualquer tipo de dado. Ao conhecer melhor o Redis, se torna mais e mais evidente que tipo de problemas ele pode ou não resolver. E quando ele pode, como um desenvolvedor, é uma grande experiência.

Embora você possa construir um sistema inteiro usando somente Redis, eu acho que a maioria das pessoas irá descobrir que ele é uma adição à sua solução genérica de dados - seja ela um banco de dados relacional, um sistema orientado a documentos ou qualquer outra coisa. É o tipo de solução que você usa para implementar funcionalidades específicas. Dessa forma, ele se assemelha a um mecanismo de indexação. Você não iria construir sua aplicação inteira em Lucene. Mas quando você precisa de uma boa busca, é uma experiência muito melhor - tanto para você como para seus usuários. É claro que as semelhanças entre o Redis e os mecanismos de indexação acabam aqui.

O objetivo deste livro é construir a fundação que você precisa para dominar o Redis. Nós iremos focar em aprender os cinco tipos de estruturas de dados do Redis e olhar diversas abordagens para a modelagem de dados. Nós também iremos tocar em alguns detalhes chave da parte administrativa e técnicas de depuração.

## Começando

Todos aprendemos de maneiras diferentes: alguns gostam de botar a mão na massa, outros de assistir vídeos e alguns de ler. Nada irá ajudar você a entender Redis mais do que utilizá-lo. Redis é fácil de instalar e vem com um console simples que nos dará tudo que precisamos. Vamos gastar alguns minutos para deixá-lo pronto para rodar em nossa máquina.

### No Windows

O Redis em si não suporta o Windows oficialmente, mas existem algumas opções disponívels. Você não deveria rodá-las em produção, mas eu nunca experimentei limitações no ambiente de desenvolvimento.

Um port feito pela Microsoft Open Technologies, Inc. pode ser encontrado em <https://github.com/MSOpenTech/redis>. Quando este livro foi escrito, a solução não estava pronta para ser usada em ambiente de produção.

Outra solução, que já está disponível há algum tempo, pode ser encontrada em <https://github.com/dmajkic/redis/downloads>. Você pode baixar a versão mais recente (que deve estar no topo da lista). Extraia o arquivo zip e, dependendo da sua arquitetura, abra a pasta `64bit` ou `32bit`.

### No *nix e MacOSX

Para usuários de *nix e Mac, a melhor opção é compilar a partir do código-fonte. As instruções, juntamente com o número da versão mais recente, estão disponíveis em <http://redis.io/download>. No tempo em que este livro foi escrito, a versão mais recente era a 2.4.6. Para instalar esta versão nós executaríamos:

	wget http://redis.googlecode.com/files/redis-2.4.6.tar.gz
	tar xzf redis-2.4.6.tar.gz
	cd redis-2.4.6
	make

Alternativamente, o Redis está disponível via diversos gerenciadores de pacote. Por exemplo, usuários de MacOSX com Homebrew instalado pode simplesmente digitar `brew install redis`.)

Se você compilar do fonte, as saídas binárias serão colocadas na pasta `src`. Acesse a pasta `src` executando o comando `cd src`.

### Rodando e Conectando ao Redis

Se tudo funcionou, os binários do Redis devem estar disponíveis nas pontas de seus dedos. Redis tem um monte de executáveis. Nós iremos focar no servidor do Redis e na interface em linha de comando (como um cliente DOS). Vamos iniciar o servidor. No Windows, dê um clique duplo em `redis-server`. No *unix/MacOSX rode o comando `./redis-server`.

Na mensagem de inicialização você verá um aviso informando que o arquivo `redis.conf` não foi encontrado. Redis irá então usar as configurações padrão – isso basta para o que nós vamos fazer.

Em seguida inicie o console do Redis dando um clique duplo em `redis-cli` (no Windows) ou executando o comando `./redis-cli` (no *nix/MacOSX). Isso irá lhe conectar ao servidor local rodando na porta padrão (6379).

Você pode testar se tudo está funcionando digitando `info` na interface de linha de comando. Então você verá uma porção de pares chave-valor que fornecem uma boa idéia sobre o estado do servidor.

Se você está tendo problemas com o procedimento anterior, eu sugiro que você busque ajuda no [grupo oficial de suporte do Redis](https://groups.google.com/forum/#!forum/redis-db).

## Drivers do Redis

Como você vai aprender em breve, a API do Redis é melhor definida como um conjuto explícito de funções. Ela tem um aspecto simples e processual. Isto significa que tanto usando a ferramenta da linha de comando como um driver para sua linguagem favorita, vai ser tudo muito parecido. Portanto, você não deve ter problemas problemas em acompanhar os passos seguintes se você preferir usar uma linguagem de programação. Se preferir, siga para a [página de clientes] e baixe o driver apropriado.

\clearpage

## Chapter 1 - The Basics

O que faz o Redis especial? Que tipos de problemas ele resolve? What should developers watch out for when using it? Before we can answer any of these questions, we need to understand what Redis is.

Redis is often described as an in-memory persistent key-value store. I don't think that's an accurate description. Redis does hold all the data in memory (more on this in a bit), and it does write that out to disk for persistence, but it's much more than a simple key-value store. It's important to step beyond this misconception otherwise your perspective of Redis and the problems it solves will be too narrow.

The reality is that Redis exposes five different data structures, only one of which is a typical key-value structure. Understanding these five data structures, how they work, what methods they expose and what you can model with them is the key to understanding Redis. First though, let's wrap our heads around what it means to expose data structures.

If we were to apply this data structure concept to the relational world, we could say that databases expose a single data structure - tables. Tables are both complex and flexible. There isn't much you can't model, store or manipulate with tables. However, their generic nature isn't without drawbacks. Specifically, not everything is as simple, or as fast, as it ought to be. What if, rather than having a one-size-fits-all structure, we used more specialized structures? There might be some things we can't do (or at least, can't do very well), but surely we'd gain in simplicity and speed?

Using specific data structures for specific problems? Isn't that how we code? You don't use a hashtable for every piece of data, nor do you use a scalar variable. To me, that defines Redis' approach. If you are dealing with scalars, lists, hashes, or sets, why not store them as scalars, lists, hashes and sets? Why should checking for the existence of a value be any more complex than calling `exists(key)` or slower than O(1) (constant time lookup which won't slow down regardless of how many items there are)?

## The Building Blocks

### Databases

Redis has the same basic concept of a database that you are already familiar with. A database contains a set of data. The typical use-case for a database is to group all of an application's data together and to keep it separate from another application's.

In Redis, databases are simply identified by a number with the default database being number `0`. If you want to change to a different database you can do so via the `select` command. In the command line interface, type `select 1`. Redis should reply with an `OK` message and your prompt should change to something like `redis 127.0.0.1:6379[1]>`. If you want to switch back to the default database, just enter `select 0` in the command line interface..

### Commands, Keys and Values

While Redis is more than just a key-value store, at its core, every one of Redis' five data structures has at least a key and a value. It's imperative that we understand keys and values before moving on to other available pieces of information.

Keys are how you identify pieces of data. We'll be dealing with keys a lot, but for now, it's good enough to know that a key might look like `users:leto`. One could reasonably expect such a key to contain information about a user named `leto`. The colon doesn't have any special meaning, as far as Redis is concerned, but using a separator is a common approach people use to organize their keys.

Values represent the actual data associated with the key. They can be anything. Sometimes you'll store strings, sometimes integers, sometimes you'll store serialized objects (in JSON, XML or some other format). For the most part, Redis treats values as a byte array and doesn't care what they are. Note that different drivers handle serialization differently (some leave it up to you) so in this book we'll only talk about string, integer and JSON.

Let's get our hands a little dirty. Enter the following command:

	set users:leto "{name: leto, planet: dune, likes: [spice]}"

This is the basic anatomy of a Redis command. First we have the actual command, in this case `set`. Next we have its parameters. The `set` command takes two parameters: the key we are setting and the value we are setting it to. Many, but not all, commands take a key (and when they do, it's often the first parameter). Can you guess how to retrieve this value? Hopefully you said (but don't worry if you weren't sure!):

	get users:leto

Go ahead and play with some other combinations. Keys and values are fundamental concepts, and the `get` and `set` commands are the simplest way to play with them. Create more users, try different types of keys, try different values.

### Querying

As we move forward, two things will become clear. As far as Redis is concerned, keys are everything and values are nothing. Or, put another way, Redis doesn't allow you to query an object's values. Given the above, we can't find the user(s) which live on planet `dune`.

For many, this is will cause some concern. We've lived in a world where data querying is so flexible and powerful that Redis' approach seems primitive and unpragmatic. Don't let it unsettle you too much. Remember, Redis isn't a one-size-fits-all solution. There'll be things that just don't belong in there (because of the querying limitations). Also, consider that in some cases you'll find new ways to model your data.

We'll look at more concrete examples as we move on, but it's important that we understand this basic reality of Redis. It helps us understand why values can be anything - Redis never needs to read or understand them. Also, it helps us get our minds thinking about modeling in this new world.

### Memory and Persistence

We mentioned before that Redis is an in-memory persistent store. With respect to persistence, by default, Redis snapshots the database to disk based on how many keys have changed. You configure it so that if X number of keys change, then save the database every Y seconds. By default, Redis will save the database every 60 seconds if 1000 or more keys have changed all the way to 15 minutes if 9 or less keys has changed.

Alternatively (or in addition to snapshotting), Redis can run in append mode. Any time a key changes, an append-only file is updated on disk. In some cases it's acceptable to lose 60 seconds worth of data, in exchange for performance, should there be some hardware or software failure. In some cases such a loss is not acceptable. Redis gives you the option. In chapter 5 we'll see a third option, which is offloading persistence to a slave.

With respect to memory, Redis keeps all your data in memory. The obvious implication of this is the cost of running Redis: RAM is still the most expensive part of server hardware.

I do feel that some developers have lost touch with how little space data can take. The Complete Works of William Shakespeare takes roughly 5.5MB of storage. As for scaling, other solutions tend to be IO- or CPU-bound. Which limitation (RAM or IO) will require you to scale out to more machines really depends on the type of data and how you are storing and querying it. Unless you're storing large multimedia files in Redis, the in-memory aspect is probably a non-issue. For apps where it is an issue you'll likely be trading being IO-bound for being memory bound.

Redis did add support for virtual memory. However, this feature has been seen as a failure (by Redis' own developers) and its use has been deprecated.

(On a side note, that 5.5MB file of Shakespeare's complete works can be compressed down to roughly 2MB. Redis doesn't do auto-compression but, since it treats values as bytes, there's no reason you can't trade processing time for RAM by compressing/decompressing the data yourself.)

### Putting It Together

We've touched on a number of high level topics. The last thing I want to do before diving into Redis is bring some of those topics together. Specifically, query limitations, data structures and Redis' way to store data in memory.

When you add those three things together you end up with something wonderful: speed. Some people think "Of course Redis is fast, everything's in memory." But that's only part of it. The real reason Redis shines versus other solutions is its specialized data structures.

How fast? It depends on a lot of things - which commands you are using, the type of data, and so on. But Redis' performance tends to be measured in tens of thousands, or hundreds of thousands of operations **per second**. You can run `redis-benchmark` (which is in the same folder as the `redis-server` and `redis-cli`) to test it out yourself.

I once changed code which used a traditional model to using Redis. A load test I wrote took over 5 minutes to finish using the relational model. It took about 150ms to complete in Redis. You won't always get that sort of massive gain, but it hopefully gives you an idea of what we are talking about.

It's important to understand this aspect of Redis because it impacts how you interact with it. Developers with an SQL background often work at minimizing the number of round trips they make to the database. That's good advice for any system, including Redis. However, given that we are dealing with simpler data structures, we'll sometimes need to hit the Redis server multiple times to achieve our goal. Such data access patterns can feel unnatural at first, but in reality it tends to be an insignificant cost compared to the raw performance we gain.

### In This Chapter

Although we barely got to play with Redis, we did cover a wide range of topics. Don't worry if something isn't crystal clear - like querying. In the next chapter we'll go hands-on and any questions you have will hopefully answer themselves.

The important takeaways from this chapter are:

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

	lpush newusers goku
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

Regardless of how many friends a user has, we can efficiently tell (O(1)) whether userX is a friend of userY or not:

	sismember friends:leto jessica
	sismember friends:leto vladimir

Furthermore we can see what two or more people share the same friends:

	sinter friends:leto friends:duncan

and even store the result at a new key:

	sinterstore friends:leto_duncan friends:leto friends:duncan

Sets are great for tagging or tracking any other properties of a value for which duplicates don't make any sense (or where we want to apply set operations such as intersections and unions).

### Sorted Sets

The last and most powerful data structure are sorted sets. If hashes are like strings but with fields, then sorted sets are like sets but with a score. The score provides sorting and ranking capabilities. If we wanted a ranked list of friends, we might do:

	zadd friends:duncan 70 ghanima 95 paul 95 chani 75 jessica 1 vladimir

Want to find out how many friends `duncan` has with a score of 90 or over?

	zcount friends:duncan 90 100

How about figuring out `chani`'s rank?

	zrevrank friends:duncan chani

We use `zrevrank` instead of `zrank` since Redis' default sort is from low to high (but in this case we are ranking from high to low). The most obvious use-case for sorted sets is a leaderboard system. In reality though, anything you want sorted by an some integer, and be able to efficiently manipulate based on that score, might be a good fit for a sorted set.

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

Next we have linear commands, or O(N). Looking for a non-indexed column in a table is an O(N) operation. So is using the `ltrim` command. However, in the case of `ltrim`, N isn't the number of elements in the list, but rather the elements being removed. Using `ltrim` to remove 1 item from a list of millions will be faster than using `ltrim` to remove 10 items from a list of thousands. (Though they'll probably both be so fast that you wouldn't be able to time it.)

`zremrangebyscore` which removes elements from a sorted set with a score between a minimum and a maximum value has a complexity of O(log(N)+M). This makes it a mix. By reading the documentation we see that N is the number of total elements in the set and M is the number of elements to be removed. In other words, the number of elements that'll get removed is probably going to be more significant, in terms of performance, than the total number of elements in the set.

The `sort` command, which we'll discuss in greater detail in the next chapter has a complexity of O(N+M*log(M)). From its performance characteristic, you can probably tell that this is one of Redis' most complex commands.

There are a number of other complexities, the two remaining common ones are O(N^2) and O(C^N). The larger N is, the worse these perform relative to a smaller N. None of Redis' commands have this type of complexity.

It's worth pointing out that the Big O notation deals with the worst case. When we say that something takes O(N), we might actually find it right away or it might be the last possible element.


### Pseudo Multi Key Queries

A common situation you'll run into is wanting to query the same value by different keys. For example, you might want to get a user by email (for when they first log in) and also by id (after they've logged in). One horrible solution is to duplicate your user object into two string values:

	set users:leto@dune.gov "{id: 9001, email: 'leto@dune.gov', ...}"
	set users:9001 "{id: 9001, email: 'leto@dune.gov', ...}"

This is bad because it's a nightmare to manage and it takes twice the amount of memory.

It would be nice if Redis let you link one key to another, but it doesn't (and it probably never will). A major driver in Redis' development is to keep the code and API clean and simple. The internal implementation of linking keys (there's a lot we can do with keys that we haven't talked about yet) isn't worth it when you consider that Redis already provides a solution: hashes.

Using a hash, we can remove the need for duplication:

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

Again, having to manually deal with references in Redis is unfortunate. But any initial concerns you have about the performance or memory implications of this should be tested. I think you'll find it a non-issue.

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
	current = redis.get('powerlevel')
	redis.set('powerlevel', current + 1)
	redis.exec()

That isn't how Redis transactions work. But, if we add a `watch` to `powerlevel`, we can do:

	redis.watch('powerlevel')
	current = redis.get('powerlevel')
	redis.multi()
	redis.set('powerlevel', current + 1)
	redis.exec()

If another client changes the value of `powerlevel` after we've called `watch` on it, our transaction will fail. If no client changes the value, the set will work. We can execute this code in a loop until it works.

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

This makes Redis an ideal caching engine. You can find out how long an item has to live until via the `ttl` command and you can remove the expiration on a key via the `persist` command:

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

Although you can have millions of keys within Redis, I think the above can get a little messy. Thankfully `sort` can also work on hashes and their fields. Instead of having a bunch of top-level keys you can leverage hashes:

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

## Capítulo 5 - Administração

Nosso último capítulo é dedicado a alguns aspectos administrativos do Redis. De forma alguma isso é um guia abrangente de administração do Redis. No melhor dos casos, vamos responder algumas das dúvidas mais básicas que novos usuários do Redis provavelmente vão ter.

### Configuração

Quando você rodou o servidor do Redis pela primeira vez, ele lhe avisou que o arquivo `redis.conf` não pôde ser encontrado. Este arquivo pode ser usado para configurar vários aspetos do Redis. Um arquivo `redis.conf` bem documentado está disponível em cada versão do Redis. O arquivo de exemplo contém as opções padrão de configuração, sendo útil tanto para entender o que os ajustes fazem quanto para saber quais são os valores padrão de cada ajuste. Você pode encontrá-lo em <https://github.com/antirez/redis/raw/2.4.6/redis.conf>.

**Este é o arquivo de configuração do Redis 2.4.6. Você deve substituir o "2.4.6" na URL acima com a sua versão. Você pode descobrir sua versão executando o comando `info` e olhando o primeiro valor.**

Como o arquivo é bem documentado, nós não vamos analisar cada ajuste.

Além de configurar o Redis pelo arquivo `redis.conf`, podemos usar o comando `config set` para mudar valores individuais. Na verdade, nós já usamos o `config set` quando passamos a configuração `slowlog-log-slower-than` para 0.

Também existe o comando `config get`, que exibe o valor de uma configuração. Este comando suporta a busca por padrões. Assim, se queremos exibis tudo relacionado a logs, podemos fazer:

   config get *log*

### Authentication

O Redis pode ser configurado para exigir uma senha. Isto pode ser feito através da configuração `requirepass` (que pode ser informado tanto no `redis.conf` quando com o comando `config set`). Quando o `requirepass` possui um valor configurado (que é a senha que deve ser utilizada), os clientes vão precisar executar um comando `auth password`.

Uma vez que o cliente está autenticado, ele pode executar qualquer comando em qualquer banco de dados. Isto inclui o `flushall`, que apaga todas as chaves de todos os bancos. Na configuração você pode renomear os comandos para obter um pouco de segurança por obfuscação:

	rename-command CONFIG 5ec4db169f9d4dddacbfb0c26ea7e5ef
	rename-command FLUSHALL 1041285018a942a4922cbf76623b741e

Ou você pode desabilitar um comando setando o novo nome para uma string vazia.

### Limitações de Tamanho

Enquanto você começa a usar o Redis, você pode se perguntar "quantas chaves eu posso guardar?" Você pode também pensar quantos campos um hash pode ter (especialmente quando você usa-os para organizar seus dados), ou quantos elementos as listas e sets podem ter. Por instância, o limite prático para todos estes é na casa das centenas de milhões.

### Replicação

O Redis suporta replicação, o que significa que enquanto você escreve para uma instância do Redis (o master, ou "mestre"), uma ou mais outras instâncias (os slaves, ou "escravos") são mantidas atualizadas pelo master. Para configurar um slave, você pode usar tanto a configuração `slaveof` quanto o comando `slaveof` (instâncias que rodam sem esta configuração são ou podem ser masters).

A replicação ajuda a proteger seus dados copiando-os para servidores diferentes. A replicação também pode ser usada para melhorar o desempenho, já que as leituras podem ser feitas a partir dos slaves. Eles podem responder com dados levemente desatualizados, mas para a maioria das aplicações esta é uma troca que vale a pena.

Infelizmente, a replicação do Redis não provê ainda failover automático. Se o master morrer, um slave precisa ser promovido manualmente. Ferramentas tradicionais de alta disponibilidade que usem monitoração de heartbeats e scripts para automatizar a troca são atualmente dores-de-cabeça necessárias se você quer atingir algum grau de alta disponibilidade com o Redis.

### Backups

Fazer backup do Redis é uma simples questão de copiar o snapshot do Redis para onde quer que você queira (S3, FTP, ...). Por padrão, o Redis salva o snapshot em um arquivo chamado `dump.rdb`. Você pode simplesmente copiar este arquivo a qualquer momento com `scp`, `ftp` ou `cp` (ou qualquer outra coisa).

Não é incomum desabilitar tanto a gravação de snapshots quando o arquivo append-only (aof) no master e deixar um slave tomar conta disso. Isso ajuda a reduzir a carga no maste e lhe permite setar parâmetros de salvamento mais agressivos no slave sem ferir a responsividade geral do sistema.

### Escalabilidade e Cluster Redis

Replicação é a primeira ferramenta da qual um site em crescimento pode tirar vantagem. Alguns comandos são mais custosos que outros (`sort` por exemplo) e despachar a execução deles para um slave pode manter o sistema em geral responsivo às queries que chegam.

Além disso, escalar de verdade o Redis acaba chegando em distribuir suas chaves em várias instâncias do Redis (que podem rodar na mesma máquina - lembre-se: o Redis só possui uma thread). Até agora, isso é algo que é você quem vai precisar resolver (apesar que vários drivers de Redis provêem algoritmos de hash consistentes). Pensar seus dados em termos de distribuição horizontal é algo que não vamos cobrir neste livro. Também é algo que você provavelmente não vai ter que se preocupar por um bom tempo, mas é algo do qual você precisa estar ciente independente da solução que você use.

A boa notícia é que já existem esforços no sentido de um Cluster Redis. Isso não apenas vai oferecer escalabilidade horizontal – incluindo rebalanceamento – como também vai fornecer failover automático para alta disponibilidade.

Alta disponibilidade e escalabilidade são algo que pode ser atingido hoje, contanto que você esteja disposto a direcionar seu tempo e esforço para isto. Mais adiante, o Cluster Redis deve deixar as coisas bem mais fáceis.

### Neste Capítulo

Dado o número de projetos e sites que já usam Redis, não pode haver nenhuma dúvida que o Redis está pronto para produção, e que já tem estado assim por um bom tempo. Entretanto, algumas das ferramentas - especialmente as de segurança e disponibilidade - ainda são jovens. O Cluster Redis, que esperamos ver em breve, deve ajudar a resolver alguns dos desafios de administração atuais.

\clearpage

## Conclusão

De várias maneiras, o Redis representa uma simplificação na forma como lidamos com dados. Ele remove muito da complexidade e abstração disponíveis em outros sistemas. Em vários casos, isso faz do Redis a escolha errada. Em outros casos, é como se o Redis tivesse sido feito especificamente para os seus dados.

No fim das contas voltamos para algo que eu disse bem no início: o Redis é fácil de aprender. Há muitas tecnologias novas, e pode ser difícil descobrir em quais vale a pena investir o tempo de aprender. Quando você considera os reais benefícios que o Redis tem a oferecer com sua simplicidade, eu acredito sinceramente que ele é um dos melhores investimentos – em termos de aprendizado – que você e sua equipe podem fazer.