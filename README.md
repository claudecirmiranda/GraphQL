<h1>API GraphQL</h1>

GraphQL é uma **Declarative Query Language**, uma especificação para **interações client/server**, independente de **data store** e de **plataforma**.

Desenvolvido em 2012 pela equipe do Facebook e Open Source desde Julho de 2015.

Utilizado principalmente para melhorar a performance e experiência de uso de aplicações Mobile.

O GraphQL tem três principais características:

* *Ele permite que o cliente especifique exatamente os dados que ele quer;*
* *Torna mais fácil agregar dados de várias fontes;*
* *Usa um sistema de tipos para descrever os dados.*

Com o GraphQL, o usuário pode fazer uma única chamada para buscar todas as informações que precisa. Diferente do que acontece na arquitetura REST, que é necessário fazer várias requisições para buscar as mesmas informações.

Uma consulta usando GraphQL é uma string enviada e interpretada pelo servidor, que após processar retorna o JSON no formato solicitado.

<h3>Princípios Básicos</h3>

O GraphQL, de modo geral, possui três estruturas principais:

* *as queries;*
*  *as mutations;*
* *e as subscriptions.*

Tomando como exemplo a seguinte estrutura:

|Cachorro|Pessoa|
| ---  | ---  |
|Id: Int|Id: Int|
|Nome: String|Nome: String|
|Raça: String|Idade: Int|
|Dono: Pessoa|Cachorro: Cachorro|

Para expormos estas estruturas através de uma API GraphQL, precisamos descrevê-la através de types (nesse caso específico, teríamos object types). Os types, na verdade, servem para representar recursos em geral em uma API GraphQL.

Para nosso exemplo, os types seriam definodos da seguinte forma:

<h4> Example Type</h4>

	type Pessoa {
		  id: ID
		  nome: String!
		  idade: Integer
		  cachorros: \[Cachorro\]
	}

	type Cachorro {
	  id: ID
	  nome: String!
	  raca: String
	  dono: Pessoa!
	}

Como uma pessoa pode ter mais de um cachorro, veja que informamos que o atributo “cachorros” é um conjunto do type Cachorro, já que o colocamos entre colchetes. O mesmo não acontece no atributo “dono” do type Cachorro, já que um cachorro só pode pertencer a um único dono por vez.

As exclamações na frente dos atributos dos types indicam que aqueles atributos em questão são `obrigatórios` e, por decorrência, não podem possuir valores nulos. Não colocamos a exclamação na frente do atributo “cachorros” do type Pessoa, porque nem toda pessoa possui um ou mais cachorros. Colocamos a exclamação para o atributo dono do type Pessoa, pois todo cachorro deve possuir um dono em questão.

Os ids estão definidos com o tipo `ID` do GraphQL. Com isso, nós informamos que aquele atributo é um identificador único para cada um dos recursos que são expostos pela API.

Algo que seja definido como um ID, necessariamente não precisa ser um número. Pode ser um GUID, por exemplo, desde que represente um identificador único.

Temos um schema com a definição dos types Pessoa e Cachorro, types estes que possuem `fields` – o equivalente aos `atributos`. Um schema define a estrutura, entre outras coisas, dos `object types`. Os types Pessoa e Cachorro são object types.

<h4>Utilização do GraphQL</h4>

[![GraphQL Utilization](http://soaone.com.br/graphql/graphqlutilization.png "GraphQL Utilization")](http://soaone.com.br/graphql/graphqlutilization.png "GraphQL Utilization")

A implementação da API GraphQL é realizada No GraphQL-Server. Em tempo de execução, o GraphQL-Server tem as seguintes responsabilidades:

* **Validation**: determinar se a Query ou Mutation é válida ou não.
* **Resolver**: especificar onde os dados serão obtidos.
* **Producing the result**: tratar cada atributo de forma a produzir um payload que espelha a consulta (query) que foi realizada.

<h4>Nota</h3>

* O **request** sempre será feito na mesma URL e utilizando o mesmo Método HTTP (**POST**), independentemente de qual dado esteja sendo requisitado ou manipulado. Exemplo: POST: http://api.grupomult.com/graphql
* O response sempre será um “**HTTP 200: OK**”, independentemente do resultado da requisição ter sido com sucesso ou com erro

<h4>Querys no GraphQL</h4>

As **queries** são utilizadas quando desejamos recuperar recursos do servidor GraphQL. Se trata de uma operação de consulta. Além disso, as queries são como types especiais para o GraphQL, já que elas definem **operações**.

Em GraphQL, é possível realizar **Queries** (consulta de dados) e **Mutations** (mutações / alterações de dados).

As **Queries** são análogo a operações **GET** do Restful.

Se quiséssemos que nossa API GraphQL retorne uma consulta dos donos e seus respectivos cachorros através do id do dono, teríamos algo similar ao trecho de schema abaixo:

<h5>Query Exemple</h5>

	type Query {
	  dono(id: String): Pessoa
	}

Dessa forma estamos definindo uma consulta que estamos chamando de dono dentro das queries expostas pela nossa API GraphQL.

Entenda isso como se fosse um possível endpoint de nossa API.

Podemos receber um parâmetro em nossa consulta: o id do dono a ser pesquisado. Por fim, essa consulta promete nos retornar um objeto do tipo Pessoa.

Se fôssemos invocar uma consulta para nossa API querendo recuperar as informações do dono e seus respectivos cachorros com o id 1, poderíamos ter a consulta abaixo:

<h4>Example Request</h4>
	query DonoPorId($id: String){
		  dono(id: $id){
			    id
			    nome
			    idade
			    cachorros {
			      id
			      nome
			      raca
			    }
		  }
	}
	// Variáveis
	{
		  "id": 1
	}

Criamos uma consulta nomeada chamada **DonoPorId**, que recebe um parâmetro chamado **id**. Sabe-se que **id** é um parâmetro por causa do **$** à frente dele – o $ é indicador de variável para o GraphQL. Internamente, essa consulta nomeada chamará a query **dono** em nossa API, repassando o valor da variável id como valor para o parâmetro id que a query espera em nosso serviço. A consulta é nomeada para que seu significado fique mais claro no cliente – *considere-se como uma boa prática quando estamos lidando com o GraphQL*.

O intuito dessa consulta é  recuperar o dono cujo id é 1, especificando os campos que desejamos como resposta: o id, o nome, a idade e os `cachorros` do dono com o id repassado. Ainda especificamos as informações dos cachorros que desejamos: o id, o nome e a raça.

Como resposta teríamos:

<h4>Response Example</h4>

	{
	  "data": {
		    "dono": {
			      "id": 1,
			      "nome": "João"
			      "idade": 20,
			      "cachorros": [{
				        "id": 1,
				        "nome": "Manu",
				        "raca": "Poodle"
				      }]
			    }
		  }
	}

<h3>Querys Models</h3>

<h4>Declarative query language</h3>
O client declara os dados que precisa e a resposta é um espelho do que foi solicitado.

[![GraphQL Utilization](http://soaone.com.br/graphql/declarative.png "Query Models")](http://soaone.com.br/graphql/declarative.png "GraphQL Utilization")

<h4>Permite múltiplas queries na mesma requisição</h4>
O GraphQL-Server executa as queries em paralelo para optimizar a requisição.
O client declara os dados que precisa e a resposta é um espelho do que foi solicitado.

[![GraphQL Utilization](http://soaone.com.br/graphql/multiplequery.png "Query Models")](http://soaone.com.br/graphql/multiplequery.png "GraphQL Utilization")

<h4>Merge da mesma instância de objeto</h4>
[![GraphQL Utilization](http://soaone.com.br/graphql/merge.png "Query Models")](http://soaone.com.br/graphql/merge.png "GraphQL Utilization")

<h4>Alias</h4>
Evita conflitos entre intâncias diferentes do mesmo objeto.

[![GraphQL Utilization](http://soaone.com.br/graphql/alias.png "Query Models")](http://soaone.com.br/graphql/alias.png "GraphQL Utilization")

<h4>Objetos aninhados</h4>
[![GraphQL Utilization](http://soaone.com.br/graphql/objetosaninhados.png "Query Models")](http://soaone.com.br/graphql/objetosaninhados.png "GraphQL Utilization")

<h4>Referências cíclicas</h4>
[![GraphQL Utilization](http://soaone.com.br/graphql/cyclicreference.png "Query Models")](http://soaone.com.br/graphql/cyclicreference.png "GraphQL Utilization")

<h4>Argumentos</h4>
Cada Atributo e objeto aniuinhado pode ter seu prórpio conjunto de argumentos.

[![GraphQL Utilization](http://soaone.com.br/graphql/arguments.png "Query Models")](http://soaone.com.br/graphql/arguments.png "GraphQL Utilization")

[![GraphQL Utilization](http://soaone.com.br/graphql/arguments2.png "Query Models")](http://soaone.com.br/graphql/arguments2.png "GraphQL Utilization")

<h4>Paginação</h4>
[![GraphQL Utilization](http://soaone.com.br/graphql/pagination.png "Query Models")](http://soaone.com.br/graphql/pagination.png "GraphQL Utilization")

<h4>Variáveis e Diretivas</h4>
[![GraphQL Utilization](http://soaone.com.br/graphql/varanddirectives.png "Query Models")](http://soaone.com.br/graphql/varanddirectives.png "GraphQL Utilization")

<h4>Fragmentos</h4>
[![GraphQL Utilization](http://soaone.com.br/graphql/varanddirectives.png "Query Models")](http://soaone.com.br/graphql/varanddirectives.png "GraphQL Utilization")

<h4>União e Polimorfismo</h4>

Situações em que não se sabe o tipo de retorno, precisa de alguma maneira determinar como lidar com esses dados no cliente.

[![GraphQL Utilization](http://soaone.com.br/graphql/polymorphism.png "Query Models")](http://soaone.com.br/graphql/polymorphism.png "GraphQL Utilization")

<h4>Mutações</h4>

Análogo as operações **POST / PUT / DELETE** do Restful.

Enquanto que as Queries são executados em paralelo, as mutação são executados em série, uma após a outra. Isto significa que, se enviarmos duas mutações de “CadastroCliente”, o primeiro irá terminar antes que o segundo inicie.

Caso a operação de mutação retorne um objeto, você pode solicitar atributos específicos e objetos aninhados.

[![GraphQL Utilization](http://soaone.com.br/graphql/mutation.png "Query Models")](http://soaone.com.br/graphql/mutation.png "GraphQL Utilization")

<h4>Validações</h4>

[![GraphQL Utilization](http://soaone.com.br/graphql/validation.png "Query Models")](http://soaone.com.br/graphql/validation.png "GraphQL Utilization")

<h3>GrapphQL Resume<h3>

* Único endpoint
* Query Language para API
* Pedir o que precisa e obter exatamente isso
* Obter muitos recursos em um único pedido
* Descrever o que é possível com um sistema de tipagem
* Evoluir a API sem necessidade de versionamento
* Utilizar as próprias bases de dados e código

<h3>Pontos de Atenção<h3>

As operações não são claras como em REST (GET / POST / PUT / DELETE)
Todo retorno é “HTTP `200` OK”
Mais poder para os developers de APPs
Não facilita o lado de prover as APIs
Efeito colateral de utilizar mais dados de entrada na chamada e uso do servidor (processamento)
Não resolve sozinho os problemas (esforço) de design
Dificuldade em fazer troubleshooting de problemas e dar suporte aos desenvolvedores de APPs

<h3>Ecossistema GraphQL</h4>

<h4>GraphQL Clients</h4>
* JavaScript – Relay, Apollo Client, Lokka
* Swift / iOS – Apollo iOS
* Java / Android – Apollo Android
* C# / .NET – graphql-net-client

<h4>Server Libraries</h4>

* JavaScript – GraphQL.js, express-graphql, graphql-server
* Ruby – graphql-ruby
* Python – Graphene
* Scala – Sangria
* Java – graphql-java
* Clojure – alumbra, graphql-clj, lacinia
* Go – graphql-go, graphql-relay-go, neelance/graphql-go
* PHP – graphql-php, graphql-relay-php
* C# / .NET – graphql-dotnet, graphql-net
* Elixir – absinthe, graphql-elixir

<h3>IDE para Documentar e Explorar APIs GraphQL<h3>

* GraphiQL – https://github.com/graphql/graphiql

