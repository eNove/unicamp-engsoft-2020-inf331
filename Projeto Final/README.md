# Ranqueamento por Avaliação de Fornecedores e Leilão invertido

# Equipe
* eNove

### Membros
* Andrew Siqueira Guedes
* Fernando de Morais
* Rafael Mardegan Marquini
* Rodolfo Dalla Costa
* Ronaldo de Moraes Galvão

# Nível 1

## Diagrama Geral do Nível 1

Apresente um diagrama conforme o modelo a seguir:

> ![Modelo de diagrama no nível 1](images/barramento_nivel1.png)

### Detalhamento da interação de componentes

1. O componente `Comprador` publica o tópico "`/leilao/{idUsuario}/produtoDesejado`" pela interface **ILeilao** que é subscrito pelo componente `Leilao`. Desta forma, há demonstração de interesse de um comprador por um produto;
2. O componente `Leilao` publica o tópico "`/leilao/{idLeilao}/buscaProduto`" pela interface **IProduto** que é subscrito pelo componente `Produto`. Assim, ocorre a interação Leilão - Produto pela interface **IProduto**;
3. O componente `Produto` publica o tópico pela interface **IProduto** "`/produto/{idLeilao}/{idProduto}`" que é subscrito pelo componente `Leilao`, novamente. Logo, um novo leilão poderá ser iniciado ou um em andamento será associado com este novo comprador;
4. O componente `Leilao` inicia o leilao publicando o tópico "`/leilao/{idLeilao}/inicio`" pela interface **ILeilao** que é subscrito pelo componente `Fornecedor`;
5. O componente `Fornecedor` publica o tópico "`/oferta/{idLeilao}/{idOferta}`" através da interface **IOferta** que é subscrito pelo componente `Leilao`, onde pelo _idLeilao_ aguarda distintas ofertas;
6. Por fim, o componente `Leilao` publica o tópico "`/oferta/{idLeilao}/{idOferta}/menorPreco`" através da interface **IOferta** que é subscrito pelo componente `Comprador`, que aguarda e irá receber o resultado do leilão.

## A seguir temos a descrição dos componentes:

### Componente `Comprador`

> Componente responsável pela iteração entre cliente e os leilões que ocorrem sobre o produto em que o cliente demonstra interesse. Recebe as atualizações dos lances realizados e comunica o cliente sobre os lances e os produtos disponíveis.

![Componente](images/barramento-c_Comprador.png)

**Interfaces**  
> * ILeilao
> * IOferta

### Componente `Produto`

> Componente responsável por avaliar a disponibilidade dos produtos que irão entrar em leilão. Devolve lista de fornecedores que comercializam o produto e código do produto requisitado para leilão.

![Componente](images/barramento-c_Produto.png)

**Interfaces**  
> * IProduto

### Componente `Fornecedor`

> Fornecedor é acionado pelo Componente leilão, que informa o código do produto que irá entrar em leilão, avalia a quantidade de produtos disponíveis para leilão e realiza o lance do produto desejado pelo comprador.

![Componente](images/barramento-c_Fornecedor.png)

**Interfaces**  
> * IOferta
> * ILeilao

### Componente `Leilao`

> O Componente Leilão é o orquestrador de todo o leilão que é relaizado. É o componente responsável por receber os interesses do componente Comprador, verifica a disponibilidade dos produtos, aciona a lista de fornecedores que comercializão o produto que irá para leilão. Recebe todos os lances realizados pelos fornecedores e encaminha para o comprador.

![Componente](images/barramento-c_Leilao.png)

**Interfaces**  
> * IProduto
> * IOferta
> * ILeilao

## A seguir temos as interfaces uitlizadas:

### Interface ILeilao

Interface realiza o envio de dados do produto desejado pelo cliente ao componente Leilão. 
Comunica os fornecedores que comercialização o produto desejado.

**Tópico**: `leilao/{id}/produtoDesejado`

![Diagrama Classes REST](images/diagrama-classes-rest-topico1.jpg)

~~~json
{
    "idUsuario": "098234",
    "produtoDesejado":"Geladeira",
}
~~~

Detalhamento da mensagem JSON:

**ILeilao**
Atributo | Descrição
-------- | --------
idUsuario | Identificação do usuário
produtoDesejado | Identificador do produto desejado pelo comprador


**Tópico**: `leilao/{idLeilao}/inicio`

![Diagrama Classes REST](images/diagrama-classes-rest-topico2.jpg)

~~~json
{
    "idLeilao": "000001",
    "fornecedores": [
      { "idFornecedore": "001", "nome": "Brastemp" },
      { "idFornecedore": "002", "nome": "Lojas 100" },
      { "idFornecedore": "003", "nome": "Eletro Norte" },
      { "idFornecedore": "004", "nome": "Cibelar" },
    ],
    "produto": {
        "idProduto": "0123",
        "nome": "Geladeira"
    },
    "oferta_menor_preco":{
      "valor": 00.00
    }
}
~~~

**ILeilao**
Atributo | Descrição
------- | --------
idLeilao | Identificação do leilão
fornecedores | Array com lista de fornecedores
idFornecedore | Identificação do fornecedor
nome | Nome do fornecedor
produto | Objeto de produto
idProduto  | Identificação do Produto
nome | Nome do Produto


### Interface IOferta

Interface que faz o envio de dados do "lance" ofertado pelo `Fornecedor` que é recebido e orquestrado pelo componente `Leilao`, que por sua vez disponibiliza a mensagem para o componente `Comprador`priorizando os menores valores "lance" ofertados.

**Tópicos**:
> * `oferta/{idLeilao}/{idOferta}/menorPreco` 
> * `oferta/{idLeilao}/{idOferta}`

![Diagrama Classes REST](images/classe_ioferta.png)

~~~json
{
  "idLeilao": "000001",
  "idOferta": "000001",
  "dtIniOferta": "2020-09-18T21:20:00Z",
  "dtFimOferta": "2020-09-20T21:20:00Z",
  "produtos": [
    {
      "idProduto": "0123",
      "nome": "Geladeira",
      "quantidade": 1,
      "vlrLance": 3999.99
    }
  ]     
}
~~~

Detalhamento da mensagem JSON:

**Oferta**
Atributo | Descrição
-------| --------
idLeilao | número do leilão que requisitou o produto
idOferta | número da oferta
dtIniOferta | data de requisicao
dtFimOferta | itens do pedido
produtos | itens ofertados

**Item**
Atributo | Descrição
-------| --------
idProduto | identificador do item
nome | nome do item
quantidade | quantidade do item
vlrLance | valor da oferta


### Interface IProduto

Interface para comunicação entre `Leilao` e `Produto`, utilizada pelo componente `Leilao` para início do processo de validação de disponibilidade de um produto e pelo componente `Produto` para retorno de detalhes e lista de fornecedores.

**Tópico**: `leilao/{idLeilao}/buscaProduto`

![Consulta de Produto](images/classe_iproduto_consulta.png)

~~~json
{
  "idLeilao": "000001",
  "produtoDesejado": "Geladeira",
}
~~~

Detalhamento da mensagem JSON:

**Consulta**  

Atributo | Descrição
-------| --------
idLeilao | número do leilão que requisitou o produto
produtoDesejado | palavras chave para localização do produto

**Tópico**: `produto/{idLeilao}/{idProduto}`

![Retorno de Produto](images/classe_iproduto_retorno.png)

~~~json
{
  "idLeilao": "000001",
  "produto": {
    "idProduto": "0123",
    "nome": "Geladeira"
  },
  "fornecedores": [
    { "idFornecedore": "001", "nome": "Brastemp" },
    { "idFornecedore": "002", "nome": "Lojas 100" },
    { "idFornecedore": "003", "nome": "Eletro Norte" },
    { "idFornecedore": "004", "nome": "Cibelar" },
  ]
}
~~~

Detalhamento da mensagem JSON:

**Retorno**
Atributo | Descrição
-------| --------
idLeilao | número do leilão que requisitou o produto
produto | detalhes do produto solicitado
fornecedores | listagem dos fornecedores do produto

**Produto**
Atributo | Descrição
-------| --------
idProduto | número único do produto
nome | nome do produto normalizado

**Fornecedor**
Atributo | Descrição
-------| --------
idFornecedor | número único do fornecedor
nome | nome do fornecedor cadastrado no sistema

# Nível 2

Apresente aqui o detalhamento do Nível 2 conforme detalhado na especificação com, no mínimo, as seguintes subseções:

## Diagrama do Nível 2

Apresente um diagrama conforme o modelo a seguir:

> ![Modelo de diagrama no nível 2](images/diagrama-subcomponentes.png)

### Detalhamento da interação de componentes

O detalhamento deve seguir um formato de acordo com o exemplo a seguir:

* O componente `Entrega Pedido Compra` assina no barramento mensagens de tópico "`pedido/+/entrega`" através da interface `Solicita Entrega`.
  * Ao receber uma mensagem de tópico "`pedido/+/entrega`", dispara o início da entrega de um conjunto de produtos.
* Os componentes `Solicita Estoque` e `Solicita Compra` se comunicam com componentes externos pelo barramento:
  * Para consultar o estoque, o componente `Solicita Estoque` publica no barramento uma mensagem de tópico "`produto/<id>/estoque/consulta`" através da interface `Consulta Estoque` e assina mensagens de tópico "`produto/<id>/estoque/status`" através da interface `Posição Estoque` que retorna a disponibilidade do produto.

## Diagrama UML 

> ![Diagrama de UML](images/diagrama_classes_nivel_2.jpg)

Para cada componente será apresentado um documento conforme o modelo a seguir:

## Componente `<Nome do Componente>`

> <Resumo do papel do componente e serviços que ele oferece.>

![Componente](images/diagrama-componente.png)

**Interfaces**
> * ILeilao (ja especificada)
> * ITemplate

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `ITemplate`
![Diagrama da Interface](images/diagrama-Itemplate.jpg)

> A Interface Itemplate reune um montante de informacoes que sao importantes para o fornecedor e que podem ser gerenciadas em sua maioria pelo proprio model do fornecedor. Utilizada para se comunicar para com a view.

Detalhamento do json:
~~~json
{
  "MenorOferta": 10.10,
  "SuaUltimaOferta": 09.09,
  "MelhorOfertaEhSua": false,
  "TempoDesdeUltimaOferta": "00:05:13",
  "produto":{
    "idProduto" : "0001",
    "nome": "Geladeira",
    "qtdEstoque": 3,
    "imgProduto": "example.com/<idLocalizacao>"
  }
}
~~~

Método | Objetivo
-------| --------
`MenorOferta` | `Valor da menor oferta em leilao`
`SuaUltimaOferta` | `Valor da ultima oferta feita pelo fornecedor em leilao (caso nao tenha feito nenhuma o valor eh negativo`
`MelhorOfertaEhSua` | `Indicador que determina se a melhor oferta do leilao pertence ao fornecedor`
`TempoDesdeUltimaOferta` | `Tempo decorrido desde seu ultimo lance`
`produto` | `Informacoes sobre o Produto como Id, nome entre outras.`


# Multiplas Interfaces

> Escreva um texto detalhando como seus componentes  podem ser preparados para que seja possível trocar de interface apenas trocando o componente View e mantendo o Model e Controller.
>
> É recomendado a inserção de, pelo menos, um diagrama que deve ser descrito no texto. O formato do diagrama é livre e deve ilustrar a arquitetura proposta.
