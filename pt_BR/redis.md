# Sobre Este Livro

## Licença

The Little Redis Book é licenciado sob a licença Creative Commons de Atribuição - Uso Não Comercial 3.0 Unported. Você não deve pagar por este livro.

Sinta-se livre para copiar, distribuir, modificar ou expor o livro. Entretanto, peço que você sempre atribua este livro a mim, Karl Seguin, e não use-o para propósitos comerciais.

Você pode ver o *texto completo* da **licença em**:

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>

## Sobre o Autor

Karl Seguin é um desenvolvedor com experiência em vários campos e tecnologias. Ele é um contribuidor ativo em projetos de Software Livre, um escritor técnico e palestrante ocasional. Ele escreveu diversos artigos sobre e algumas ferramentas para Redis. Redis é usado no ranking e estatística de seu serviço gratuito para desenvolvedores casuais de jogos: [mogade.com](http://mogade.com/).

Karl escreveu [The Little MongoDB Book](http://openmymind.net/2011/3/28/The-Little-MongoDB-Book/), um livro gratuito e popular sobre MongoDB.

Seu blog pode ser acessado em <http://openmymind.net> e seus tweets via [@karlseguin](http://twitter.com/karlseguin)

## Agradecimentos

Um obrigado especial para [Perry Neal](https://twitter.com/perryneal) por ter me emprestado seus olhos, mente e paixão. Você me forneceu uma ajuda sem preço. Obrigado.

## Versão Mais Atual

O código mais recente deste livro está disponível em:
<http://github.com/karlseguin/the-little-redis-book>

# Introdução

Nos últimos anos, as técnicas e ferramentas usadas para persistência e busca de dados têm crescido num ritmo impressionante. Ao mesmo tempo em que é seguro dizer que os bancos de dados relacionais não vão a lugar algum, nós também podemos dizer que o ecossistema de dados nunca será o mesmo.

De todas as ferramentas e soluções novas, para mim, Redis tem sido a mais excitante. Porquê? Primeiramente porque é inacreditavelmente fácil de aprender. Pode-se falar em termos de horas para descrever a quantidade de tempo necessária para sentir-se confortável com Redis. Em segundo lugar, ele resolve um conjunto específico de problemas e ao mesmo consegue ser bastante genérico. O que isso significa exatamente? Redis não tenta ser a solução de todos os problemas para qualquer tipo de dado. Ao conhecer melhor o Redis, se torna mais e mais evidente que tipo de problemas ele pode ou não resolver. E quando ele pode, como um desenvolvedor, é uma grande experiência.

Embora você possa construir um sistema inteiro usando somente Redis, eu acho que a maioria das pessoas irá descobrir que ele é uma adição à sua solução genérica de dados - seja ela um banco de dados relacional, um sistema orientado a documentos ou qualquer outra coisa. É o tipo de solução que você usa para implementar funcionalidades específicas. Dessa forma, ele se assemelha a um mecanismo de indexação. Você não iria construir sua aplicação inteira em Lucene. Mas quando você precisa de uma boa busca, é uma experiência muito melhor - tanto para você como para seus usuários. É claro que as semelhanças entre o Redis e os mecanismos de indexação acabam aqui.

O objetivo deste livro é construir a fundação que você precisa para dominar o Redis. Nós iremos focar em aprender os cinco tipos de estruturas de dados do Redis e olhar diversas abordagens para a modelagem de dados. Nós também iremos tocar em alguns detalhes chave da parte administrativa e técnicas de depuração.

# Começando

Todos aprendemos de maneiras diferentes: alguns gostam de botar a mão na massa, outros de assistir vídeos e alguns de ler. Nada irá ajudar você a entender Redis mais do que utilizá-lo. Redis é fácil de instalar e vem com um console simples que nos dará tudo que precisamos. Vamos gastar alguns minutos para deixá-lo pronto para rodar em nossa máquina.

## No Windows

O Redis em si não suporta o Windows oficialmente, mas existem algumas opções disponívels. Você não deveria rodá-las em produção, mas eu nunca experimentei limitações no ambiente de desenvolvimento.

Um port feito pela Microsoft Open Technologies, Inc. pode ser encontrado em <https://github.com/MSOpenTech/redis>. Quando este livro foi escrito, a solução não estava pronta para ser usada em ambiente de produção.

Outra solução, que já está disponível há algum tempo, pode ser encontrada em <https://github.com/dmajkic/redis/downloads>. Você pode baixar a versão mais recente (que deve estar no topo da lista). Extraia o arquivo zip e, dependendo da sua arquitetura, abra a pasta `64bit` ou `32bit`.

## No *nix e MacOSX

Para usuários de *nix e Mac, a melhor opção é compilar a partir do código-fonte. As instruções, juntamente com o número da versão mais recente, estão disponíveis em <http://redis.io/download>. No tempo em que este livro foi escrito, a versão mais recente era a 2.6.2. Para instalar esta versão nós executaríamos:

    wget http://redis.googlecode.com/files/redis-2.6.2.tar.gz
    tar xzf redis-2.6.2.tar.gz
    cd redis-2.6.2
    make

Alternativamente, o Redis está disponível via diversos gerenciadores de pacote. Por exemplo, usuários de MacOSX com Homebrew instalado pode simplesmente digitar `brew install redis`.)

Se você compilar do fonte, as saídas binárias serão colocadas na pasta `src`. Acesse a pasta `src` executando o comando `cd src`.

## Rodando e Conectando ao Redis

Se tudo funcionou, os binários do Redis devem estar disponíveis nas pontas de seus dedos. Redis tem um monte de executáveis. Nós iremos focar no servidor do Redis e na interface em linha de comando (como um cliente DOS). Vamos iniciar o servidor. No Windows, dê um clique duplo em `redis-server`. No *unix/MacOSX rode o comando `./redis-server`.

Na mensagem de inicialização você verá um aviso informando que o arquivo `redis.conf` não foi encontrado. Redis irá então usar as configurações padrão – isso basta para o que nós vamos fazer.

Em seguida inicie o console do Redis dando um clique duplo em `redis-cli` (no Windows) ou executando o comando `./redis-cli` (no *nix/MacOSX). Isso irá lhe conectar ao servidor local rodando na porta padrão (6379).

Você pode testar se tudo está funcionando digitando `info` na interface de linha de comando. Então você verá uma porção de pares chave-valor que fornecem uma boa idéia sobre o estado do servidor.

Se você está tendo problemas com o procedimento anterior, eu sugiro que você busque ajuda no [grupo oficial de suporte do Redis](https://groups.google.com/forum/#!forum/redis-db).

# Drivers do Redis

Como você vai aprender em breve, a API do Redis é melhor definida como um conjuto explícito de funções. Ela tem um aspecto simples e processual. Isto significa que tanto usando a ferramenta da linha de comando como um driver para sua linguagem favorita, vai ser tudo muito parecido. Portanto, você não deve ter problemas problemas em acompanhar os passos seguintes se você preferir usar uma linguagem de programação. Se preferir, siga para a [página de clientes] e baixe o driver apropriado.

# Capítulo 1 - O Básico

O que torna o Redis especial? Que tipos de problemas ele resolve? Em quê os desenvolvedores precisam prestar atenção ao usá-lo? Antes de responder qualquer uma destas perguntas, precisamos entender o que é o Redis.

O Redis é frequentemente descrito como um armazenamento persistente de chave-valor em memória. Eu não acho que esta seja uma descrição precisa. O Redis mantém todos os dados em memória (veremos mais sobre isso daqui a pouco), e realmente escreve em disco para garantir persistência, mas é muito mais do que um simples armazenamento de chave-valor. É importante ir além deste conceito equivocado; caso contrário, sua perspectiva sobre o Redis e os problemas que ele resolve será muito limitada.

A realidade é que o Redis expõe cinco estruturas de dados diferentes, e apenas uma delas é um típico par chave-valor. Entender essas cinco estruturas de dados, como elas funcionam, quais métodos elas expõem e o que você pode modelar com elas é a chave para entender o Redis. Primeiro, porém, vamos tentar entender o que significa expor estruturas de dados.

Se fôssemos aplicar este conceito de estruturas de dados ao mundo relacional, poderíamos dizer que bancos de dados expõem uma única estrutura de dados - tabelas. Tabelas são tanto complexas quanto flexíveis. Não há muito o que você não possa modelar, guardar ou manipular com elas. No entanto, sua natureza genérica também tem suas desvantagens. Especificamente, nem tudo é tão simples, ou tão rápido, quanto deveria ser. E se, ao invés de termos uma estrutura única que resolve tudo, nós utilizássemos estruturas mais especializadas? Poderia haver algumas coisas que não conseguiríamos fazer (ou pelo menos, não conseguiríamos fazer tão bem), mas certamente ganharíamos em simplicidade e velocidade?

Usar estruturas de dados específicas para problemas específicos? Não já é assim que nós programamos? Você não usa uma tabela hash para todos os tipos de dados, nem uma variável escalar. Para mim, isto define a abordagem do Redis. Se você está lidando com escalares, listas, hashes, ou conjuntos, por que não armazená-los como escalares, listas, hashes e conjuntos? Por que checar a existência de um valor deve ser mais complexo do que simplesmente chamar `exists(key)` ou mais lento do que O(1) (tempo constante de busca que não vai demorar mais independente da quantidade de itens)?

# Os Blocos de Construção

## Bancos de Dados

O Redis possui o mesmo conceito básico de um banco de dados com o qual você já está familiarizado: um banco de dados contém um conjunto de dados. O caso de uso típico de um banco de dados é agrupar todos os dados de uma aplicação e mantê-los separados dos dados de outra aplicação.

No Redis, bancos de dados são simplesmente identificados por um número, sendo o padrão o número `0`. Se você quiser mudar para um banco de dados diferente, você pode fazê-lo com o comando `select`. Na interface de linha de comando, digite `select 1`. O Redis deve responder com uma mensagem `OK` e seu prompt deve mudar para algo como `redis 127.0.0.1:6379[1]>`. Se você quiser voltar para o banco de dados padrão, apenas digite `select 0` na linha de comando.

## Comandos, Chaves e Valores

Mesmo que o Redis seja mais do que apenas um armazenamento chave-valor, no seu cerne, cada uma de suas cinco estruturas de dados possuem pelo menos uma chave e um valor. É primordial que entendamos chaves e valores antes de continuar com os outros tipos de informação disponíveis.

Chaves são como você identifica seções de dados. Vamos lidar muito com chaves, mas, por enquanto, basta saber que uma chave pode ser algo do tipo `users:leto`. Pode-se esperar que tal chave contenha informações sobre um usuário chamado `leto`. Os dois pontos não têm nenhum significado especial para o Redis, mas usar um separador é um padrão bastante utilizado para organizar chaves.

Valores representam os dados efetivamente associados com uma chave. Podem ser qualquer coisa. Algumas vezes você irá guardar strings, outras vezes números inteiros, outras irá guardar objetos serializados (em JSON, XML ou algum outro formato). A maioria parte do tempo, o Redis trata valores como uma matriz de bytes, sem importar com o que eles são. Note que cada driver trata a serialização de uma forma diferente (alguns deixam isso para você) então neste livro iremos apenas falar sobre strings, inteiros e JSON.

Vamos sujar um pouco as mãos. Digite o seguinte comando:

	set users:leto '{"name": "leto", "planet": "dune", "likes": ["spice"]}'

Esta é a anatomia básica de um comando do Redis. Primeiro nós temos o comando de fato, neste caso o `set`. Depois temos seus parâmetros. O comando `set` recebe dois argumentos: a chave que estamos definindo e o valor que atribuímos a ela. Muitos, mas não todos, comandos recebem uma chave (e, quando recebem, muitas vezes é o primeiro argumento). Você consegue adivinhar como pegar este valor? Espero que você consiga (mas não se preocupe se não tinha certeza!):

	get users:leto

Siga em frente e brinque com outras combinações. Chaves e valores são conceitos fundamentais, e os comandos `get` e `set` são a maneira mais simples de brincar com eles. Crie mais usuários, tente outros tipos de chaves, tente valores diferentes.

## Consultando

A medida que prosseguirmos, duas coisas se tornarão claras. No que diz respeito ao Redis, chaves são tudo e valores não são nada. Ou, de outra forma, o Redis não permite que você faça consultas nos valores de um objeto. Face ao exposto, não podemos encontrar os usuários que vivem no planeta `dune`.

Para muitos, isto pode ser preocupante. Vivemos em um mundo onde as consultas de dados são tão flexíveis e poderosas que a abordagem do Redis parece primitiva e nem um pouco prática. Não deixe que isto lhe perturbe muito. Lembre-se: Redis não é a solução para tudo. Haverá coisas que simplesmente não se encaixam nele (devido à limitação das consultas). Além disso, considere que em alguns casos você vai encontrar novas formas de modelar seus dados.

Vamos analisar exemplos mais concretos à medida que avançamos, mas é importante entender que esta é a realidade básica do Redis. Ela nos ajudará a entender porque valores podem ser qualquer coisa - o Redis nunca precisa ler ou entendê-los. Isto também ajuda nossa mente a pensar sobre a modelagem neste novo mundo.

## Memória e Persistência

Já dissemos que o Redis é um armazenamento persistente em memória. Em relação à persistência, por padrão, o Redis escreve uma versão do banco de dados no disco baseado em quantas chaves foram alteradas. Você configura isto para que, se um número X de chaves mudar, então salve o banco de dados a cada Y segundos. Por padrão, o Redis irá salvar o banco de dados a cada 60 segundos se 1000 ou mais chaves foram alteradas, e vai levar até 15 minutos para salvar novamente se 9 ou menos chaves foram alteradas.

Alternativamente (ou em adição à geração de instantâneos), o Redis pode rodar em modo append. Toda vez que uma chave mudar, um arquivo somente-adição é atualizado no disco. Em alguns casos é aceitável perder 60 segundos de dados, em troca de performance, caso haja alguma falha de hardware ou software. Em alguns casos é inaceitável. O Redis lhe dá a opção. No capítulo 6 veremos uma terceira opção, que é delegar a persistência a um escravo.

Quanto à memória, o Redis mantém todos os dados em memória. A implicação óbvia disto é o custo de rodar o Redis: a parte mais cara do hardware do servidor ainda é a RAM.

Eu tenho a impressão que alguns desenvolvedores perderam a noção de quão pouco espaço os dados podem ocupar. A obra completa de William Shakespeare ocupa cerca de 5.5MB de armazenamento. Quanto à escalabilidade, outras soluções tendem a ser limitadas por IO ou CPU. Qual limitaçõo (RAM ou IO) será necessária para você escalar para mais máquinas realmente vai depender do tipo de dado e de como você está armazenando e consultando. A menos que você esteja guardando grandes arquivos multimídia no Redis, o fato dele guardar tudo em memória provavelmente não será problema. Para aplicações onde são, você provavelmente optará por ser limitado a IO do que a memória.

O Redis adicionou suporte a memória virtual. No entanto, este recurso foi visto como erro (pelos próprios desenvolvedores do Redis) e seu uso foi tornado obsoleto.

(Para informação, os 5.5MB de arquivos com a obra completa de Shakespeare podem ser compactados em quase 2MB. O Redis não faz compressão automática, mas, como ele trata valores como bytes, não há razão para que você não possa trocar tempo de processamento por memória compactando/descompactando os dados você mesmo.)

## Juntando Tudo

Nós tivemos contato com vários tópicos de alto nível. A última coisa que eu gostaria de fazer antes de mergulhar no Redis é juntar alguns desses tópicos. Especificamente, as limitações de consulta, estruturas de dados e a maneira com a qual o Redis guarda os dados em memória.

Quando você une estas três coisas termina com algo maravilhoso: velocidade. Algumas pessoas imaginam "Claro que o Redis é rápido, está tudo em memória." Mas isso é apenas uma parte da história. O motivo real do Redis se destacar entre outras soluções está em suas estruturas de dados especializadas.

Quão rápido? Depende de muita coisa - quais comandos você está utilizando, o tipo de dados, e por aí vai. Mas o desempenho do Redis costuma ser medido em dezenas de milhares, ou centenas de milhares de operações **por segundo**. Você pode executar o `redis-benchmark` (que está no mesmo diretório do `redis-server` e do `redis-cli`) para testar você mesmo.

Uma vez eu alterei um código que usava em um modelo tradicional para usar o Redis. Um teste de carga que escrevi levou mais de 5 minutos para finalizar usando o modelo relacional. No Redis, levou 150ms. Nem sempre você tem este ganho massivo de performance, mas espero que isto dê uma idéia a respeito do que estamos falando.

É importante entender este aspecto do Redis porque ele impacta em como você interage com ele. Desenvolvedores com experiência em SQL frequentemente trabalham em minimizar o número de consultas que fazem ao banco de dados. É um bom conselho para qualquer sistema, incluindo o Redis. No entanto, como estamos lidando com estruturas de dados mais simples, em algumas ocasiões vamos precisar ir ao servidor do Redis várias vezes para atingir nosso objetivo. Tais padrões de acesso a dados podem não parecer naturais à primeira vista, mas na realidade o custo tende a ser insignificante comparado a toda a performance que ganhamos.

## Neste Capítulo

Mesmo mal tendo brincado com o Redis, nós cobrimos uma vasta gama de assuntos. Não se preocupe se algo não ficou muito claro - como as consultas. No próximo capítulo nós vamos colocar a mão na massa e as perguntas que você está se fazendo agora certamente se responderão.

Os pontos importantes deste capítulo são:

* Chaves são strings que identificam seções dos dados (valores)

* Valores são matrizes de bytes arbitrários com os quais o Redis não se importa

* O Redis expõe (e é implementado como) cinco estruturas de dados especializadas

* Combinados, os pontos acima deixam o Redis rápido e fácil de usar, mas não adequado para todos os cenários

# Capítulo 2 - As Estruturas de Dados

É hora de ver as 5 estruturas de dados do Redis. Vamos explicar o que cada uma faz, quais métodos estão disponíveis e em quais tipos de recursos você pode usá-las.

As únicas construções do Redis que vimos até agora foram os comandos, chaves e valores. Até então, nada concreto sobre estruturas de dados. Quando usamos o comando `set`, como o Redis sabia que devia usar uma string? Acontece que cada comando é específico para uma estrutura de dados. Por exemplo, quando você usa `set`, você está guardando o valor numa estrutura do tipo string. Quando você usa `hset`, você está guardando o mesmo valor num hash. Dado o pequeno tamanho do vocabulário do Redis, isso é bem gerenciável.

**[O site do Redis](http://redis.io/commands) tem uma ótima documentação de referência. Não há porque repetir o trabalho que eles já fizeram. Vamos cobrir apenas os comandos mais importantes para entender o propósito de uma estrutura de dados.**

Não há nada mais importante que se divertir e fazer testes. Você sempre pode apagar todos os valores de um banco de dados com o `flushdb`, então não se intimide e tente fazer coisas loucas!

## Strings

Strings são a estrutura de dados mais básica do Redis. Quando você pensa num par chave-valor, você está pensando em strings. Não se deixe confundir pelo nome: como sempre, seu valor pode ser qualquer coisa. Eu prefiro chamá-los de "escalares", mas talvez seja só eu.

Nós já vimos um caso de uso comum para strings: guardar instâncias de objetos por chave. Isto é algo do qual você pode fazer uso pesado:

	set users:leto '{"name": "leto", "planet": "dune", "likes": ["spice"]}'

O Redis também lhe permite fazer algumas operações adicionais. Por exemplo, `strlen <chave>` pode ser usado para obter o comprimento do valor de uma chave; `getrange <key> <start> <end>` retorna a sequência especificada de valores; `append <key> <value>` adiciona um valor ao final de outro (ou cria um, se já não existir). Vá em frente e tente-os. Isso é o que eu obtenho:

	> strlen users:leto
	(integer) 50

	> getrange users:leto 31 48
	"\"likes\": [\"spice\"]"

	> append users:leto " OVER 9000!!"
	(integer) 62

Você deve estar pensando: isso é ótimo, mas não faz sentido. Você não pode adicionar uma string ou obter um range a partir de um JSON e isso funcionar. Você está certo - a lição aqui é que alguns comandos, especialmente os que lidam com strings, só fazem sentido com alguns tipos específicos de dados.

Mais cedo, aprendemos que o Redis não se importa com seus valores. Isso quase sempre é verdade. Entretanto, alguns comandos de strings são específicos a alguns tipos ou estruturas de valores. Como um exemplo vago, poderíamos ver os comandos `append` e `getrange` acima como úteis em alguma serialização customizada com uso de espaço eficiente. Estes aqui incrementam ou decrementam o valor de uma string:

	> incr stats:page:about
	(integer) 1
	> incr stats:page:about
	(integer) 2

	> incrby ratings:video:12333 5
	(integer) 5
	> incrby ratings:video:12333 3
	(integer) 8

Como você deve imaginar, strings do Redis são ótimas para analytics. Tente incrementar `users:leto` (um valor não-inteiro) e veja o que acontece (você deve receber um erro).

Um exemplo mais avançado são os comandos `setbit` e `getbit`. Há um [artigo excelente](http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps/) sobre como a Spool usa esses dois comandos para responder eficientemente à questão "quantos visitantes únicos tivemos hoje". Para 128 milhões de usuários, um laptop gera a resposta em menos de 50ms e usa apenas 16MB de memória.

Não importa que você entenda como os bitmaps funcionam ou como a Spool os utiliza, mas sim entender como as strings do Redis são mais poderosas do que parecem. Mesmo assim, os casos mais comuns são os que vimos acima: guardar objetos (complexos ou não) e contadores. E, como obter um valor por chave é muito rápido, strings são muito usadas para fazer cache de dados.

## Hashes

Hashes são um bom exemplo de porque não é muito preciso chamar o Redis de um sistema chave-valor. Veja, de várias formas, hashes são como strings. A diferença importante é que eles fornecem um nível extra de indireção: um campo. Assim, os comandos correspondentes a `set` e `get` em um hash são:

	hset users:goku powerlevel 9000
	hget users:goku powerlevel

Também podemos definir ou obter vários campos ao mesmo tempo, obter todos os campos e valores, listar todos os campos ou excluir um campo especificado:

	hmset users:goku race saiyan age 737
	hmget users:goku race powerlevel
	hgetall users:goku
	hkeys users:goku
	hdel users:goku age

Como você pode notar, hashes nos dão um pouco mais de controle que strings normais. Ao invés de guardar um usuário como um valor único serializado, poderíamos usar um hash para obter uma representação mais precisa. O benefício seria ter a habilidade de obter, atualizar ou excluir partes específicas dos dados, sem ter que ler ou escrever o valor inteiro.

Ver os hashes pela perspectiva de um objeto bem definido, como um usuário, é a chave para entender como eles funcionam. E é verdade que, por razões de performance, um controle mais granular pode ser útil. Todavia, no próximo capítulo vamos ver como os hashes podem ser usados para organizar seus dados e facilitar as consultas. Na minha opinião, é nisso que os hashes se destacam de verdade.

## Listas

Listas deixam você guardar e manipular um vetor de valores para uma determinada chave. Você pode adicionar valores à lista, pegar o primeiro ou o último valor e manipular valores em uma determinada posição. As listas mantêm a ordem inicial e tem operações baseadas em índices muito eficientes. Nós poderíamos ter uma lista `newusers` para acompanhar quais os usuários que se registraram por último em nosso site:

	lpush newusers goku
	ltrim newusers 0 49

Primeiro nós adicionamos um novo usuário ao início da lista, depois cortamos a lista de forma que ela contenha apenas os 50 últimos usuários. Este é um padrão comum. `ltrim` é uma operação O(N), onde N é o número de valores que estamos removendo. Neste caso, quando reduzimos a lista sempre após uma inserção, o `ltrim` terá na verdade um desempenho O(1) (porque N sempre será igual a 1).

Esta também é a primeira vez que estamos vendo um valor em uma chave referenciando um valor em outra. Se quiséssemos obter os detalhes dos últimos 10 usuários, faríamos a seguinte combinação:

    ids = redis.lrange('newusers', 0, 9)
    redis.mget(*ids.map {|u| "users:#{u}"})

O código acima é um exemplo em Ruby que nos mostra o tipo de consultas múltiplas das quais já falamos.

É claro, listas não são boas apenas para guardar referências para outras chaves. Os valores podem ser qualquer coisa. Você poderia usar listas para guardar logs ou seguir o caminho que um usuário está fazendo pelo site. Se você estivesse escrevendo um jogo, poderia usá-las para guardar a fila de ações do jogador.

## Conjuntos (Sets)

Conjuntos servem para guardar valores únicos, e fornecem uma série de operações baseadas em conjuntos, como a união. Conjuntos não possuem ordem, mas fornecem operações baseadas valor muito eficientes. O exemplo clássico do uso de conjuntos é uma lista de amigos:

	sadd friends:leto ghanima paul chani jessica
	sadd friends:duncan paul jessica alia

Independentemente de quantos amigos um usuário possua, nós podemos dizer eficientemente (O(1)) se o usuário X é ou não amigo do usuário Y.

	sismember friends:leto jessica
	sismember friends:leto vladimir

Além disso nós podemos ver se duas ou mais pessoas possuem amigos em comum:

	sinter friends:leto friends:duncan

E até guardar o resultado em uma nova chave:

	sinterstore friends:leto_duncan friends:leto friends:duncan

Conjuntos são ótimos para etiquetar ou acompanhar outras propriedades de um valor para o qual não faz sentido haver duplicatas (ou quando queremos aplicar operações de conjuntos como interseção e união).

## Conjuntos Ordenados (Sorted Sets)

A última e mais poderosa estrutura de dados é o conjunto ordenado. Se hashes são como strings com campos, então conjuntos ordenados são como conjuntos com pontuação. O score fornece as capacidades de ordenação e ranking. Se quiséssemos uma lista rankeada de amigos, poderíamos fazer:

	zadd friends:duncan 70 ghanima 95 paul 95 chani 75 jessica 1 vladimir

Quer descobrir quantos amigos `duncan` tem com um score de 90 ou mais?

	zcount friends:duncan 90 100

Que tal descobrir o rank de `chani`?

	zrevrank friends:duncan chani

Nós usamos `zrevrank` em vez de `zrank` pois a ordenação padrão do Redis é do menor para o maior valor, e neste caso queremos do maior para o menor. O caso de uso mais óbvio para conjuntos ordenados é um sistema de leaderboard. Na verdade, qualquer coisa que você queira ordenada por um inteiro, e que você queira manipular eficientemente baseado neste score, vai ser uma boa oportunidade para usar um conjunto ordenado.

## Neste Capítulo

Esta é uma visão geral das cinco estruturas de dados do Redis. Uma das coisas mais legais do Redis é que na maioria das vezes você pode fazer mais do que você percebe a princípio. Provavelmente, há formas de usar strings e conjuntos ordenados que ninguém pensou ainda. Contanto que você entenda o caso de uso normal, você vai achar o Redis ideal para todos os tipos de problemas. Além disso, não é porque o Redis expõe cinco estruturas de dados e vários métodos que você precisa usar todos eles. É comum implementar um recurso usando apenas uns poucos comandos.

# Capítulo 3 - Explorando as Estruturas de Dados

No capítulo anterior, falamos sobre as cinco estruturas de dados e demos alguns exemplos de quais tipos de problemas elas podem resolver. Agora é hora de ver alguns tópicos e padrões de projeto mais avançados, porém comuns.

## Notação do Grande-O

Através do livro nós fizemos referência à notação do Grande-O na forma O(n) ou O(1). A notação do Grande-O é usada para explicar como algo se comporta dado um certo número de elementos. No Redis, ela é usada para nos dizer o quão rápido é um comando, baseado no número de itens com os quais estamos lidando.

A documentação do Redis nos fala a notação do Grande-O para cada um de seus comandos. Ela também nos diz quais os fatores que influenciam o desempenho. Vamos olhar alguns exemplos.

O mais rápido possível é O(1), que é uma constante. Tanto faz se estamos lidando com 5 ou 5 milhões de itens, vamos ter o mesmo desempenho. O comando `sismember`, que nos diz se um valor pertence a um conjunto, é O(1). `sismember` é um comando poderoso, e as características de desempenho dele são um motivo forte para isso. Vários comandos do Redis são O(1).

A complexidade logarítmica, ou O(log(N)), é a segunda possibilidade mais rápida, porque ela varre seções cada vez menores. Usando esta abordagem de "dividir e conquistar", um grande número de itens é rapidamente quebrado em poucas iterações. `zadd` é um comando O(log(N)), onde N é o número de elementos que já pertencem ao conjunto ordenado.

Depois temos comandos lineares, ou O(N). Procurar por uma coluna não-indexada em uma tabela é uma operação O(N), assim como usar o comando `ltrim`. Entretanto, no caso do `ltrim`, N não é o número de elementos na lista, mas sim os elementos a serem removidos. Usar `ltrim` para remover 1 item de uma lista de milhões vai ser mais rápido que usar `ltrim` para remover 10 itens de uma lista de milhares (apesar que eles provavelmente vão ser tão rápidos que você nem conseguiria medir).

`zremrangebyscore`, que remove os elementos de um conjunto ordenado com um score entre certos valores mínimo e máximo, tem uma complexidade de O(log(N)+M). Isto faz dele um misto. Lendo a documentação, nós vemos que N é o número de elementos totais no conjunto e M é o número de elementos a serem removidos. Em outras palavras, o número de elementos que serão removidos provavelmente vai ser mais significativo, em termos de desempenho, que o número total de elementos no conjunto.

O comando `sort`, que nós vamos discutir mais detalhadamente no próximo capítulo, tem uma complexidade de O(N+M*log(M)). Das características de desempenho dele, você provavelmente já pode inferir que este é um dos comandos mais complexos do Redis.

Há várias outras complexidades. As duas mais comuns que restam são O(N^2) e O(C^N). Quanto maior for N, pior o desempenho deles relativo a um N pequeno. Nenhum dos comandos do Redis tem esse tipo de complexidade.

Vale lembrar que a notação do Grande-O trata do pior caso. Quando dizemos que algo leva O(N), na verdade podemos encontrar o item logo de cara ou ele pode ser o último elemento possível.


## Consultas Pseudo-Multi-Chave

Uma situação comum com a qual você vai se deparar é querer consultar o mesmo valor em chaves diferentes. Por exemplo, você pode querer obter um usuário pelo email (quando ele loga pela primeira vez) e também por id (depois que ele logou). Uma solução terrível é duplicar seu objeto usuário como duas strings:

	set users:leto@dune.gov '{"id": 9001, "email": "leto@dune.gov", ...}'
	set users:9001 '{"id": 9001, "email": "leto@dune.gov", ...}'

Isto é ruim porque é um pesadelo para gerenciar e porque ocupa o dobro da memória.

Seria legal se o Redis deixasse você apontar uma chave para outra, mas ele não deixa (e provavelmente nunca vai deixar). Uma das principais diretrizes do desenvolvimento do Redis é manter o código e a API simples e claras. A implementação interna para ponteiros de chaves (há muitas operações que podemos fazer com chaves que não discutimos ainda) não vale a pena quando você considera que o Redis já tem uma solução: hashes.

Usando um hash, nós eliminamos a necessidade de duplicação:

	set users:9001 '{"id": 9001, "email": "leto@dune.gov", ...}'
	hset users:lookup:email leto@dune.gov 9001

O que estamos fazendo é usar o campo como um pseudo-índice secundário e referenciando o objeto usuário único. Para obter um usuário por id, basta um `get` normal:

	get users:9001

Para obter o mesmo usuário por email, executamos um `hget` seguido por um `get` (em Ruby):

	id = redis.hget('users:lookup:email', 'leto@dune.gov')
	user = redis.get("users:#{id}")

Isto é algo que você vai acabar fazendo bastante. Para mim, é aqui que os hashes realmente brilham, mas não é um caso de uso óbvio até que você o veja pela primeira vez.

## Referências e Índices

Nós já vimos alguns exemplos de um valor apontando para outro. Vimos isso no nosso exemplo de listas, e vimos também na seção acima quando usamos hashes para facilitar um pouco as consultas. Essencialmente, isto se resume a ter que gerenciar manualmente os seus índices e referências entre valores. Sendo franco, eu acho que isso pode ser um pouco desestimulante, principalmente quando você pensa em ter que manter, atualizar e remover manualmente todas essas referências. Não há solução mágica para resolver esse problema no Redis.

Nós já vimos como os conjuntos são usados muitas vezes para resolver este tipo de índice manual:

	sadd friends:leto ghanima paul chani jessica

Cada membro deste conjunto é uma referência a um valor string do Redis contendo detalhes do usuário atual. E se `chani` mudar de nome, ou excluir a conta dela? Talvez faça sentido também acompanhar a relação inversa:

	sadd friends_of:chani leto paul

Ignorando os custos de manutenção, se você se parecer só um pouquinho comigo, vai se contorcer só de pensar no custo de memória e processamento de ter esses valores extra indexados. Na próxima seção, vamos discutir formas de reduzir o custo de performance de ter que fazer estes passos extras (falamos disso rapidamente no primeiro capítulo).

Mas se você parar para pensar, bancos de dados relacionais tem o mesmo trabalho extra. Índices ocupam memória, precisam ser varridos (ou idealmente lidos direto a partir da posição correta) e depois os registros correspondentes precisam ser localizados. O trabalho extra é abstraído elegantemente (e eles fazem várias otimizações de processamento para que isso seja muito eficiente).

Novamente, ter que lidar manualmente com referências no Redis é desagradável. Mas quaisquer preocupações iniciais que você tenha sobre o desempenho ou as implicações de memória devem ser testadas. Você vai descobrir que não há problema.

## Round Trips e Pipelines

Nós já mencionamos que consultas frequentes ao servidor é um padrão comum no Redis. Já que é algo que você vai fazer sempre, é bom dar uma olhada em alguns dos recursos que podemos usar para aproveitar isso ao máximo.

Primeiro, vários comandos aceitam um ou mais parâmetros, ou têm um comando irmão que aceita vários parâmetros. Nós vimos o `mget` mais cedo, que recebe múltiplas chaves e retorna os valores:

	ids = redis.lrange('newusers', 0, 9)
	redis.mget(*ids.map {|u| "users:#{u}"})

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

## Transações

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

## Anti-padrões de Chaves

No próximo capítulo, vamos falar de comandos que não estão relacionados a nenhuma estrutura de dados específica. Alguns destes são ferramentas administrativas e de depuração. Mas há um em particular do qual eu gostaria de falar: o comando `keys`. Este comando recebe um padrão e encontra todas as chaves que se encaixem nele. Este comando parece ser útil para várias tarefas, mas nunca deve ser usado em código de produção. Porque? Porque ele faz uma varredura linear por todas as chaves, vendo quais se encaixam. Ou, simplesmente, é lento.

Como as pessoas usam-no? Digamos que você está escrevendo um sistema de acompanhamento de bugs. Cada conta vai ter um `id` e você pode decidir guardar cada bug em uma string com uma chave do tipo `bug:account_id:bug_id`. Se você algum dia precisar encontrar todos os bugs de uma conta (para exibi-los, ou talvez removê-los se alguém cancelar uma conta), você pode ficar tentado (como eu fiquei!) a usar o comando `keys`:

	keys bug:1233:*

A melhor solução é usar um hash. De forma bem parecida a como usamos hashes para prover uma forma de expor índices secundários, também podemos usá-los para organizar nossos dados:

	hset bugs:1233 1 '{"id":1, "account": 1233, "subject": "..."}'
	hset bugs:1233 2 '{"id":2, "account": 1233, "subject": "..."}'

Para obter todos os ids de bugs de uma conta, nós simplesmente rodamos `hkeys bugs:1233`. Para excluir um bug específico nós podemos fazer `hdel bugs:1233 2`, e para excluir uma conta nós podemos apagar a chave usando `del bugs:1233`.


## Neste Capítulo

Esperamos que este capítulo, junto com o anterior, tenha lhe dado uma boa visão sobre como usar o Redis para implementar recursos reais. Há vários outros padrões que você pode usar para construir todo tipo de coisa, mas o segredo é entender as estruturas de dados fundamentais e como elas podem ser usadas para atingir coisas além da sua perspectiva inicial.


# Capítulo 4 - Além das Estruturas de Dados

Enquanto as cinco estruturas de dados formam o alicerce do Redis, há outros comandos que não são específicos de nenhuma estrutura de dados. Já vimos alguns destes: `info`, `select`, `flushdb`, `multi`, `exec`, `discard`, `watch` e `keys`. Este capítulo vai tratar de alguns outros comandos importantes.

## Expiração

O Redis permite que você marque uma chave para expiração. Você pode fornecer uma hora absoluta no formato de timestamp Unix (segundos desde 1º de janeiro de 1970) ou um tempo de vida em segundos. Este comando se baseia em chaves, então não importa que tipo de estrutura de dados a chave representa.

	expire pages:about 30
	expireat pages:about 1356933600

O primeiro comando vai apagar a chave (e os valores associados a ela) após 30 segundos. O segundo vai fazer o mesmo no dia 31 de dezembro de 2012 ao meio-dia.

Isto faz do Redis um mecanismo de cache ideal. Você pode descobrir quanto tempo de vida um item possui através do comando `ttl`, e pode remover a expiração de uma chave usando o `persist`:

	ttl pages:about
	persist pages:about

Por fim, há um comando especial de strings – `setex` – que lhe permite definir uma string e especificar um tempo de vida em um só comando atômico (mais por conveniência mesmo):

	setex pages:about 30 '<h1>about us</h1>....'

## Publicação e Assinaturas

As listas do Redis possuem os comandos `blpop` e `brpop`, que retornam e removem o primeiro (ou último) elemento da lista, ou bloqueiam enquanto não houver um disponível. Eles podem ser usados para implementar uma fila simples.

Além disso, o Redis possui suporte de primeira classe a publicação de mensagens e assinatura de canais. Você pode testar isso abrindo uma segunda janela com o `redis-cli`. Na primeira janela, assine um canal (vamos chamá-lo de `warnings`):

	subscribe warnings

A resposta é a informação da sua assinatura. Agora, na outra janela, publique uma mensagem no canal `warnings`:

	publish warnings "it's over 9000!"

Se você voltar para a primeira janela, você deve ter recebido a mensagem no canal `warnings`.

Você pode assinar vários canais (`subscribe channel1 channel2 ...`), assinar um padrão de canais (`psubscribe warnings:*`) e usar os comandos `unsubscribe` e `punsubscribe` para cancelar a assinatura de um ou mais canais, ou de um padrão de canais.

Finalmente, note que o comando `publish` retornou o valor 1. Isto indica o número de clientes que receberam a mensagem.


## Monitor e Slow Log

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

## Ordenação

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
	hset bug:12339 details '{"id": 12339, ....}'

	hset bug:1382 severity 2
	hset bug:1382 priority 2
	hset bug:1382 details '{"id": 1382, ....}'

	hset bug:338 severity 5
	hset bug:338 priority 3
	hset bug:338 details '{"id": 338, ....}'

	hset bug:9338 severity 4
	hset bug:9338 priority 2
	hset bug:9338 details '{"id": 9338, ....}'

Assim, não apenas fica tudo mais organizado – e nós podemos ordenar por gravidade ou prioridade – como também podemos dizer qual campo o `sort` deve retornar:

	sort watch:leto by bug:*->priority get bug:*->details

Ocorre a mesma substituição de valor, mas o Redis também reconhece o operador `->` e usa-o para procurar o campo especificado no nosso hash. Nós também incluímos o parâmetro `get`, que faz a substituição e a busca de campos da mesma forma, para trazer os detalhes do bug.

O `sort` pode ser lento em sets grandes. A boa notícia é que a saída de um `sort` pode ser guardada:

	sort watch:leto by bug:*->priority get bug:*->details store watch_by_priority:leto

Combinar a capacidade de guardar os resultados do `sort` com os comandos de expiração que já vimos antes resulta num ótimo combo.

## Scan

No capítulo anterior, vimos como o comando `keys`, apesar de útil, não devia ser usado em produção. Redis 2.8 introduziu um novo comando, o `scan`, que é seguro de usar em produção. Apesar de `scan` ter uma função semelhante a `keys`, existem algumas diferenças. Para ser franco, a maioria das *diferenças* vão parecer idiossincrasias, mas este é o preço de se ter um comando útil.

A primeira diferença é que uma única chamada a `scan` não trás necessáriamente todos os resultados. Não há nada estranho em resultados paginados; entretanto, `scan` retorna uma número variabel de resultados que não podem ser controlados precisamente. Você pode passar um `count` (sugestão), que é por padrão 10, mas é possivel receber mais ou menos do que o especificado em `count`.

Ao invés de implementar paginação através de `limit` e `offset`, `scan` usa `cursor`. A primeira vez que você chama `scan` você passa `0` como o cursor. Vemos abaixo uma chamada inicial a `scan` com um padrão (opcional) e um contador (opcional):

    scan 0 match bugs:* count 20

Como parte da resposta, `scan` retorna o próximo cursor a ser usado. Alternativamente, scan retorna `0` para dizer que chegou ao fim dos resultados. Perceba que o próximo valor do cursor não corresponde ao número de resultado ou nada que o cliente possa considerar útil.

Um uso comum pode ser assim:

    scan 0 match bugs:* count 2
    > 1) "3"
    > 2) 1) "bugs:125"
    scan 3 match bugs:* count 2
    > 1) "0"
    > 2) 1) "bugs:124"
    >    2) "bugs:123"

Nossa primeira chamada retorna o próximo cursor (3) e um resultado. Nossa chamada seguinte, usando o próximo cursor, retorna o cursor (0) e os últimos dois resultados. O que está acima é um uso *comum*. Já que `count` é só uma sugestão, é possível que `scan` retorna o próximo `cursor` (não 0) com nenhum resultado. Em outras palavras, um conjunto vazio de resultados não significa que não existem mais resultados. Somente um cursor 0 significa que não existem mais resultados.

Vendo por um lado bom, `scan` não tem estado pelo ponto de vista do Redis. Então não há necessidade de encerrar um cursor e não há mal em não ler um resultado completamente. Se você quiser, você pode parar de iterar os resultados, mesmo se o Redis retornou um cursor válido.

Existem mais duas coisas para se manter em mente. Primeira, `scan` pode retornar várias vezes a mesma chave. Depende de você lidar com isso (geralmente mantendo um conjunto de valores já vistos). Segunda, `scan` somente garante que os valores estavam presentes durante a duração da iteração vão ser retornados. Se valores forem adicionados ou removidos enquantos você está iterando, eles podem ou não serem retornados. Novamente, isso ocorre por causa da falta de estado do `scan`; ele não tira um snapshoot dos valores existentes (como você pode observar em bancos de dados que oferecem fortes garantias de consistência), mas sim itera sobre o mesmo espaço de memória que pode ou não ser modificado.

Além de `scan`, também foram adicionados os comandos `hscan`, `sscan` e `zscan`. Esses comandos permitem que você itere através de hashes, conjuntos e conjuntos ordenados. Porque precisamos deles? Bem, por que como `keys` bloqueia todas as outras chamadas, também o faz o comando `hgetall` e o comando `smembers`. Se você quer iterar sobre hash ou conjunto muito grande, você deve considerar usar esses novos comandos. `zscan` pode parecer menos útil já que paginar sobre um conjunto ordenado através de `zrangebyscore` ou `zrangebyrank` já é possível. Entretanto, se você quer iterar completamente através de um grande conjunto ordenado, `zscan` mostra seu valor.

## Neste Capítulo

Este capítulo focou nos comandos que não são específicos às estruturas de dados. Como todo o resto, o uso deles depende da situação. É normal construir uma aplicação ou recurso que não usa expiração, publicação/assinatura nem ordenação; mas é bom saber que eles estão ali. Além disso, nós tocamos em apenas alguns comandos. Há mais, e, uma vez que você tenha digerido o material deste livro, vale a pena olhar a [lista completa](http://redis.io/commands).

# Chapter 5 - Lua Scripting

Redis 2.6 inclui um interpretador Lua embutido que desenvolvedores podem usar para escrever queries mais avançandas para serem executados no Redis. Não seria um equivoco pensar nessa capacidade como um stored procedure presente na maioria dos banco de dados relacionais.

O aspécto mais dificil dessa funcionalidade é aprender Lua. Fortunadamento, Lua é parecida com a maioria das linguagens de propósitos gerais, é bem documentada, tem um comunidade ativa e é útil para outras coisas além de fazer scripts para Redis. Este capítulo não vai falar sobre Lua em detalhe; mas os poucos exemplos que vamos ver devem servir como uma simples introdução.

## Porque?

Antes de ver como usar scripts em Lua, você deve estar imaginando porque você iria querer fazer isso. Muitos desenvolvedores não gostam dos stored procedures tradicionais, existem alguma diferença em relação a eles? A resposta curta é não. Usado de maneira errada, scripts em Lua podem resultar em código mais difícil de testar, lógica de negócios fortemente acoplada a acesso de dados ou até lógica duplicada.

Se usada corretamente, entretanto, é uma funcionalidade que pode simplificar o código e melhorar performance. Ambos benefícios são alcançados ao agrupar múltiplos comandos, junto com uma lógica simples, em uma função coesa. O código fica mais simples por que cada chamada de um script Lua é executada sem interrupção e portanto fornece uma maneira clara de criar seus próprios comandos atômicos (essencialmente eliminando a necessidade do incomodo `watch`). Isso pode melhorar a performance ao remover a necessidade de retornar resultados intermediários - a saída final pode ser calculada dentro do script.

Os exemplos a seguir vão ilustrar melhor esses pontos.

## Eval

O comando `eval` recebe um script Lua (como uma string), as chaves que usaremos, e um conjunto opcional de arguementos arbitrários. Vamos ver um exemplo simples (executado por Ruby, já que executar comandos Redis multi-linhas a partir da ferramenta linha de comando não é bacana):

    script = <<-eos
      local friend_names = redis.call('smembers', KEYS[1])
      local friends = {}
      for i = 1, #friend_names do
        local friend_key = 'user:' .. friend_names[i]
        local gender = redis.call('hget', friend_key, 'gender')
        if gender == ARGV[1] then
          table.insert(friends, redis.call('hget', friend_key, 'details'))
        end
      end
      return friends
    eos
    Redis.new.eval(script, ['friends:leto'], ['m'])

O código acima obtem os detalhes de todos os amigos de Leto do sexo masculino. Perceba que para executar comandos Redis dentro do nosso script, nós usamos o método `redis.call("command", ARG1, ARG2, ...)`.

Se você é novo em Lua, você devia examinar cada linha com cuidado. Pode ser útil saber que `{}` cria uma `table` vazia (que pode agir como uma lista ou dicionário), `#TABLE` pega o número de elementos em TABLE, e `..` é usado para concatenar strings.

`eval` na verdade recebe 4 parametros. O segundo parâmetro deve na verdade ser o número de chaves; entretanto o driver Ruby automaticamente cria isso para nós. Porque ele é necessário? Considere que o código acima parece quando executado pela CLI:

    eval "....." "friends:leto" "m"
    vs
    eval "....." 1 "friends:leto" "m"

No primeiro (incorreto) caso, como o Redis sabe qual dos parâmetros são chaves e quais são argumentos arbitrários? No segundo caso, não há ambiguidade.

Isso nos leva a uma segunda questão: porque as chaves devem ser explicitamente listadas? Cada comando no Redis sabe, em tempo de execução, quais chaves vão ser necessárias. Isso permite que ferramentas futuras, como o Redis Cluster, distribuir requests entre múltiplos servidores Redis. Você pode ter percebido que nosso exemplo acima na verdade lê chaves dinamicamente (sem ter que passar elas para `eval`). Um `hget` é executado em todos os amigos de Leto do sexo masculino. Isso porque a necessidade de listar a chaves previamente é mais uma sugestão do que uma regra imposta. O código acima vai executara normalmente em uma configuração de uma única-instancia, ou mesmo com replicação, mas não vai funcionar no ainda não lançado Redis Cluster.

## Gerenciamento de Scripts

Mesmo que os scripts executados via `eval` sejam armazenados em cache pelo Redis, enviar o corpo cada vez que você quer executar algo não é o ideal. Ao invés, você pode registrar o script no Redis e executar sua chave. Para fazer isso você dever usar o comando `script load`, que retorna o SHA1 digest do script:

    redis = Redis.new
    script_key = redis.script(:load, "THE_SCRIPT")

Uma vez que nós carregamos o script, nós podemos usar `evalsha` para executa-lo:

    redis.evalsha(script_key, ['friends:leto'], ['m'])

Outros comandos que você pode usar para gerenciar seus scripts Lua são `script kill`, `script flush` e `script exists`. Eles são usados para matar um script em execução, remover todos os scripts do cache interno e verificar se um script já existe no cache.

## Bibliotecas

A implementação de Lua no Redis vem com um punhado de bibliotecas úteis. Enquando `table.lib`, `string.lib` e `math.lib` são bem úteis, para mim, vale a pena destacar a `cjson.lib`. Primeiramente, se você tem que passar múltiplos argumentos para um script, pode ser mais claro passa-los como JSON:

    redis.evalsha ".....", [KEY1], [JSON.fast_generate({gender: 'm', ghola: true})]

Que pode ser desserializados dentro do script Lua:

    local arguments = cjson.decode(ARGV[1])

Claro, a biblioteca JSON pode ser usadas para "parsear" valores armazenados no próprios Redis. Nosso exemplo acima poderia ser escrito como:

      local friend_names = redis.call('smembers', KEYS[1])
      local friends = {}
      for i = 1, #friend_names do
        local friend_raw = redis.call('get', 'user:' .. friend_names[i])
        local friend_parsed = cjson.decode(friend_raw)
        if friend_parsed.gender == ARGV[1] then
          table.insert(friends, friend_raw)
        end
      end
      return friends

Ao invés de pegar o gênero de um campo específico de um hash, nós podemos pega-lo dos próprios dados do amigo. (Esta é uma solução mais lenta, e eu pessoalmente prefiro a original, mas isso mostra que é possível).

## Atomico

Já que Redis é single-threaded, você não deve se preocupar com o seu script Lua ser interrompido por outro comando Redis. One dos benefícios mais óbvios disso é que chaves com um TTL não vai expirar na metade da execução. Se uma chave está presente no início do script, ela vai estar presente em qualquer ponto do script - a não ser que você a remova.

## Administração

O próximo capítulo vai falar sobre a administração e configuração do Redis em mais detalhes. Por enquanto, basta saber que o `lua-time-limit` define por quanto tempo um script Lua é permitido executar antes de ser finalizado. O padrão são generoso 5 segundos. Considere abaixar esse limite.

## Neste Capítulo

Este capítulo apresentou a funcionalidade de executar scripts Lua no Redis. Como qualquer coisa, essa funcionalidade pode ser abusada. Entretanto, se usada com prudência para implementar seus próprios e objetivos comandos, ela não vai somente simplificar seu código, como também provavelmente melhorar sua performance. Scripts em Lua são como quase qualquer outro comando/funcionalidade do Redis: você faz um uso limitado dele inicialmente somente para perceber que está usando mais e mais a cada dia.

# Capítulo 6 - Administração

Nosso último capítulo é dedicado a alguns aspectos administrativos do Redis. De forma alguma isso é um guia abrangente de administração do Redis. No melhor dos casos, vamos responder algumas das dúvidas mais básicas que novos usuários do Redis provavelmente vão ter.

## Configuração

Quando você rodou o servidor do Redis pela primeira vez, ele lhe avisou que o arquivo `redis.conf` não pôde ser encontrado. Este arquivo pode ser usado para configurar vários aspetos do Redis. Um arquivo `redis.conf` bem documentado está disponível em cada versão do Redis. O arquivo de exemplo contém as opções padrão de configuração, sendo útil tanto para entender o que os ajustes fazem quanto para saber quais são os valores padrão de cada ajuste. Você pode encontrá-lo em <http://download.redis.io/redis-stable/redis.conf>.

Como o arquivo é bem documentado, nós não vamos analisar cada ajuste.

Além de configurar o Redis pelo arquivo `redis.conf`, podemos usar o comando `config set` para mudar valores individuais. Na verdade, nós já usamos o `config set` quando passamos a configuração `slowlog-log-slower-than` para 0.

Também existe o comando `config get`, que exibe o valor de uma configuração. Este comando suporta a busca por padrões. Assim, se queremos exibir tudo relacionado a logs, podemos fazer:

   config get *log*

## Authentication

O Redis pode ser configurado para exigir uma senha. Isto pode ser feito através da configuração `requirepass` (que pode ser informada tanto no `redis.conf` quando com o comando `config set`). Quando o `requirepass` possui um valor configurado (que é a senha que deve ser utilizada), os clientes vão precisar executar um comando `auth password`.

Uma vez que o cliente esteja autenticado, ele poderá executar qualquer comando em qualquer banco de dados. Isto inclui o `flushall`, que apaga todas as chaves de todos os bancos. Na configuração, você pode renomear os comandos para obter um pouco de segurança por obfuscação:

	rename-command CONFIG 5ec4db169f9d4dddacbfb0c26ea7e5ef
	rename-command FLUSHALL 1041285018a942a4922cbf76623b741e

Ou você pode desabilitar um comando, modificando o novo nome para uma string vazia.

## Limitações de Tamanho

Enquanto você está começando a usar o Redis, você pode se perguntar "quantas chaves eu posso guardar?" Você pode também pensar quantos campos um hash pode ter (especialmente quando você usa-os para organizar seus dados), ou quantos elementos as listas e sets podem ter. Por instância, o limite prático para todos estes é na casa das centenas de milhões.

## Replicação

O Redis suporta replicação, o que significa que enquanto você escreve em uma instância do Redis (o master, ou "mestre"), uma ou mais outras instâncias (os slaves, ou "escravos") são mantidas atualizadas pelo master. Para configurar um slave, você pode usar tanto a configuração `slaveof` quanto o comando `slaveof` (instâncias que rodam sem esta configuração são ou podem ser masters).

A replicação ajuda a proteger seus dados copiando-os para servidores diferentes. A replicação também pode ser usada para melhorar o desempenho, já que as leituras podem ser feitas a partir dos slaves. Eles podem responder com dados ligeiramente desatualizados, mas para a maioria das aplicações esta é uma troca que vale a pena.

Infelizmente, a replicação do Redis não provê ainda failover automático. Se o master morrer, um slave precisa ser promovido manualmente. Ferramentas tradicionais de alta disponibilidade que usem monitoração de heartbeats e scripts para automatizar a troca são atualmente dores-de-cabeça necessárias se você quer atingir algum grau de alta disponibilidade com o Redis.

## Backups

Fazer backup do Redis é uma simples questão de copiar o snapshot do Redis para onde quer que você queira (S3, FTP, ...). Por padrão, o Redis salva o snapshot em um arquivo chamado `dump.rdb`. Você pode simplesmente copiar este arquivo a qualquer momento com `scp`, `ftp` ou `cp` (ou qualquer outra coisa).

Não é incomum desabilitar tanto a gravação de snapshots quanto o arquivo append-only (aof) no master e deixar um slave tomar conta disso. Isso ajuda a reduzir a carga no master e lhe permite definir parâmetros de gravação mais agressivos no slave, sem ferir a responsividade geral do sistema.

## Escalabilidade e Cluster Redis

Replicação é a primeira ferramenta da qual um site em crescimento pode tirar vantagem. Alguns comandos são mais custosos que outros (`sort` por exemplo) e despachar a execução deles para um slave pode manter o sistema respondendo bem às consultas que chegam.

Além disso, escalar de verdade o Redis acaba chegando em distribuir suas chaves em várias instâncias do Redis (que podem rodar na mesma máquina - lembre-se: o Redis só possui uma thread). Até agora, isso é algo que é você quem vai precisar resolver (apesar que vários drivers de Redis provêem algoritmos de hash consistentes). Pensar seus dados em termos de distribuição horizontal é algo que não vamos cobrir neste livro. Também é algo que você provavelmente não vai ter que se preocupar por um bom tempo, mas é algo do qual você precisa estar ciente independentemente da solução que você use.

A boa notícia é que já existem esforços no sentido de um Cluster Redis. Isso não apenas vai oferecer escalabilidade horizontal – incluindo rebalanceamento – como também vai fornecer failover automático para alta disponibilidade.

Alta disponibilidade e escalabilidade são algo que pode ser atingido hoje, contanto que você esteja disposto a direcionar seu tempo e esforço para isto. Mais adiante, o Cluster Redis deve deixar as coisas bem mais fáceis.

## Neste Capítulo

Dado o número de projetos e sites que já usam Redis, não resta dúvida que o Redis está pronto para produção, e que já tem estado assim por um bom tempo. Entretanto, algumas das ferramentas - especialmente as de segurança e disponibilidade - ainda são muito "verdes". O Cluster Redis, que esperamos ver em breve, deve ajudar a resolver alguns dos desafios de administração atuais.

# Conclusão

De várias maneiras, o Redis representa uma simplificação na forma como lidamos com dados. Ele remove muito da complexidade e abstração disponíveis em outros sistemas. Em vários casos, isso faz do Redis a escolha errada. Em outros casos, é como se o Redis tivesse sido feito especificamente para os seus dados.

No fim das contas voltamos para algo que eu disse bem no início: o Redis é fácil de aprender. Há muitas tecnologias novas, e pode ser difícil descobrir em quais vale a pena investir o tempo de aprender. Quando você considera os reais benefícios que o Redis tem a oferecer com sua simplicidade, eu acredito sinceramente que ele é um dos melhores investimentos – em termos de aprendizado – que você e sua equipe podem fazer.
