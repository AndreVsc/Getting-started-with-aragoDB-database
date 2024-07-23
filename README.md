# ARANGODB GET-START

## Passo 1: Instalar o Docker no Windows.
  > A instação do [Docker](https://www.docker.com/) é necessária para a configuração do amniente. Caso sua maquina
  >
  > estiver com porblemas de baixa-lo separamos um [link alternativo](https://docs.docker.com/desktop/install/windows-install/).

## Passo 2: Baixar e instalar o Docker Desktop.
  > Pressione o botão de download e abra o aplicativo assim que possuivel executar. Assim comece a instação.

## Passo 3: Acesse a página de download do Docker Desktop.
  > Docker Desktop is a one-click-install application.
  >
  > Execute o instalador e siga as instruções na tela para concluir a instalação.
  >
  > Manter todos os tickets - Computador será reiniciado. - Entrar sem login.

## Passo 4: Configurar o Docker Desktop:
  > O Docker pode solicitar que você habilite a virtualização no BIOS se ainda não estiver habilitada. Siga as instruções fornecidas pelo Docker para habilitar a virtualização.
  > Certifique-se de que o Docker Desktop está em execução e que a interface mostra que o Docker está "Running".

# Passo 5: Abrir o PowerShell ou Docker terminal e executar:
  > docker pull arangodb
  > para puxar a imagem oficial do ArangoDB. site https://hub.docker.com/_/arangodb/

# Passo 6: Execute o comando para iniciar o contêiner do ArangoDB. 

> Neste exemplo, estamos configurando uma senha  para o usuário root:
  
~~~~ javascript
 unix> docker run -e ARANGO_RANDOM_ROOT_PASSWORD=$(PASS) -d --name $(INSTANCENAME) -p 8529:8529 arangodb
~~~~

> Para verificar:

~~~~ javascript
 CONTAINERID     IMAGE      COMMAND            CREATED              STATUS               PORTS                  NAMES
$(IDCONTAINER)  arangodb  "/entrypoin…"   About a minute ago   Up About a minute   0.0.0.0:8529->8529/tcp  $(INSTANCENAME)
~~~~

> Abrindo seu navegador no http://localhost:8529/ :

~~~~ javascript
  Username: $(ROOT)
  sua senha aparece quando se digita:
  Docker logs {três primeiros digitos do seu container Id}
  Senha : *****************
~~~~

# Passo 7 , 8 , 9 dentro da interface do Arango:
 
 > O próximo passo e criar um database e um usuário com username e senha para o próximo login.
 >
 > Editar as permissões clicando no usuário criado no passo anterior e setar para administrate e SALVAR.
 >
 > Fazer o Logout e entrar com seu o novo usuário.

# POREM ENTRETANTO TODA VIA EXISTE A OPÇÃO DE FAZER TUDO ISSO PELO TERMINAL

> A partir do passo 6 as coisas mudam um pouco.
>
> Você devera executar o arangosh para permitir a navegabilidade:

~~~~ javascript
docker exec -it $(INSTANCENAME) arangosh --server.endpoint tcp://127.0.0.1:8529 --server.database $(DATABESENAME) --server.username $(USERNAME) --server.password $(PASS)
~~~~

# Só para facilitar o passo 7 , 8 , 9 podem ser feitos dessa maneira:

~~~~ javascript
  db._createDatabase("$(DATABASENAME)")
  db._useDatabase("$(DATABASENAME)") // USAR O BANCO
~~~~ 

~~~~ javascript
  // Substitua "novousuario" e "senhanova" pelos valores desejados
  var users = require("@arangodb/users");
  var adminUser = "novousuario";
  var adminPassword = "senhanova";
~~~~

~~~~ javascript
  // Cria o usuário 
  users.save(adminUser, adminPassword);
~~~~

~~~~ javascript
  // Concede permissões administrativas
  users.grantDatabase(adminUser, "$(DATABASENAME)", "rw");
  users.grantCollection(adminUser, "$(DATABASENAME)", "rw", "all");
~~~~ 

  >No arangosh, digite exit ou pressione Ctrl + C para sair.
  >E faça o comando de executar o arangosh com as novas informações.

# Você pode verificar se o novo usuário tem as permissões adequadas com o seguinte comando:

~~~~ javascript
var users = require("@arangodb/users");
var user = users.document("novousuario");
print(user);
~~~~ 


> Agora para implementação no console relacionada a grafos prosseguimos com:

# passo 1:  módulo de grafos para criar um grafo e definir as coleções de vértices e arestas:

~~~~ javascript
    var graph_module = require("@arangodb/general-graph");

    if (!graph_module._exists("meuGrafo")) {
      var graph = graph_module._create(
        "meuGrafo",
        [
          graph_module._relation("arestas", ["vertices"], ["vertices"])
        ]
      );
    } else {
      var graph = graph_module._graph("meuGrafo");
    }

// Caso cause algum erro use 
  var graph_module = require("@arangodb/general-graph");
  graph_module._drop("meuGrafo", true);
~~~~

# Passo 2: Adicione vértices e arestas ao grafo criado:

~~~~ javascript
    db.vertices.save({ _key: "1", nome: "Alice" });
    db.vertices.save({ _key: "2", nome: "Bob" });
~~~~

  > Adicionar Arestas:

~~~~ javascript 
   db.arestas.save({ _from: "vertices/1", _to: "vertices/2", tipo: "amizade" });
~~~~ 

# Passo 3: Use consultas AQL para interagir com o grafo. Por exemplo, para obter todos os vizinhos de um vértice:

~~~~ javascript

    var query = `
      FOR v, e, p IN 1..1 ANY 'vertices/1' GRAPH 'meuGrafo'
      RETURN p
    `;
    db._query(query).toArray();

~~~~

> Pronto, Você agora tem um grafos simples.
>
>Explicação por etapa:

~~~~ javascript
db._createDatabase("meu_grafo"):  // Cria um novo banco de dados chamado "meu_grafo".
~~~~ 

~~~~ javascript
db._useDatabase("meu_grafo"): // Seleciona o banco de dados "meu_grafo" para uso.
~~~~ 

~~~~ javascript
db._create("vertices"): // Cria uma coleção de documentos chamada "vertices" para armazenar vértices.
~~~~ 

~~~~ javascript
db._create("arestas", { type: "edge" }): // Cria uma coleção de documentos chamada "arestas" do tipo "edge"
para armazenar arestas. O tipo "edge" indica que esta coleção é específica para arestas.
~~~~

~~~~ javascript
require("@arangodb/general-graph"): // Importa o módulo de grafos do ArangoDB.
~~~~

~~~~ javascript
graph_module._create("meuGrafo", [...]): // Cria um novo grafo chamado "meuGrafo" com uma relação
(arestas) conectando vértices na coleção "vertices".
~~~~

~~~~ javascript
graph_module._relation("arestas", ["vertices"], ["vertices"]): // Define uma relação chamada "arestas"
conectando vértices da coleção "vertices" para "vertices".
~~~~

~~~~ javascript
graph_module._exists("meuGrafo"): // Verifica se o grafo "meuGrafo" já existe.
~~~~ 

~~~~ javascript
db.vertices.save({ _key: "1", nome: "Alice" }): // Adiciona um vértice com a chave "1" e o nome "Alice"
à coleção "vertices".
~~~~ 

> _key: É um identificador único para o vértice.

~~~~ javascript
db.arestas.save({ _from: "vertices/1", _to: "vertices/2", tipo: "amizade" }): Adiciona uma aresta conectando o vértice "1" ao vértice "2" com o tipo "amizade".
~~~~ 

> _from: Especifica o vértice de origem.
>
> _to: Especifica o vértice de destino.
>
> FOR v, e, p IN 1..1 ANY 'vertices/1' GRAPH 'meuGrafo': Percorre o grafo "meuGrafo",
> começando do vértice "1" e retornando os caminhos encontrados.
> RETURN p: Retorna o caminho (p) encontrado na consulta.

# Horizontes para Modelagem de Grafos

> Para modelar grafos mais complexos, considere os seguintes conceitos:
>
  > Multigrafos: Grafos que permitem múltiplas arestas entre os mesmos pares de vértices.
  > 
  > Arestas Direcionadas e Não-Direcionadas: Determine se as relações têm uma direção ou não.
  > 
  > Propriedades em Vértices e Arestas: Adicione mais propriedades aos vértices e arestas para enriquecer o modelo.
  >  
  > Subgrafos: Defina subgrafos para representar diferentes domínios de interesse dentro do mesmo grafo.
>
>Adicionar Vértices e arestas com Propriedades Adicionais:

~~~~ javascript
db.vertices.save({ _key: "3", nome: "Charlie", idade: 25, cidade: "São Paulo" });

db.arestas.save({ _from: "vertices/1", _to: "vertices/3", tipo: "colega", desde: "2020" });
~~~~ 

> Consulta Avançada:

~~~~ javascript
  var query = `
    FOR v, e, p IN 1..1 OUTBOUND 'vertices/1' GRAPH 'meuGrafo'
    FILTER e.tipo == 'amizade'
    RETURN { amigo: v.nome, desde: e.desde }
  `;
  db._query(query).toArray();
~~~~ 

> OUTBOUND: Especifica a direção da aresta (saindo do vértice de origem).
> 
> FILTER e.tipo == 'amizade': Filtra as arestas para incluir apenas aquelas com o tipo "amizade".


# Exercício Didático: Configuração e Uso Básico do ArangoDB

> Objetivo:
> Criar um banco de dados gráfico simples.
> Criar um novo usuário com permissões administrativas.
> Realizar operações básicas no banco de dados usando o novo usuário.

