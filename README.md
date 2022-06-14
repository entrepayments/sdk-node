# EntrePayments

## SDK de integração

O objetivo desse SDK é facilitar a integração, abstraindo toda a complexidade envolvida com os protocolos de comunicação com a API. Com esse SDK, o desenvolvedor precisará apenas utilizar alguns métodos e fornecer os dados informados pelo cliente, para executar quaisquer operações suportadas pelo SDK.

## Licença

Esse SDK utiliza licença livre MIT. A licença pode ser lida abaixo e no arquivo LICENSE, localizado na raiz desse SDK.

> MIT License
>
> Copyright (c) 2021 EntrePayments
> 
> Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
> 
> The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
> 
> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Recursos

* [Autorização com captura](#autorização-com-captura)
* [Autorização com captura e definição do softdescriptor](#autorização-com-captura-e-definição-do-softdescriptor)
* [Autorização sem captura - pré-autorização](#autorização-sem-captura---pré-autorização)
* [Autorização e tokenização do cartão](#autorização-e-tokenização-de-cartão)
* [Autorização com parcelamento](#autorização-com-parcelamento)
* [Verificação de cartão - transação zero dolar](#verificação-de-cartão---transação-zero-dolar)
* [Captura posterior](#captura-posterior)
* [Captura posterior parcial](#captura-posterior-parcial)
* [Cancelamento/reembolso](#cancelamentoreembolso)
* [Desfazimento/void](#desfasimentovoid)
* [Consulta](#consulta)
* [3DS](#3ds)

> Tanto a autorização com captura quanto a autorização sem captura suportam parcelamentos.
 
### Criação de uma instância EntrePayments

A instância de `EntrePayments` é a base de todas as operações da API; por isso o desenvolvedor precisará informar suas chaves:

```javascript
let merchantKey = "<sua-merchant-key>";
let merchantCode = "<seu-merchant-code>";
let terminal = "1";
let entrePayments = EntrePayments.create(
    new Merchant(
        merchantKey,
        merchantCode,
        terminal
    )
);
```

### Autorização com captura

Com a instância de `EntrePayments` criada, o processo é fluído; para autorizar e capturar uma transação, basta definir o parâmetro `capture` como true:

```javascript
let orderNumber = "1234-pedido";
let  orderAmount = 123;
let  payment = entrePayments.authorize(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.creditCard(
            "4548812049400004",
            "2023",
            "12",
            "Nome do portador",
            "123"
        )
    )
);
```

### Autorização com captura e definição do softdescriptor

Softdescriptor é o nome que aparece na fatura do cartão do cliente. Definir esse nome é útil para facilitar que o cliente identifique a origem da transação, evitando, assim, eventuais chargebacks.

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.authorize(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.creditCard(
            "4548812049400004",
            "2023",
            "12",
            "Nome do portador",
            "123"
        ),
        "LOJATESTE"
    )
);
```

### Autorização sem captura - pré-autorização

Assim como a autorização com captura basta definir o parâmetro `capture` como true, a sem captura basta definir o parâmetro `capture` como false:

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.authorize(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.creditCard(
            "4548812049400004",
            "2023",
            "12",
            "Nome do portador",
            "123"
        )
    ),
    false
);
```

### Autorização e tokenização de cartão

Durante o processo de autorização - com ou sem captura - é possível tokenizar o cartão para ser utilizado posteriormente. É o famoso one click payment, que permite que a loja armazene um token do cartão para compras futuras.

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.authorize(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.creditCard(
            "4548812049400004",
            "2023",
            "12",
            "Nome do portador",
            "123"
        )
    )
    ,
    true,
    true
);

let token = payment.card.identifier;
```

A tokenização funciona sem captura também:

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.authorize(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.creditCard(
            "4548812049400004",
            "2023",
            "12",
            "Nome do portador",
            "123"
        )
    )
    ,
    false,
    true
);

let token = payment.card.identifier;
```

A variável `$token` conterá um hash que poderá ser armazenado de forma segura pela loja. Numa futura compra, será possível autorizar uma nova compra utilizando o token em vez dos dados do cartão do cliente:

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.authorize(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.token("token-gerado-em-uma-transação-anterior")
    )
);
```

### Autorização com parcelamento

Também é possível definir o parcelamento de uma transação. O exemplo abaixo vai autorizar um pagamento em 12 vezes e vai gerar um token do cartão para ser utilizado em compras futuras.

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.authorize(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.creditCard(
            "4548812049400004",
            "2023",
            "12",
            "Nome do portador",
            "123"
        )
    ).setInstallments(12),
    true,
    true
);

let token = payment.cart.identifier;
```

### Verificação de cartão - transação zero dolar

Durante um processo de assinatura, é normal que a loja precise verificar o cartão do cliente. Para isso, é possível criar uma transação `zeroDolar`:

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.zeroDolar(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.creditCard(
            "4548812049400004",
            "2023",
            "12",
            "Nome do portador",
            "123"
        )
    )
);
```

Ao chamar o método `zeroDolar`, a classe `EntrePayments` vai definir o valor do pedido para `0` e vai enviar uma requisição para a API. Isso verificação se o cartão está ativo e se a loja poderá seguir com a venda, assinatura, etc. Um caso de uso para esse tipo de transação, é quando a loja está vendendo uma assinatura e só vai fazer a autorização ao final de um período; dessa forma, o cliente informa seus dados de cartão e é feita a verificação do cartão.

### Captura posterior

Um caso de uso comum para a pré-autorização, é em reservas. Por exemplo, o cliente reserva um quarto e faz checkin no hotel; nesse momento, o hotel faz a pré-autorização: 

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.authorize(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.creditCard(
            "4548812049400004",
            "2023",
            "12",
            "Nome do portador",
            "123"
        )
    ),
    false
);
```

Perceba o `capture: false`, que garante apenas a autorização. Quando o cliente faz o checkout, o hotel verifica consumo e outros e faz a captura do valor:

```javascript
let payment = entrePayments.capture(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        )
    )
);
```

Na pré-autorização, o valor é apenas travado no limite do cartão do cliente; somente após a captura que o valor é deduzido do limite do cliente.

### Captura posterior parcial

Pode ocorrer de o valor que se deseja capturar ser menor daquele que foi autorizado previamente. Nesse caso, o estabelecimento precisa informar o valor que deseja capturar:

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.authorize(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.creditCard(
            "4548812049400004",
            "2023",
            "12",
            "Nome do portador",
            "123"
        )
    ),
    false
);

// Em um segundo momento
payment = entryPayments.capture(
    new Payment(
        new Order(
            orderNumber,
            orderAmount - 100
        )
    )
);
```

## Cancelamento/reembolso

O cancelamento, ou reembolso, pode ocorrer após uma autorização ter sido capturada. Um detalhe importante aqui, é que a transação precisa ter sido capturada; caso a transação não tenha sido capturada, será necessário fazer um [desfazimento/void](#desfazimentovoid).

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.authorize(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.creditCard(
            "4761120000000148",
            "34",
            "12",
            "Fulano de tal",
            "123"
        )

    )
);

payment = entrePayments.cancel(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        )
    )
);
```

## Desfazimento/void

O desfazimento ou void acontece quando é feita uma autorização sem que se tenha feito a captura.

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.void(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        )
    )
);
```

## Consulta

Todas as transações e pagamentos de determinado pedido, podem ser consultados. Para isso, basta executar o método `consult`:

```javascript
let orderNumber = "1234-pedido";
let orderAmount = 123;
let payment = entrePayments.authorize(
    (new Payment(
        new Order(
            orderNumber,
            orderAmount
        ),
        Card.creditCard(
            "4761120000000148",
            "34",
            "12",
            "Fulano de tal",
            "123"
        )
    ))
);

payment = entrePayments.consult(
    new Payment(
        new Order(
            orderNumber,
            orderAmount
        )
    ),
    Payment.AUTHORIZATION_WITHOUT_3DS
);
```
