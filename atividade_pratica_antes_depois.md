# Atividade Prática - MongoDB: Antes e Depois

## Configuração Inicial

```javascript
// Criar/selecionar banco de dados
use store;

// Criar coleção
db.createCollection("customers");

// Inserir os documentos iniciais
db.customers.insertMany([
    { "name": "Ana", "age": 25, "city": "Salvador", "active": true, "points": 120 },
    { "name": "Bruno", "age": 32, "city": "Feira de Santana", "active": true, "points": 300 },
    { "name": "Carlos", "age": 28, "city": "Salvador", "active": false, "points": 80 },
    { "name": "Daniela", "age": 40, "city": "São Paulo", "active": true, "points": 500 },
    { "name": "Eduarda", "age": 22, "city": "Rio de Janeiro", "active": false, "points": 50 }
]);
```

---

## Exercício 1 - Consulta

```javascript
// Buscar clientes de Salvador, exibindo apenas name e city
db.customers.find(
    { "city": "Salvador" },
    { "_id": 0, "name": 1, "city": 1 }
);
```

## Exercício 2 - Atualização

```javascript
// Atualizar Carlos para active: true
db.customers.updateOne(
    { "name": "Carlos" },
    { $set: { "active": true } }
);
```

## Exercício 3 - Atualizar vários documentos

```javascript
// Adicionar o campo state a todos os clientes de Salvador
db.customers.updateMany(
    { "city": "Salvador" },
    { $set: { "state": "BA" } }
);
```

## Exercício 4 - Incremento

```javascript
// Aumentar os pontos de Ana em 50 (120 -> 170)
db.customers.updateOne(
    { "name": "Ana" },
    { $inc: { "points": 50 } }
);
```

## Exercício 5 - Inserção

```javascript
// Inserir novo cliente
db.customers.insertOne({
    "name": "Fernando",
    "age": 29,
    "city": "Recife",
    "active": true,
    "points": 90
});
```

## Exercício 6 - Remoção

```javascript
// Remover Eduarda
db.customers.deleteOne({ "name": "Eduarda" });
```

## Exercício 7 - Criar um novo campo

```javascript
// Adicionar o campo vip a Daniela
db.customers.updateOne(
    { "name": "Daniela" },
    { $set: { "vip": true } }
);
```

## Exercício 8 - Remover um campo

```javascript
// Remover o campo points de Bruno
db.customers.updateOne(
    { "name": "Bruno" },
    { $unset: { "points": "" } }
);
```

## Exercício 9 - Ordenação

```javascript
// Ordenar clientes por idade, decrescente
db.customers.find().sort({ "age": -1 });
```

## Exercício 10 - Filtro com múltiplas condições

```javascript
// Clientes ativos com mais de 30 anos, exibindo apenas o nome
db.customers.find(
    { "active": true, "age": { $gt: 30 } },
    { "_id": 0, "name": 1 }
);
```

---

## Desafio

```javascript
// 1. Mostrar apenas os nomes dos clientes
db.customers.find({}, { "_id": 0, "name": 1 });

// 2. Contar quantos clientes existem
db.customers.countDocuments();

// 3. Contar apenas os clientes ativos
db.customers.countDocuments({ "active": true });

// 4. Mostrar o cliente com maior pontuação
db.customers.find().sort({ "points": -1 }).limit(1);

// 5. Mostrar o cliente com menor idade
db.customers.find().sort({ "age": 1 }).limit(1);

// 6. Mostrar clientes com pontuação entre 100 e 400
db.customers.find({ "points": { $gte: 100, $lte: 400 } });

// 7. Mostrar clientes das cidades de Salvador ou São Paulo
db.customers.find({ "city": { $in: ["Salvador", "São Paulo"] } });

// 8. Mostrar todos os clientes ordenados por nome
db.customers.find().sort({ "name": 1 });

// 9. Mostrar apenas os três primeiros clientes
db.customers.find().limit(3);

// 10. Mostrar apenas os clientes inativos
db.customers.find({ "active": false });
```
