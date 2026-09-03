# CRUD - MongoDB

## Create

### Banco de Dados

```javascript
// Exibir os bancos de dados
show databases;

// Criar/selecionar banco de dados
use loja_informatica;
```

### Collections

```javascript
// Criar nova collection
db.createCollection("cliente");

// Mostrar todas as collections
show collections;
```

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

**Métodos utilizados em Delete**

| Método | Descrição |
|---|---|
| `deleteOne()` | Remove o primeiro documento que corresponde ao filtro |
| `deleteMany()` | Remove todos os documentos que correspondem ao filtro |
| `remove()` | Método alternativo/mais antigo para remoção de documentos |
