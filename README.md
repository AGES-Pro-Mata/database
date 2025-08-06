# DynamoDB

### O que é o DynamoDB e como funciona?

O [**Amazon DynamoDB**](https://aws.amazon.com/dynamodb/) é um banco de dados NoSQL totalmente gerenciado pela AWS, projetado para oferecer alta performance, escalabilidade automática e baixa latência. Ele é baseado em **tabelas** e armazena os dados como **pares de chave-valor** ou **estruturas de dados semi-estruturadas (sem schema fixo)**.

Cada tabela no DynamoDB exige a definição de uma **chave primária**, que pode ser:

- **Simples** (apenas uma `partition key`)
- **Composta** (uma `partition key` + `sort key`).

A partir dessa chave, o DynamoDB organiza e indexa os dados, permitindo leitura rápida, mesmo em grandes volumes.
### Uso com Node.js: abstração baseada em documentos

Na versão [**v3 do SDK da AWS para Node.js**](https://www.npmjs.com/package/@aws-sdk/client-dynamodb), a interação padrão com o DynamoDB exige que os dados sejam enviados e recebidos no **formato bruto**, ou seja, cada atributo precisa ser descrito com seu tipo:


```js
// Formato bruto
const item = {
  id: { S: '123' },
  age: { N: '30' }
}
```

Para resolver isso, o SDK v3 oferece o pacote [**`@aws-sdk/util-dynamodb`**](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-util-dynamodb/), que fornece utilitários como:

- `marshall`: converte objetos JavaScript em formato compatível com o DynamoDB (tipado);
- `unmarshall`: converte dados do DynamoDB para objetos JavaScript simples.

Exemplo:

```js
import { marshall, unmarshall } from "@aws-sdk/util-dynamodb";

const item = marshall({ id: '123', name: 'Kayky' });
// => { id: { S: '123' }, name: { S: 'Kayky' } }

const plain = unmarshall(item);
// => { id: '123', name: 'Kayky' }
```

Com isso, conseguimos uma **experiência semelhante à de bancos baseados em documentos** (como o MongoDB), facilitando muito o trabalho com objetos comuns em JavaScript.

 **Importante:** A maior parte dos tutoriais disponíveis na internet ainda utiliza o SDK v2, onde existia o `DynamoDB.DocumentClient` para esse tipo de conversão automática. No SDK v3, esse cliente foi substituído por essa abordagem manual com `marshall/unmarshall`.
#### Alternativa mais prática: `@aws-sdk/lib-dynamodb` e o `DynamoDBDocumentClient`

Além do uso direto do `marshall` e `unmarshall` com o `@aws-sdk/util-dynamodb`, a AWS oferece uma camada de abstração ainda mais conveniente no pacote [**`@aws-sdk/lib-dynamodb`**](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-lib-dynamodb/), que fornece o **`DynamoDBDocumentClient`**.

##### Exemplo de uso com `DynamoDBDocumentClient`:
```js
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import {
  DynamoDBDocumentClient,
  PutCommand,
  GetCommand
} from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

// Inserindo um item
await docClient.send(new PutCommand({
  TableName: "Users",
  Item: {
    id: "123",
    name: "Kayky",
    age: 28
  }
}));

// Buscando um item
const result = await docClient.send(new GetCommand({
  TableName: "Users",
  Key: { id: "123" }
}));

console.log(result.Item); // => { id: '123', name: 'Kayky', age: 28 }
```

Esse cliente documentado encapsula automaticamente as chamadas `marshall` e `unmarshall`, permitindo que você envie e receba objetos JavaScript simples diretamente — **sem precisar fazer a conversão manual** toda vez.
### Relações entre tabelas no DynamoDB

O DynamoDB **não possui relacionamentos nativos** como bancos de dados relacionais. Porém, é possível modelar relacionamentos de forma estratégica, de acordo com os padrões de acesso da aplicação:

- **1-to-1**: pode-se armazenar os dados em uma mesma tabela, compartilhando a mesma chave primária.
- **1-to-many**: comum usar a mesma `partition key` e diferenciar com a `sort key` (ex: posts de um usuário).
- **many-to-many**: exige tabelas auxiliares (join tables) ou duplicação de dados (denormalização), já que **joins não são suportados**.

Essa modelagem depende fortemente da **forma como os dados serão lidos**, não apenas de como estão organizados.

### Links Importantes
(Podem ir adicionando mais links conforme forem achando)

- [Relacionamentos de dados](https://www.devmedia.com.br/modelagem-de-dados-2-os-relacionamentos/4142)
- [Amazon DynamoDB](https://docs.aws.amazon.com/dynamodb/) (Documentação)
- [Programar o Amazon DynamoDB com JavaScript](https://docs.aws.amazon.com/pt_br/amazondynamodb/latest/developerguide/programming-with-javascript.html)
- [Modelagem de dados com DynamobDB](https://docs.aws.amazon.com/pt_br/amazondynamodb/latest/developerguide/data-modeling.html)
- [NoSQL Workbench para DynamoDB](https://docs.aws.amazon.com/pt_br/amazondynamodb/latest/developerguide/workbench.html) (Ferramenta de visualização de bancos NoSQL)
