# Backend Challenge

Bem-vindx ao desafio técnico para a vaga de backend na Delivery Much! 🍽

O mercadinho de Seu Zé, como diversos pequenos empreendimentos do Brasil, sofreu com a redução do volume de vendas no meio físico durante a pandemia de COVID-19. Buscando encontrar soluções para seu problema, viu na Delivery Much uma possível saída para sustentar seu negócio: levá-lo para o meio digital, atraindo clientes de diferentes bairros de sua cidade em Alegrete, no Rio Grande do Sul.

Atualmente, a única etapa de seu negócio digitalizada é o controle de estoque. Há um serviço que, com atualizações na quantidade dos produtos em estoque, envia mensagens no protocolo AMQP (via [RabbitMQ](https://www.rabbitmq.com/)). Um dos maiores desafios que Seu Zé possui para embarcar na Delivery Much seria acompanhar a quantidade de produtos em estoque ao receber pedidos.

Sua tarefa como dev backend consistirá em auxiliar o Seu Zé no processo de digitalização de seu negócio, desenvolvendo um serviço que consuma essas atualizações de estoque e faça o controle de pedidos: rejeitando-os se os produtos não estiverem disponíveis e aceitando-os em caso contrário.

> Ao final do desafio, compartilhe um repositório **privado** no GitHub com a sua solução com o usuário [@deliverymuchdevs](https://github.com/deliverymuchdevs).

## Desafio

Para começar, Seu Zé irá disponibilizar um [.csv](products.csv) com a lista de produtos que seu mercado vende, com nome (`name`), preço (`price`) e quantidade (`quantity`). Você precisará criar um processo de carregamento desses dados (por exemplo: um script de populate) para armazená-los em uma base de dados de sua escolha para o projeto. Eles serão os dados iniciais para efetuar o controle de estoque, calcular o preço dos pedidos e validar a existência de produtos enviados pelo serviço de estoque, pois podem haver casos onde será enviado adição ou subtração de produtos que não estão na lista.

Seu projeto deve consumir as mensagens enviadas pelo serviço de estoque no RabbitMQ e atualizar a quantidade do produto enviado. Essas mensagens de atualizações de estoque serão enviadas ininterruptamente, com intervalo de 1 segundo entre cada envio.

Ele também deve ser uma API REST, contendo quatro endpoints para fazer as ações de criar e buscar pedidos e produtos. Avaliaremos a API em questão na estruturação da resposta conforme os exemplos na seção de [API](#API), código de status apropriado e desempenho.

### RabbitMQ

As mensagens de alteração de estoque de produto enviadas pelo serviço de estoque são disparadas na exchange `stock`, com a routing key `incremented` para adição e `decremented` para a subtração. O tipo da exchange é `direct`. O body da mensagem possui o nome do produto em questão como conteúdo, ex. `Lettuce`.

O serviço de estoque e o RabbitMQ estão disponíveis no [docker-compose.yml](docker-compose.yml) e serão executados pelo comando abaixo:

```shell
$ docker-compose up
```

O serviço do RabbitMQ estará disponível na porta: `5672` e a UI do RabbitMQ Management estará na porta: `15672` (`http://localhost:15672/`) com user: `guest` e password: `guest`.

  > Caso já exista uma instância do RabbitMQ rodando na sua máquina é preciso interrompê-la e utilizar a que é disponibilizada ao executar o docker-compose. 

Aguarde os containers terminarem de iniciar. Se der tudo certo, você verá o log de quais alimentos entraram no estoque:

```shell
stock-service_1  | Message sent to incremented:
stock-service_1  |     Tea
stock-service_1  | Message sent to decremented:
stock-service_1  |     Coffee
```

### API

Sua API REST deve conter os endpoints conforme:

1. Um deles deve retornar os produtos com o preço e quantidade em estoque atual:

```
[GET] /products/:name
```

Response exemplo: 

```json
{
  "name": "Brazil nut",
  "price": 5.16,
  "quantity": 5
}
```

2. Para registrar um pedido novo, seu serviço deverá receber uma lista de produtos do pedido via POST para o seguinte endpoint:

```
[POST] /orders
```

Body exemplo: 

```json
{
  "products": [
    {
      "name": "Kiwi",
      "quantity": 1
    }
  ]
}
```

Response exemplo:

```json
{
  "id": "42",
  "products": [
    {
      "name": "Kiwi",
      "quantity": 1,
      "price": 9.21
    }
  ],
  "total": 9.21
}
```

  > Lembre-se de fazer a checagem da disponibilidade do estoque dos produtos que vieram na requisição antes de registrar o pedido e de atualizar a quantidade dos produtos em estoque. Ao salvá-lo, crie também um identificador único `id` para o pedido.

3. Além disso, o serviço terá que possuir um endpoint para listar os pedidos realizados e aprovados, retornando os produtos do pedido e o valor total da compra. O endpoint de listagem deverá seguir o seguinte formato:

```
[GET] /orders
```

Response exemplo:

```json
{
  "orders": [
      {
        "id": "123",
        "products": [
          {
            "name": "Watermelon",
            "quantity": 2,
            "price": 5.47
          }
         ],
        "total": 10.94
     }
  ]
}
```

4. Endpoint irá retornar apenas pedidos individuais, dado um `id`:

```
[GET] /orders/:id
```

Response exemplo:

```json
{
  "id": "456",
  "products": [
    {
      "name": "Coffee",
      "quantity": 3,
      "price": 2.43
    }
  ],
  "total": 7.29
}
```
## Avaliação

A intenção principal deste desafio é avaliar suas habilidades em:

- Estruturar e armazenar dados de modo eficiente;
- Realizar comunicação com serviços externos;
- Escrever código legível, desacoplado e modularizado;
- Lidar com serviços de mensageria;
- Efetuar o design e arquitetura de APIs.

## Requisitos

- Utilizar Go ou Node.js;
- Disponibilizar documentação suficiente para a execução do projeto no README;
- Utilizar arquivos de ambiente para armazenar configurações/chaves de ambiente (caso precise);
- Atender os cenários de uso explicitados;
- Tratar erros e indisponibilidade de serviços externos;
- Desenvolver testes;
- Utilizar Docker;
- Compartilhar repositório privado da solução com [@deliverymuchdevs](https://github.com/deliverymuchdevs).
