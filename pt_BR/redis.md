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

## Capítulo 1 - O Básico

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

## Capítulo 3 - Explorando as Estruturas de Dados

No capítulo anterior, falamos sobre as cinco estruturas de dados e demos alguns exemplos de quais tipos de problemas elas podem resolver. Agora é hora de ver alguns tópicos e padrões de projeto mais avançados, porém comuns.

### Notação do Grande-O

Através do livro nós fizemos referência à notação do Grande-O na forma O(n) ou O(1). A notação do Grande-O é usada para explicar como algo se comporta dado um certo número de elementos. No Redis, ela é usada para nos dizer o quão rápido é um comando, baseado no número de itens com os quais estamos lidando.

A documentação do Redis nos fala a notação do Grande-O para cada um de seus comandos. Ela também nos diz quais os fatores que influenciam o desempenho. Vamos olhar alguns exemplos.

O mais rápido possível é O(1), que é uma constante. Tanto faz se estamos lidando com 5 ou 5 milhões de itens, vamos ter o mesmo desempenho. O comando `sismember`, que nos diz se um valor pertence a um conjunto, é O(1). `sismember` é um comando poderoso, e as características de desempenho dele são um motivo forte para isso. Vários comandos do Redis são O(1).

A complexidade logarítmica, ou O(log(N)), é a segunda possibilidade mais rápida, porque ela varre seções cada vez menores. Usando esta abordagem de "dividir e conquistar", um grande número de itens é rapidamente quebrado em poucas iterações. `zadd` é um comando O(log(N)), onde N é o número de elementos que já pertencem ao conjunto.

Depois temos comandos lineares, ou O(N). Procurar por uma coluna não-indexada em uma tabela é uma operação O(N), assim como usar o comando `ltrim`. Entretanto, no caso do `ltrim`, N não é o número de elementos na lista, mas sim os elementos a serem removidos. Usar `ltrim` para remover 1 item de uma lista de milhões vai ser mais rápido que usar `ltrim` para remover 10 itens de uma lista de milhares (apesar que eles provavelmente vão ser tão rápidos que você nem conseguiria medir).

`zremrangebyscore`, que remove os elementos de um conjunto ordenado com um score entre certos valores mínimo e máximo, tem uma complexidade de O(log(N)+M). Isto faz dele um misto. Lendo a documentação, nós vemos que N é o número de elementos totais no conjunto e M é o número de elementos a serem removidos. Em outras palavras, o número de elementos que serão removidos provavelmente vai ser mais significativo, em termos de desempenho, que o número total de elementos no conjunto.

O comando `sort`, que nós vamos discutir mais detalhadamente no próximo capítulo, tem uma complexidade de O(N+M*log(M)). Das características de desempenho dele, você provavelmente já pode inferir que este é um dos comandos mais complexos do Redis.

Há várias outras complexidades. As duas mais comuns que restam são O(N^2) e O(C^N). Quanto maior for N, pior o desempenho deles relativo a um N pequeno. Nenhum dos comandos do Redis tem esse tipo de complexidade.

Vale lembrar que a notação do Grande-O trata do pior caso. Quando dizemos que algo leva O(N), na verdade podemos encontrar o item logo de cara ou ele pode ser o último elemento possível.


### Consultas Pseudo-Multi-Chave

Uma situação comum com a qual você vai se deparar é querer consultar o mesmo valor em chaves diferentes. Por exemplo, você pode querer obter um usuário pelo email (quando ele loga pela primeira vez) e também por id (depois que ele logou). Uma solução terrível é duplicar seu objeto usuário como duas strings:

	set users:leto@dune.gov "{id: 9001, email: 'leto@dune.gov', ...}"
	set users:9001 "{id: 9001, email: 'leto@dune.gov', ...}"

Isto é ruim porque é um pesadelo para gerenciar e porque ocupa o dobro da memória.

Seria legal se o Redis deixasse você apontar uma chave para outra, mas ele não deixa (e provavelmente nunca vai deixar). Uma das principais diretrizes do desenvolvimento do Redis é manter o código e a API simples e claras. A implementação interna para ponteiros de chaves (há muitas operações que podemos fazer com chaves que não discutimos ainda) não vale a pena quando você considera que o Redis já tem uma solução: hashes.

Usando um hash, nós eliminamos a necessidade de duplicação:

	set users:9001 "{id: 9001, email: leto@dune.gov, ...}"
	hset users:lookup:email leto@dune.gov 9001

O que estamos fazendo é usar o campo como um pseudo-índice secundário e referenciando o objeto usuário único. Para obter um usuário por id, basta um `get` normal:

	get users:9001

Para obter o mesmo usuário por email, executamos um `hget` seguido por um `get` (em Ruby):

	id = redis.hget('users:lookup:email', 'leto@dune.gov')
	user = redis.get("users:#{id}")

Isto é algo que você vai acabar fazendo bastante. Para mim, é aqui que os hashes realmente brilham, mas não é um caso de uso óbvio até que você o veja pela primeira vez.

### Referências e Índices

Nós já vimos alguns exemplos de um valor apontando para outro. Vimos isso no nosso exemplo de listas, e vimos também na seção acima quando usamos hashes para facilitar um pouco as consultas. Essencialmente, isto se resume a ter que gerenciar manualmente os seus índices e referências entre valores. Sendo franco, eu acho que isso pode ser um pouco desestimulante, principalmente quando você pensa em ter que manter, atualizar e remover manualmente todas essas referências. Não há solução mágica para resolver esse problema no Redis.

Nós já vimos como os conjuntos são usados muitas vezes para resolver este tipo de índice manual:

	sadd friends:leto ghanima paul chani jessica

Cada membro deste conjunto é uma referência a um valor string do Redis contendo detalhes do usuário atual. E se `chani` mudar de nome, ou excluir a conta dela? Talvez faça sentido também acompanhar a relação inversa:

	sadd friends_of:chani leto paul

Ignorando os custos de manutenção, se você se parecer só um pouquinho comigo, vai se contorcer só de pensar no custo de memória e processamento de ter esses valores extra indexados. Na próxima seção, vamos discutir formas de reduzir o custo de performance de ter que fazer estes passos extras (falamos disso rapidamente no primeiro capítulo).

Mas se você parar para pensar, bancos de dados relacionais tem o mesmo trabalho extra. Índices ocupam memória, precisam ser varridos (ou idealmente lidos direto a partir da posição correta) e depois os registros correspondentes precisam ser localizados. O trabalho extra é abstraído elegantemente (e eles fazem várias otimizações de processamento para que isso seja muito eficiente).

Novamente, ter que lidar manualmente com referências no Redis é desagradável. Mas quaisquer preocupações iniciais que você tenha sobre o desempenho ou as implicações de memória devem ser testadas. Você vai descobrir que não há problema.

### Round Trips e Pipelines

Nós já mencionamos que consultas frequentes ao servidor é um padrão comum no Redis. Já que é algo que você vai fazer sempre, é bom dar uma olhada em alguns dos recursos que podemos usar para aproveitar isso ao máximo.

Primeiro, vários comandos aceitam um ou mais parâmetros, ou têm um comando irmão que aceita vários parâmetros. Nós vimos o `mget` mais cedo, que recebe múltiplas chaves e retorna os valores:

	keys = redis.lrange('newusers', 0, 10)
	redis.mget(*keys.map {|u| "users:#{u}"})

Ou o comando `sadd`, que adiciona um ou mais membros a um conjunto:

	sadd friends:vladimir piter
	sadd friends:paul jessica leto "leto II" chani

O Redis também suporta pipelines. Normalmente, quando um cliente envia uma requisição ao Redis, ele espera a resposta antes de enviar a próxima requisição. Com pipelines, você pode enviar qualquer número de requisições sem esperar pelas respostas. Isto reduz a comunicação de rede e pode resultar em ganhos significativos de desempenho.

É importante frisar que o Redis vai usar a memória para enfileirar os comandos, então é uma boa ideia agrupá-los em lotes. O tamanho do lote vai depender dos comandos que você está usando e, mais especificamente, do tamanho dos parâmetros. Mas se você está executando comandos em chaves de ~50 caracteres, você provavelmente pode agrupá-los em milhares ou dezenas de milhares.

Exatamente quais comandos você executa em um pipeline vai variar de driver para driver. Em Ruby, você passa um bloco para o método `pipelined`:

	redis.pipelined do
	  9001.times do
		redis.incr('powerlevel')
	  end
	end

Como você já deve ter percebido, pipelines podem agilizar bastante uma importação em lotes!

### Transações

Todos os comandos do Redis são atômico, inclusive os que realizam múltiplas tarefas. Adicionalmente, o Redis tem suporte a transações quando se usa múltiplos comandos.

Você pode não saber, mas o Redis roda em apenas uma thread, que é como se pode garantir que todos os comandos são atômicos. Enquanto um comando está sendo executado, nenhum outro comando vai rodar (vamos falar daqui a pouco sobre escalabilidade, nos próximos capítulos). Isso é particularmente útil quando você considera que alguns comandos podem fazer várias coisas. Por exemplo:

`incr` é essencialmente um `get` seguido por um `set`

`getset` define um novo valor e retorna o original

`setnx` primeiro checa se a chave existe, e só define o valor caso não exista

Apesar desses comandos serem úteis, você inevitavelmente vai precisar rodar vários comandos como um grupo atômico. Você faz isso rodando primeiro o comando `multi`, seguido por todos os comandos que você que executar como parte da transação, e finalmente rodando o `exec` para executar de fato os comandos ou `discard` para desprezá-los, e não executar os comandos. Que garantia o Redis dá sobre as transações?

* Os comandos serão executados na ordem

* Os comandos serão executados como uma operação atômica única (sem que outro comando cliente seja executado entre eles)

* Que serão executados todos ou nenhum dos comandos da transação

Você pode, e deveria, testar este comando na linha de comando. Note também que não há motivo pelo qual você não possa combinar pipelines e transações.

	multi
	hincrby groups:1percent balance -9000000000
	hincrby groups:99percent balance 9000000000
	exec

Por fim, o Redis deixa você especificar uma ou mais chaves para observar e aplicar condicionalmente uma transação quando as chaves mudarem. Isto é usado quando você precisa obter valores e executar código baseado nesses valores, tudo em uma transação. Com o código acima, nós não conseguiríamos implementar nosso próprio comando `incr`, já que eles seriam executados todos ao mesmo tempo quando o o `exec` for chamado. A partir do código, não podemos fazer:

	redis.multi()
	current = redis.get('powerlevel')
	redis.set('powerlevel', current + 1)
	redis.exec()

Não é assim que as transações funcionam no Redis. Mas, se você adicionar um `watch` a `powerlevel`, poderá fazer:

	redis.watch('powerlevel')
	current = redis.get('powerlevel')
	redis.multi()
	redis.set('powerlevel', current + 1)
	redis.exec()

Se outro cliente mudar o valor de `powerlevel` após termos usado o `watch` nele, nossa transação vai falhar. Se nenhum cliente mudar o valor, o `set` vai funcionar. Nós podemos executar este código em um loop até ele dar certo.

### Anti-padrões de Chaves

No próximo capítulo, vamos falar de comandos que não estão relacionados a nenhuma estrutura de dados específica. Alguns destes são ferramentas administrativas e de depuração. Mas há um em particular do qual eu gostaria de falar: o comando `keys`. Este comando recebe um padrão e encontra todas as chaves que se encaixem nele. Este comando parece ser útil para várias tarefas, mas nunca deve ser usado em código de produção. Porque? Porque ele faz uma varredura linear por todas as chaves, vendo quais se encaixam. Ou, simplesmente, é lento.

Como as pessoas usam-no? Digamos que você está escrevendo um sistema de acompanhamento de bugs. Cada conta vai ter um `id` e você pode decidir guardar cada bug em uma string com uma chave do tipo `bug:account_id:bug_id`. Se você algum dia precisar encontrar todos os bugs de uma conta (para exibi-los, ou talvez removê-los se alguém cancelar uma conta), você pode ficar tentado (como eu fiquei!) a usar o comando `keys`:

	keys bug:1233:*

A melhor solução é usar um hash. De forma bem parecida a como usamos hashes para prover uma forma de expor índices secundários, também podemos usá-los para organizar nossos dados:

	hset bugs:1233 1 "{id:1, account: 1233, subject: '...'}"
	hset bugs:1233 2 "{id:2, account: 1233, subject: '...'}"

Para obter todos os ids de bugs de uma conta, nós simplesmente rodamos `hkeys bugs:1233`. Para excluir um bug específico nós podemos fazer `hdel bugs:1233 2`, e para excluir uma conta nós podemos apagar a chave usando `del bugs:1233`.


### Neste Capítulo

Esperamos que este capítulo, junto com o anterior, tenha lhe dado uma boa visão sobre como usar o Redis para implementar recursos reais. Há vários outros padrões que você pode usar para construir todo tipo de coisa, mas o segredo é entender as estruturas de dados fundamentais e como elas podem ser usadas para atingir coisas além da sua perspectiva inicial.

\clearpage


## Capítulo 4 - Além das Estruturas de Dados

Enquanto as cinco estruturas de dados formam o alicerce do Redis, há outros comandos que não são específicos de nenhuma estrutura de dados. Já vimos alguns destes: `info`, `select`, `flushdb`, `multi`, `exec`, `discard`, `watch` e `keys`. Este capítulo vai tratar de alguns outros comandos importantes.

### Expiração

O Redis permite que você marque uma chave para expiração. Você pode fornecer uma hora absoluta no formato de timestamp Unix (segundos desde 1º de janeiro de 1970) ou um tempo de vida em segundos. Este comando se baseia em chaves, então não importa que tipo de estrutura de dados a chave representa.

	expire pages:about 30
	expireat pages:about 1356933600

O primeiro comando vai apagar a chave (e os valores associados a ela) após 30 segundos. O segundo vai fazer o mesmo no dia 31 de dezembro de 2012 ao meio-dia.

Isto faz do Redis um mecanismo de cache ideal. Você pode descobrir quanto tempo de vida um item possui através do comando `ttl`, e pode remover a expiração de uma chave usando o `persist`:

	ttl pages:about
	persist pages:about

Por fim, há um comando especial de strings – `setex` – que lhe permite definir uma string e especificar um tempo de vida em um só comando atômico (mais por conveniência mesmo):

	setex pages:about 30 '<h1>about us</h1>....'

### Publicação e Assinaturas

As listas do Redis possuem os comandos `blpop` e `brpop`, que retornam e removem o primeiro (ou último) elemento da lista, ou bloqueiam enquanto não houver um disponível. Eles podem ser usados para implementar uma fila simples.

Além disso, o Redis possui suporte de primeira classe a publicação de mensagens e assinatura de canais. Você pode testar isso abrindo uma segunda janela com o `redis-cli`. Na primeira janela, assine um canal (vamos chamá-lo de `warnings`):

	subscribe warnings

A resposta é a informação da sua assinatura. Agora, na outra janela, publique uma mensagem no canal `warnings`:

	publish warnings "it's over 9000!"

Se você voltar para a primeira janela, você deve ter recebido a mensagem no canal `warnings`.

Você pode assinar vários canais (`subscribe channel1 channel2 ...`), assinar um padrão de canais (`psubscribe warnings:*`) e usar os comandos `unsubscribe` e `punsubscribe` para cancelar a assinatura de um ou mais canais, ou de um padrão de canais.

Finalmente, note que o comando `publish` retornou o valor 1. Isto indica o número de clientes que receberam a mensagem.


### Monitor e Slow Log

O comando `monitor` lhe permite saber o que o Redis anda fazendo. É uma grande ferramenta de depuração que lhe permite ver de que forma a aplicação está interagindo com o Redis. Em uma das suas duas janelas do `redis-cli` (se uma ainda está com assinaturas, você pode usar o comando `unsubscribe` ou simplesmente fechar a janela e abrir outra) digite o comando `monitor`. Em outra, execute qualquer tipo de comando (como `get` ou `set`). Você deve ver estes comandos, bem como seus parâmetros, na primeira janela.

Evite rodar o `monitor` em produção - ele é mais uma ferramenta de desenvolvimento e depuração. Fora isso, não há muito o que falar dele. É apenas uma ferramenta muito útil.

Além do `monitor`, o Redis possui um comando `slowlog`, que funciona como uma ótima ferramenta de profile. Ele loga qualquer comando que demore mais que um número especificado de **microssegundos**. Na próxima seção, nós vamos cobrir resumidamente como configurar o Redis; por enquanto, você pode configurar o Redis para logar todos os comandos digitando:

    config set slowlog-log-slower-than 0

A seguir, digite alguns comandos. Quando terminar, você pode exibir todos os logs – ou apenas os mais recentes – executando:

	slowlog get
	slowlog get 10

Você também pode pegar o número de items no slow log digitando `slowlog len`.

Para cada comando digitado, você deve ver quatro parâmetros:

* Um id auto-incrementado

* Um timestamp Unix de quando o comando aconteceu

* O tempo que o comando levou para rodar (em microssegundos)

* O comando e seus parâmetros

O slow log é guardado em memória, então não deve ser um problema habilitá-lo em produção, mesmo com um limite baixo. Por padrão, ele vai manter os últimos 1024 logs.

### Ordenação

Um dos comandos mais poderosos do Redis é o `sort`. Ele permite que você ordene os valores de uma lista, set ou sorted set (sorted sets são ordenados por score, não pelos membros dentro do set). Em sua forma mais simples, ele nos permite fazer:

	rpush users:leto:guesses 5 9 10 2 4 10 19 2
	sort users:leto:guesses

Isso vai retornar os valores ordenados do menor para o maior. Aqui vai um exemplo mais avançado:

	sadd friends:ghanima leto paul chani jessica alia duncan
	sort friends:ghanima limit 0 3 desc alpha

O comando acima nos mostra como paginar as entradas ordenadas (com `limit`), como retornar os resultados em ordem decrescente (com `desc`) e como ordenar lexicograficamente em vez de numericamente (com `alpha`).

O verdadeiro poder do `sort` é sua habilidade de ordenar baseado num objeto referenciado. Mais cedo, nós mostramos como as listas, sets e sorted sets são usados muitas vezes para referenciar outros objetos no Redis. O comando `sort` pode seguir essas referências e ordenar pelo valor subjacente. Por exemplo, digamos que nós temos um sistema de acompanhamento de bugs que permite aos usuários observar determinados bugs. Nós podemos usar um set para acompanhar quais são os bugs observados:

	sadd watch:leto 12339 1382 338 9338

Pode fazer total sentido ordenar esses bugs por id (que é o que a ordenação padrão vai fazer), mas nós também podemos querer ordená-los por gravidade. Para isso, nós dizemos ao Redis por qual padrão ordenar. Primeiro, vamos adicionar mais alguns dados para podermos ver um resultado significativo:

	set severity:12339 3
	set severity:1382 2
	set severity:338 5
	set severity:9338 4

Para ordenar os bugs por gravidade, da maior para a menor, você faria:

	sort watch:leto by severity:* desc

O Redis vai substituir o `*` do nosso padrão (identificado pelo `by`) com os valores na nossa lista/set/sorted set. Isto vai criar o nome da chave que o Redis vai consultar para pegar os valores que serão usados de fato na ordenação.

Apesar de você poder ter milhões de chaves dentro do Redis, eu acho que o método acima pode acabar um pouco bagunçado. Ainda bem que o `sort` também pode trabalhar com hashes e seus campos. Em vez de um monte de chaves de primeiro nível, você pode tirar proveito dos hashes:

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

Assim, não apenas fica tudo mais organizado – e nós podemos ordenar por gravidade ou prioridade – como também podemos dizer qual campo o `sort` deve retornar:

	sort watch:leto by bug:*->priority get bug:*->details

Ocorre a mesma substituição de valor, mas o Redis também reconhece o operador `->` e usa-o para procurar o campo especificado no nosso hash. Nós também incluímos o parâmetro `get`, que faz a substituição e a busca de campos da mesma forma, para trazer os detalhes do bug.

O `sort` pode ser lento em sets grandes. A boa notícia é que a saída de um `sort` pode ser guardada:

	sort watch:leto by bug:*->priority get bug:*->details store watch_by_priority:leto

Combinar a capacidade de guardar os resultados do `sort` com os comandos de expiração que já vimos antes resulta num ótimo combo.


### Neste Capítulo

Este capítulo focou nos comandos que não são específicos às estruturas de dados. Como todo o resto, o uso deles depende da situação. É normal construir uma aplicação ou recurso que não usa expiração, publicação/assinatura nem ordenação; mas é bom saber que eles estão ali. Além disso, nós tocamos em apenas alguns comandos. Há mais, e, uma vez que você tenha digerido o material deste livro, vale a pena olhar a [lista completa](http://redis.io/commands).

\clearpage

## Capítulo 5 - Administração

Nosso último capítulo é dedicado a alguns aspectos administrativos do Redis. De forma alguma isso é um guia abrangente de administração do Redis. No melhor dos casos, vamos responder algumas das dúvidas mais básicas que novos usuários do Redis provavelmente vão ter.

### Configuração

Quando você rodou o servidor do Redis pela primeira vez, ele lhe avisou que o arquivo `redis.conf` não pôde ser encontrado. Este arquivo pode ser usado para configurar vários aspetos do Redis. Um arquivo `redis.conf` bem documentado está disponível em cada versão do Redis. O arquivo de exemplo contém as opções padrão de configuração, sendo útil tanto para entender o que os ajustes fazem quanto para saber quais são os valores padrão de cada ajuste. Você pode encontrá-lo em <https://github.com/antirez/redis/raw/2.4.6/redis.conf>.

**Este é o arquivo de configuração do Redis 2.4.6. Você deve substituir o "2.4.6" na URL acima com a sua versão. Você pode descobrir sua versão executando o comando `info` e olhando o primeiro valor.**

Como o arquivo é bem documentado, nós não vamos analisar cada ajuste.

Além de configurar o Redis pelo arquivo `redis.conf`, podemos usar o comando `config set` para mudar valores individuais. Na verdade, nós já usamos o `config set` quando passamos a configuração `slowlog-log-slower-than` para 0.

Também existe o comando `config get`, que exibe o valor de uma configuração. Este comando suporta a busca por padrões. Assim, se queremos exibir tudo relacionado a logs, podemos fazer:

   config get *log*

### Authentication

O Redis pode ser configurado para exigir uma senha. Isto pode ser feito através da configuração `requirepass` (que pode ser informada tanto no `redis.conf` quando com o comando `config set`). Quando o `requirepass` possui um valor configurado (que é a senha que deve ser utilizada), os clientes vão precisar executar um comando `auth password`.

Uma vez que o cliente esteja autenticado, ele poderá executar qualquer comando em qualquer banco de dados. Isto inclui o `flushall`, que apaga todas as chaves de todos os bancos. Na configuração, você pode renomear os comandos para obter um pouco de segurança por obfuscação:

	rename-command CONFIG 5ec4db169f9d4dddacbfb0c26ea7e5ef
	rename-command FLUSHALL 1041285018a942a4922cbf76623b741e

Ou você pode desabilitar um comando, modificando o novo nome para uma string vazia.

### Limitações de Tamanho

Enquanto você está começando a usar o Redis, você pode se perguntar "quantas chaves eu posso guardar?" Você pode também pensar quantos campos um hash pode ter (especialmente quando você usa-os para organizar seus dados), ou quantos elementos as listas e sets podem ter. Por instância, o limite prático para todos estes é na casa das centenas de milhões.

### Replicação

O Redis suporta replicação, o que significa que enquanto você escreve em uma instância do Redis (o master, ou "mestre"), uma ou mais outras instâncias (os slaves, ou "escravos") são mantidas atualizadas pelo master. Para configurar um slave, você pode usar tanto a configuração `slaveof` quanto o comando `slaveof` (instâncias que rodam sem esta configuração são ou podem ser masters).

A replicação ajuda a proteger seus dados copiando-os para servidores diferentes. A replicação também pode ser usada para melhorar o desempenho, já que as leituras podem ser feitas a partir dos slaves. Eles podem responder com dados ligeiramente desatualizados, mas para a maioria das aplicações esta é uma troca que vale a pena.

Infelizmente, a replicação do Redis não provê ainda failover automático. Se o master morrer, um slave precisa ser promovido manualmente. Ferramentas tradicionais de alta disponibilidade que usem monitoração de heartbeats e scripts para automatizar a troca são atualmente dores-de-cabeça necessárias se você quer atingir algum grau de alta disponibilidade com o Redis.

### Backups

Fazer backup do Redis é uma simples questão de copiar o snapshot do Redis para onde quer que você queira (S3, FTP, ...). Por padrão, o Redis salva o snapshot em um arquivo chamado `dump.rdb`. Você pode simplesmente copiar este arquivo a qualquer momento com `scp`, `ftp` ou `cp` (ou qualquer outra coisa).

Não é incomum desabilitar tanto a gravação de snapshots quanto o arquivo append-only (aof) no master e deixar um slave tomar conta disso. Isso ajuda a reduzir a carga no master e lhe permite definir parâmetros de gravação mais agressivos no slave, sem ferir a responsividade geral do sistema.

### Escalabilidade e Cluster Redis

Replicação é a primeira ferramenta da qual um site em crescimento pode tirar vantagem. Alguns comandos são mais custosos que outros (`sort` por exemplo) e despachar a execução deles para um slave pode manter o sistema respondendo bem às consultas que chegam.

Além disso, escalar de verdade o Redis acaba chegando em distribuir suas chaves em várias instâncias do Redis (que podem rodar na mesma máquina - lembre-se: o Redis só possui uma thread). Até agora, isso é algo que é você quem vai precisar resolver (apesar que vários drivers de Redis provêem algoritmos de hash consistentes). Pensar seus dados em termos de distribuição horizontal é algo que não vamos cobrir neste livro. Também é algo que você provavelmente não vai ter que se preocupar por um bom tempo, mas é algo do qual você precisa estar ciente independentemente da solução que você use.

A boa notícia é que já existem esforços no sentido de um Cluster Redis. Isso não apenas vai oferecer escalabilidade horizontal – incluindo rebalanceamento – como também vai fornecer failover automático para alta disponibilidade.

Alta disponibilidade e escalabilidade são algo que pode ser atingido hoje, contanto que você esteja disposto a direcionar seu tempo e esforço para isto. Mais adiante, o Cluster Redis deve deixar as coisas bem mais fáceis.

### Neste Capítulo

Dado o número de projetos e sites que já usam Redis, não resta dúvida que o Redis está pronto para produção, e que já tem estado assim por um bom tempo. Entretanto, algumas das ferramentas - especialmente as de segurança e disponibilidade - ainda são muito "verdes". O Cluster Redis, que esperamos ver em breve, deve ajudar a resolver alguns dos desafios de administração atuais.

\clearpage

## Conclusão

De várias maneiras, o Redis representa uma simplificação na forma como lidamos com dados. Ele remove muito da complexidade e abstração disponíveis em outros sistemas. Em vários casos, isso faz do Redis a escolha errada. Em outros casos, é como se o Redis tivesse sido feito especificamente para os seus dados.

No fim das contas voltamos para algo que eu disse bem no início: o Redis é fácil de aprender. Há muitas tecnologias novas, e pode ser difícil descobrir em quais vale a pena investir o tempo de aprender. Quando você considera os reais benefícios que o Redis tem a oferecer com sua simplicidade, eu acredito sinceramente que ele é um dos melhores investimentos – em termos de aprendizado – que você e sua equipe podem fazer.