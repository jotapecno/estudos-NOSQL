# Atividade Prática - MongoDB: Antes e Depois

## Contexto da Atividade

Esta atividade simula a administração do banco de dados de uma loja online. Foi criado um banco chamado `store` e, dentro dele, uma coleção chamada `customers`, representando os clientes cadastrados na loja. A partir de um conjunto inicial de documentos, foram realizadas operações de consulta, atualização, inserção e remoção, sempre observando o estado da coleção **antes** e **depois** de cada comando.

---

## Configuração Inicial

```javascript
use store;

db.createCollection("customers");

db.customers.insertMany([
    { "name": "Ana", "age": 25, "city": "Salvador", "active": true, "points": 120 },
    { "name": "Bruno", "age": 32, "city": "Feira de Santana", "active": true, "points": 300 },
    { "name": "Carlos", "age": 28, "city": "Salvador", "active": false, "points": 80 },
    { "name": "Daniela", "age": 40, "city": "São Paulo", "active": true, "points": 500 },
    { "name": "Eduarda", "age": 22, "city": "Rio de Janeiro", "active": false, "points": 50 }
]);
```

O `use` seleciona o banco `store`; se ele ainda não existir, o MongoDB o cria assim que o primeiro dado é gravado. O `createCollection` cria a coleção `customers`, e o `insertMany` insere de uma vez os 5 clientes iniciais, em formato de array.

---

## Exercício 1 - Consulta

**Antes:** todos os documentos da coleção, com todos os campos.
**Depois:** apenas os clientes de Salvador, mostrando somente `name` e `city`.

```javascript
db.customers.find(
    { "city": "Salvador" },
    { "_id": 0, "name": 1, "city": 1 }
);
```

O primeiro parâmetro do `find()` é o filtro, usado para selecionar os documentos de Salvador. O segundo é a projeção, que define quais campos aparecem no resultado: `1` exibe o campo, `0` oculta — nesse caso, o `_id` foi ocultado.

---

## Exercício 2 - Atualização

**Antes:** `Carlos` estava com `active: false`.
**Depois:** `Carlos` passa a ter `active: true`.

```javascript
db.customers.updateOne(
    { "name": "Carlos" },
    { $set: { "active": true } }
);
```

O `updateOne()` localiza o primeiro documento que corresponde ao filtro e aplica a alteração indicada. O `$set` altera o valor de um campo já existente, sem afetar os demais campos do documento.

---

## Exercício 3 - Atualizar vários documentos

**Antes:** clientes de Salvador não possuíam o campo `state`.
**Depois:** todos os clientes de Salvador passam a ter `state: "BA"`.

```javascript
db.customers.updateMany(
    { "city": "Salvador" },
    { $set: { "state": "BA" } }
);
```

O `updateMany()` aplica a alteração a **todos** os documentos que atendem ao filtro, diferente do `updateOne()`, que altera apenas o primeiro encontrado. Como Ana e Carlos são de Salvador, os dois recebem o novo campo.

---

## Exercício 4 - Incremento

**Antes:** Ana tinha `points: 120`.
**Depois:** Ana passa a ter `points: 170`.

```javascript
db.customers.updateOne(
    { "name": "Ana" },
    { $inc: { "points": 50 } }
);
```

O `$inc` soma o valor informado ao que já está salvo no campo numérico, sem precisar calcular o resultado final manualmente.

---

## Exercício 5 - Inserção

**Antes:** a coleção tinha 5 documentos.
**Depois:** a coleção passa a ter 6 documentos, com a entrada de Fernando.

```javascript
db.customers.insertOne({
    "name": "Fernando",
    "age": 29,
    "city": "Recife",
    "active": true,
    "points": 90
});
```

O `insertOne()` adiciona um único documento novo à coleção.

---

## Exercício 6 - Remoção

**Antes:** Eduarda existia na coleção.
**Depois:** o documento de Eduarda não existe mais.

```javascript
db.customers.deleteOne({ "name": "Eduarda" });
```

O `deleteOne()` remove o primeiro documento que corresponde ao filtro informado.

---

## Exercício 7 - Criar um novo campo

**Antes:** Daniela não tinha o campo `vip`.
**Depois:** Daniela passa a ter `vip: true`.

```javascript
db.customers.updateOne(
    { "name": "Daniela" },
    { $set: { "vip": true } }
);
```

O `$set` também é usado para criar um campo que ainda não existia no documento — nesse caso, o campo `vip` é adicionado apenas a Daniela.

---

## Exercício 8 - Remover um campo

**Antes:** Bruno tinha o campo `points: 300`.
**Depois:** o campo `points` não existe mais no documento de Bruno.

```javascript
db.customers.updateOne(
    { "name": "Bruno" },
    { $unset: { "points": "" } }
);
```

O `$unset` remove o campo inteiro do documento. O valor `""` não tem efeito sobre o resultado, é apenas uma exigência de sintaxe do comando.

---

## Exercício 9 - Ordenação

**Antes:** os documentos estavam sem nenhuma ordem definida.
**Depois:** os clientes aparecem ordenados por idade, do mais velho para o mais novo.

```javascript
db.customers.find().sort({ "age": -1 });
```

O `sort()` ordena o resultado do `find()`. O valor `1` ordena de forma crescente e o `-1` de forma decrescente.

---

## Exercício 10 - Filtro com múltiplas condições

**Antes:** todos os documentos, sem filtro.
**Depois:** apenas os clientes ativos e com mais de 30 anos.

```javascript
db.customers.find(
    { "active": true, "age": { $gt: 30 } },
    { "_id": 0, "name": 1 }
);
```

As duas condições dentro do filtro, separadas por vírgula, funcionam como um "E": o documento só é retornado se atender às duas ao mesmo tempo. O `$gt` filtra idades maiores que 30, e a projeção limita o resultado ao campo `name`.

---

## Desafio

Nesta parte, o objetivo era apenas **consultar** os dados de formas diferentes, sem alterar nenhum documento da coleção.

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

O `countDocuments()` conta quantos documentos atendem a um filtro (ou a coleção toda, se o filtro for vazio). Para encontrar o maior ou o menor valor de um campo, a lógica usada foi ordenar com `sort()` e limitar o resultado a 1 com `limit()`. O `$gte` e o `$lte` funcionam como o `$gt`, mas incluem o próprio valor de comparação — necessário aqui para o intervalo "entre 100 e 400" incluir as pontas. O `$in` compara um campo com uma lista de valores possíveis, evitando escrever uma condição separada para cada cidade. Por fim, `limit()` também pode ser usado sozinho, sem `sort()`, apenas para restringir a quantidade de resultados retornados, como no item 9.
