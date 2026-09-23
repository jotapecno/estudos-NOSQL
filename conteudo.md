# CRUD - MongoDB

## Create

### Banco de Dados

```javascript
// Exibir os bancos de dados
show databases;

// Criar/selecionar banco de dados
use loja_informatica;
```

O `show databases` lista todos os bancos existentes no servidor. O `use` seleciona o banco `loja_informatica`; se ele ainda não existir, o MongoDB só o cria de fato quando o primeiro dado é gravado dentro dele.

### Collections

```javascript
// Criar nova collection
db.createCollection("cliente");

// Mostrar todas as collections
show collections;
```

O `createCollection` cria a estrutura `cliente` dentro do banco atual. O `show collections` lista todas as collections já existentes no banco selecionado.

### Inserção de Documentos

```javascript
// Inserir um documento
db.cliente.insertOne({
    "nome": "jefté"
});

// Inserir um documento com diferentes tipos de dados
db.cliente.insertOne({
    "nome": "jefté",
    "idade": 35,
    "pets": ["dora", "sabrina"],
    "endereco": {
        "logradouro": "Sossego"
    }
});

// Inserir vários documentos de uma vez
db.cliente.insertMany([
    { "nome": "Brenno" },
    { "nome": "João" },
    { "nome": "MAria" },
    { "nome": "José" },
    { "nome": "Noé" }
]);

// Inserir documento com valor decimal
db.products.insertOne({
    "name": "shoes",
    "price": 20.3
});

// Inserir documento definindo um _id personalizado
db.products.insertOne({
    "name": "gloves",
    "_id": "my-custom-id"
});

// Inserir vários documentos de uma vez
db.products.insertMany([
    { "name": "gloves" },
    { "name": "shoes" }
]);
```

O `insertOne()` adiciona um único documento à collection, e cada documento pode ter uma estrutura diferente — em `cliente`, por exemplo, alguns documentos têm apenas `nome`, enquanto outros têm campos aninhados como `endereco` e arrays como `pets`. O MongoDB também aceita valores decimais diretamente, como em `price: 20.3`, e permite definir manualmente o `_id` de um documento (como em `"my-custom-id"`), em vez de deixar o banco gerar um `ObjectId` automático. Já o `insertMany()` insere vários documentos de uma vez, recebidos em formato de array.

**Métodos utilizados em Create**

| Método | Descrição |
|---|---|
| `insertOne()` | Insere um único documento na collection |
| `insertMany()` | Insere vários documentos de uma vez, em formato de array |

---

## Read

```javascript
// Mostrar todos os documentos/objetos
db.cliente.find();

// Buscar documento pelo campo
db.cliente.find({
    "nome": "José"
});

// Buscar documento pelo identificador único
db.cliente.find({
    "_id": ObjectId("6a7bbab007ff2cf8649f68a9")
});

// Buscar documento pelo campo
db.products.find({
    "name": "gloves"
});

// Buscar documentos usando operador de comparação
db.products.find({
    "price": { $gt: 10 } // gt = Greater than
});
```

O `find()` sem parâmetros retorna todos os documentos da collection. Passando um filtro entre chaves, ele retorna apenas os documentos que correspondem à condição informada — seja por um campo comum, como `nome` ou `name`, seja pelo `_id`, que é o identificador único gerado (ou definido) para cada documento. O `$gt` é um operador de comparação e filtra apenas os documentos em que o campo `price` é maior que o valor informado.

**Operadores utilizados em Read**

| Operador | Significado |
|---|---|
| `$gt` | Greater than — maior que |

---

## Update

```javascript
// Atualizar um campo existente
db.cliente.updateOne(
    { "nome": "MAria" },
    { $set: { "nome": "Maria" } }
);

// Adicionar um novo campo ao documento
db.cliente.updateOne(
    { "nome": "Maria" },
    {
        $set: {
            "endereco": {
                "logradouro": "sossego"
            }
        }
    }
);

// Atualizar o primeiro documento que corresponde ao filtro
db.products.updateOne(
    { "name": "bag" },
    { $set: { "name": "gloves" } }
);

// Atualizar o primeiro documento da collection (filtro vazio)
db.products.updateOne(
    {},
    { $set: { "name": "shirt" } }
);

// Atualizar vários documentos de uma vez
db.products.updateMany(
    {},
    { $set: { "marker": "toDelete" } }
);

// Substituir um documento inteiro pelo _id
db.products.replaceOne(
    { "_id": ObjectId("6a7209b7cf1fb8d2fd794e20") },
    { "name": "gloves" }
);
```

O `updateOne()` localiza o primeiro documento que corresponde ao filtro e aplica a alteração indicada com `$set`, que tanto corrige o valor de um campo existente (como o nome "MAria" para "Maria") quanto adiciona um campo novo ao documento (como o `endereco`). Quando o filtro é vazio (`{}`), a atualização é aplicada ao primeiro documento da collection, sem distinção. Já o `updateMany()` aplica a mesma alteração a **todos** os documentos que atendem ao filtro — nesse caso, com filtro vazio, a todos os documentos da collection. O `replaceOne()` funciona de forma diferente: em vez de alterar campos específicos, ele substitui o documento inteiro pelo novo objeto informado, mantendo apenas o `_id` original.

**Métodos utilizados em Update**

| Método | Descrição |
|---|---|
| `updateOne()` | Atualiza o primeiro documento que corresponde ao filtro |
| `updateMany()` | Atualiza todos os documentos que correspondem ao filtro |
| `replaceOne()` | Substitui o documento inteiro, mantendo apenas o `_id` |
| `$set` | Define/atualiza o valor de um campo |

---

## Delete

```javascript
// Remover o primeiro documento da collection (filtro vazio)
db.products.deleteOne({});

// Remover o primeiro documento que corresponde ao filtro
db.products.deleteOne({
    "name": "shoes"
});

// Remover todos os documentos que correspondem ao filtro
db.products.deleteMany({
    "marker": "toDelete"
});

// Remover documentos usando o método remove() (alternativo)
db.products.remove({});
```

O `deleteOne()` remove apenas o primeiro documento encontrado — com filtro vazio, remove o primeiro da collection; com um filtro específico, remove o primeiro que corresponder a ele. O `deleteMany()` remove todos os documentos que atendem ao filtro, como os que foram marcados anteriormente com `marker: "toDelete"` no exemplo do Update. Por fim, `remove()` é um método mais antigo do MongoDB que também remove documentos, mas hoje é considerado descontinuado em favor de `deleteOne()` e `deleteMany()`.

**Métodos utilizados em Delete**

| Método | Descrição |
|---|---|
| `deleteOne()` | Remove o primeiro documento que corresponde ao filtro |
| `deleteMany()` | Remove todos os documentos que correspondem ao filtro |
| `remove()` | Método alternativo/mais antigo para remoção de documentos |

---

## Relacionamentos entre Documentos

No MongoDB não existem "joins" como em bancos relacionais. Para modelar relações entre dados, existem duas estratégias principais: **embarcar** (guardar o dado relacionado dentro do próprio documento) ou usar **referência** (guardar apenas o `ObjectId` de outro documento, feito em outra collection).

### One-to-One (um para um)

**Embarcado**

```javascript
db.patients.insertOne({
    name: "Jefté",
    age: 35,
    diseaseSummary: {
        diseases: ["cold", "broken leg"]
    }
});
```

Os dados relacionados (`diseaseSummary`) ficam guardados dentro do próprio documento do paciente, como um subdocumento.

**Por referência**

```javascript
db.persons.insertOne({
    name: "Jefté",
    age: 35,
    salary: 3000
});

db.cars.insertOne({
    model: "BMW",
    price: 40000,
    owner: ObjectId('6aa9e2cee9c288ce1241317e')
});
```

Aqui, `persons` e `cars` são collections separadas. O documento do carro não guarda os dados da pessoa, apenas o `ObjectId` dela no campo `owner`, criando a referência entre os dois.

### One-to-Many (um para muitos)

**Embarcado**

```javascript
db.questionThreads.insertOne({
    creator: "Jefté",
    question: "How does that work?",
    answers: [
        { text: "Like that." },
        { text: "Thanks!" }
    ]
});
```

As várias respostas (`answers`) ficam embarcadas como um array de subdocumentos dentro da própria thread de pergunta.

**Referência**

```javascript
db.cities.insertOne({
    name: "New York City",
    coordinates: { lat: 21, lng: 55 }
});

db.citizens.insertMany([
    { name: "Jefté Goes", cityId: ObjectId("5b98d6b44d01c52e1637a99f") },
    { name: "Brenno Salvador", cityId: ObjectId("5b98d6b44d01c52e1637a99f") }
]);
```

Cada cidadão, na collection `citizens`, guarda apenas o `cityId` referenciando o `_id` do documento correspondente em `cities`. Vários cidadãos podem referenciar a mesma cidade.

### Many-to-Many (muitos para muitos)

**Embarcado**

```javascript
db.customers.insertOne({
    name: "Jefté",
    age: 35
});

db.customers.updateOne(
    {},
    { $set: { orders: [{ title: "A Book", price: 12.99, quantity: 2 }] } }
);
```

Os pedidos (`orders`) do cliente são embarcados diretamente dentro do próprio documento do cliente, como um array de subdocumentos.

**Referência**

```javascript
db.authors.insertMany([
    { name: "Jorge Amado", age: 78, address: { street: "Bahia" } },
    { name: "Graciliano Ramos", age: 55, address: { street: "Rio de Janeiro" } }
]);

db.books.updateOne(
    {},
    { $set: { authors: [ObjectId("5b98d9e44d01c52e1637a9a6"), ObjectId("5b98d9e44d01c52e1637a9a7")] } }
);
```

Um livro pode ter vários autores, e um autor pode ter vários livros. Por isso, o documento do livro guarda um array de `ObjectId`s referenciando os autores correspondentes na collection `authors` — essa é a forma típica de modelar relação muitos-para-muitos por referência.

### Resumo: Embarcado vs Referência

| Estratégia | Quando usar | Vantagem | Desvantagem |
|---|---|---|---|
| **Embarcado** | Dados sempre acessados juntos, relação simples (1:1 ou 1:N pequeno) | Leitura rápida (um único documento) | Documento pode crescer muito; duplicação de dados |
| **Referência** | Dados grandes, reutilizados por vários documentos, ou relação N:N | Evita duplicação, mantém documentos menores | Precisa de consultas extras (ou `$lookup`) para juntar os dados |
