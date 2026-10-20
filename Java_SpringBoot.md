# CRUD - Spring Boot

Este é o mesmo CRUD do MongoDB, agora feito por uma **API REST** com Java e Spring Boot. Os dados são os do `cliente`, iguais aos da collection do CRUD de MongoDB.

Para não depender de nenhum banco de dados específico, os clientes ficam guardados em uma **lista na memória**. A camada **Repository** é quem cuida disso, e mais tarde ela pode ser trocada por um banco real (JPA, MongoDB etc.) sem mexer no Service nem no Controller. Como a lista fica na memória, **os dados somem quando a aplicação para**.

---

## Preparação

### Métodos HTTP e CRUD

Cada operação do CRUD corresponde a um método HTTP:

| Operação | Método HTTP | Rota | Equivalente no MongoDB |
|---|---|---|---|
| Create | `POST` | `/cliente` | `insertOne()` |
| Read | `GET` | `/cliente`, `/cliente?nome=José`, `/cliente/{id}` | `find()` |
| Update | `PUT` | `/cliente/{id}` | `replaceOne()` |
| Delete | `DELETE` | `/cliente/{id}` | `deleteOne()` |

Os dados viajam em formato **JSON**, parecido com os documentos do MongoDB.

### Arquitetura em camadas

| Camada | Responsabilidade |
|---|---|
| **Model** | Representa os dados (`Cliente`) |
| **Controller** | Recebe as requisições HTTP e devolve as respostas |
| **Service** | Processa a lógica de negócio |
| **Repository** | Guarda e busca os dados (aqui, em uma lista na memória) |

Caminho de uma requisição:

```
Requisição -> Controller -> Service -> Repository
```

A resposta volta pelo mesmo caminho.

### Criando o projeto

O projeto é gerado no **Spring Initializr** (start.spring.io):

| Campo | Escolha |
|---|---|
| Project | Maven |
| Language | Java |
| Spring Boot | A versão estável padrão (sem SNAPSHOT nem M) |
| Java | 17 |

**Dependências**

| Dependência | Para que serve |
|---|---|
| **Spring Web** | Criar os endpoints HTTP |
| **Spring Boot DevTools** (opcional) | Reiniciar a aplicação ao salvar alterações |

Não há dependência de banco de dados (nem JPA, nem H2, nem MongoDB). Com o DevTools ligado, a aplicação reinicia a cada alteração salva, e a lista de clientes volta vazia.

Depois de abrir o projeto na IDE, a classe principal (com `@SpringBootApplication`) é executada pelo triângulo verde ao lado do `main`. Quando o log mostra `Started ... Application`, a API está no ar em `http://localhost:8080`.

Para testar, usa-se o **Postman**: escolhe-se o método, digita-se a URL e, nas requisições com corpo, vai-se em **Body → raw → JSON**.

**Estrutura de pastas**

```
src/main/java/com/example/demo
├── DemoApplication.java
├── model
│   └── Cliente.java
├── repository
│   └── ClienteRepository.java
├── service
│   └── ClienteService.java
└── controller
    └── ClienteController.java
```

---

## Código Base

### Model

```java
package com.example.demo.model;

public record Cliente(Long id, String nome, Integer idade) {
}
```

O `record` é uma classe enxuta, própria para guardar dados: ele já cria o construtor, os métodos de acesso (`id()`, `nome()`, `idade()`) e o `toString()`. Aqui o `Cliente` tem uma estrutura **fixa** (`id`, `nome` e `idade`), diferente de um documento do MongoDB, em que cada documento pode ter campos diferentes.

### Repository

```java
package com.example.demo.repository;

import com.example.demo.model.Cliente;
import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

@Repository
public class ClienteRepository {

    private final Map<Long, Cliente> clientes = new LinkedHashMap<>();
    private long proximoId = 1;

    public Cliente save(Cliente cliente) {
        Long id = cliente.id();
        if (id == null) {
            id = proximoId++;
        }
        Cliente salvo = new Cliente(id, cliente.nome(), cliente.idade());
        clientes.put(id, salvo);
        return salvo;
    }

    public List<Cliente> findAll() {
        return new ArrayList<>(clientes.values());
    }

    public List<Cliente> findByNome(String nome) {
        return clientes.values().stream()
                .filter(c -> c.nome() != null && c.nome().equalsIgnoreCase(nome))
                .toList();
    }

    public Optional<Cliente> findById(Long id) {
        return Optional.ofNullable(clientes.get(id));
    }

    public boolean existsById(Long id) {
        return clientes.containsKey(id);
    }

    public void deleteById(Long id) {
        clientes.remove(id);
    }
}
```

O `@Repository` indica que a classe cuida do armazenamento dos dados. Os clientes ficam em um `Map` (o `id` é a chave) e o `proximoId` gera os identificadores em sequência, como o `_id` que o MongoDB cria sozinho. Os nomes dos métodos (`save`, `findAll`, `findById`, `deleteById`) seguem os do `JpaRepository`, o que facilita trocar por um banco real depois. É uma estrutura simples, feita só para estudo, sem controle de acessos simultâneos.

### Service

```java
package com.example.demo.service;

import com.example.demo.model.Cliente;
import com.example.demo.repository.ClienteRepository;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class ClienteService {

    private final ClienteRepository clienteRepository;

    public ClienteService(ClienteRepository clienteRepository) {
        this.clienteRepository = clienteRepository;
    }

    public Cliente criar(Cliente cliente) {
        return clienteRepository.save(new Cliente(null, cliente.nome(), cliente.idade()));
    }

    public List<Cliente> listar(String nome) {
        if (nome == null) {
            return clienteRepository.findAll();
        }
        return clienteRepository.findByNome(nome);
    }

    public Optional<Cliente> buscarPorId(Long id) {
        return clienteRepository.findById(id);
    }

    public Optional<Cliente> atualizar(Long id, Cliente cliente) {
        if (!clienteRepository.existsById(id)) {
            return Optional.empty();
        }
        return Optional.of(clienteRepository.save(new Cliente(id, cliente.nome(), cliente.idade())));
    }

    public boolean remover(Long id) {
        if (!clienteRepository.existsById(id)) {
            return false;
        }
        clienteRepository.deleteById(id);
        return true;
    }
}
```

O `@Service` indica que a classe guarda a lógica de negócio. O `ClienteRepository` chega pelo construtor: o Spring cria o objeto e entrega pronto, sem uso de `new`. Isso é a **injeção de dependência**. No `criar`, o `id` é sempre `null`, para que o repositório gere um novo (qualquer `id` enviado pelo usuário é ignorado). O `atualizar` e o `remover` conferem se o cliente existe antes de agir. O `Optional` representa um resultado que pode não existir.

### Controller

```java
package com.example.demo.controller;

import com.example.demo.model.Cliente;
import com.example.demo.service.ClienteService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/cliente")
public class ClienteController {

    private final ClienteService clienteService;

    public ClienteController(ClienteService clienteService) {
        this.clienteService = clienteService;
    }

    @PostMapping
    public ResponseEntity<Cliente> criar(@RequestBody Cliente cliente) {
        Cliente criado = clienteService.criar(cliente);
        return ResponseEntity.status(HttpStatus.CREATED).body(criado);
    }

    @GetMapping
    public List<Cliente> listar(@RequestParam(required = false) String nome) {
        return clienteService.listar(nome);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Cliente> buscarPorId(@PathVariable Long id) {
        return clienteService.buscarPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PutMapping("/{id}")
    public ResponseEntity<Cliente> atualizar(@PathVariable Long id, @RequestBody Cliente cliente) {
        return clienteService.atualizar(id, cliente)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> remover(@PathVariable Long id) {
        if (clienteService.remover(id)) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
}
```

O `@RestController` indica que a classe responde requisições HTTP devolvendo dados (JSON), e o `@RequestMapping("/cliente")` define a rota base. Cada método responde a um verbo HTTP. O `@RequestBody` converte o JSON recebido em um `Cliente`, o `@PathVariable` pega o `id` que vem na URL e o `@RequestParam` lê um parâmetro de busca (como `?nome=José`). O `ResponseEntity` permite escolher o status da resposta. O Controller não mexe nos dados: ele só chama o Service.

**Anotações utilizadas**

| Anotação | Descrição |
|---|---|
| `@Repository` | Classe que guarda e busca os dados |
| `@Service` | Classe de lógica de negócio |
| `@RestController` | Classe que responde requisições HTTP com dados |
| `@RequestMapping` | Define a rota base (`/cliente`) |
| `@PostMapping` | Mapeia requisições `POST` |
| `@GetMapping` | Mapeia requisições `GET` |
| `@PutMapping` | Mapeia requisições `PUT` |
| `@DeleteMapping` | Mapeia requisições `DELETE` |
| `@RequestBody` | Converte o corpo (JSON) da requisição em objeto |
| `@PathVariable` | Lê um valor que vem na URL |
| `@RequestParam` | Lê um parâmetro de busca da URL |

---

## Create

```http
# Cadastrar um cliente
POST http://localhost:8080/cliente
Content-Type: application/json

{
    "nome": "João",
    "idade": 20
}

# Cadastrar um cliente informando só o nome
POST http://localhost:8080/cliente
Content-Type: application/json

{
    "nome": "jefté"
}
```

Resposta do primeiro `POST` (status `201 Created`):

```json
{
    "id": 1,
    "nome": "João",
    "idade": 20
}
```

O `POST` cadastra **um** cliente e devolve o cliente criado, já com o `id` gerado. Para cadastrar vários (como o `insertMany()` do MongoDB), repete-se o `POST` para cada um. Os campos que não são enviados ficam `null`, como a `idade` do segundo exemplo, já que a estrutura do `Cliente` é fixa.

**Métodos utilizados em Create**

| Método | Descrição |
|---|---|
| `POST /cliente` | Cadastra um novo cliente |
| `criar()` | Método do Controller/Service que trata o cadastro |
| `save()` | Método do Repository que guarda o cliente na lista |

---

## Read

```http
# Mostrar todos os clientes
GET http://localhost:8080/cliente

# Buscar clientes pelo nome
GET http://localhost:8080/cliente?nome=José

# Buscar cliente pelo identificador único
GET http://localhost:8080/cliente/1
```

Resposta do primeiro `GET` (status `200 OK`):

```json
[
    { "id": 1, "nome": "João", "idade": 20 },
    { "id": 2, "nome": "jefté", "idade": null }
]
```

O `GET /cliente` sem parâmetros retorna todos os clientes, como o `find()` sem filtro. Com `?nome=José`, retorna só os que têm aquele nome, como o `find({ "nome": "José" })`. Já o `GET /cliente/1` busca pelo `id`, como o `find({ "_id": ... })`, e responde `404 Not Found` quando o `id` não existe.

**Métodos utilizados em Read**

| Método | Descrição |
|---|---|
| `GET /cliente` | Lista todos os clientes |
| `GET /cliente?nome=...` | Lista os clientes com aquele nome |
| `GET /cliente/{id}` | Busca um cliente pelo `id` |
| `findAll()` | Retorna todos os clientes da lista |
| `findByNome()` | Retorna os clientes com o nome informado |
| `findById()` | Retorna o cliente do `id` informado, se existir |

---

## Update

```http
# Atualizar um cliente
PUT http://localhost:8080/cliente/1
Content-Type: application/json

{
    "nome": "João Paulo",
    "idade": 21
}
```

Resposta (status `200 OK`):

```json
{
    "id": 1,
    "nome": "João Paulo",
    "idade": 21
}
```

O `PUT /cliente/1` localiza o cliente do `id` informado e **substitui todos os campos** pelos enviados no corpo, mantendo o mesmo `id`. Funciona como o `replaceOne()` do MongoDB: um campo que não for enviado volta a ser `null`. Se o `id` não existir, a resposta é `404 Not Found`. Para alterar apenas um campo, existe o método `PATCH`, que não é usado aqui.

**Métodos utilizados em Update**

| Método | Descrição |
|---|---|
| `PUT /cliente/{id}` | Atualiza (substitui) os dados de um cliente |
| `atualizar()` | Método do Controller/Service que trata a atualização |
| `existsById()` | Confere se o cliente existe antes de atualizar |
| `save()` | Guarda o cliente atualizado, mantendo o `id` |

---

## Delete

```http
# Remover um cliente
DELETE http://localhost:8080/cliente/1
```

A resposta é `204 No Content` (sem corpo). Se o mesmo `DELETE` for repetido, o cliente já não existe e a resposta passa a ser `404 Not Found`.

O `DELETE /cliente/1` remove o cliente do `id` informado, como o `deleteOne()` do MongoDB. O Service confere se ele existe antes de apagar, e o Repository retira o cliente da lista.

**Métodos utilizados em Delete**

| Método | Descrição |
|---|---|
| `DELETE /cliente/{id}` | Remove o cliente do `id` informado |
| `remover()` | Método do Controller/Service que trata a remoção |
| `deleteById()` | Retira o cliente da lista |

---

## Status HTTP Utilizados

| Status | Significado | Onde aparece |
|---|---|---|
| `200 OK` | Deu certo | `GET` e `PUT` |
| `201 Created` | Recurso criado | `POST` |
| `204 No Content` | Deu certo, sem corpo na resposta | `DELETE` |
| `404 Not Found` | O cliente não foi encontrado | `GET`, `PUT` e `DELETE` com `id` inexistente |

---

## Comparação com o MongoDB

| Operação | MongoDB (mongosh) | API Spring Boot | Repository |
|---|---|---|---|
| Create | `insertOne()` | `POST /cliente` | `save()` |
| Read (todos) | `find()` | `GET /cliente` | `findAll()` |
| Read (por campo) | `find({ "nome": "José" })` | `GET /cliente?nome=José` | `findByNome()` |
| Read (por id) | `find({ "_id": ... })` | `GET /cliente/{id}` | `findById()` |
| Update | `replaceOne()` | `PUT /cliente/{id}` | `save()` (com `id` existente) |
| Delete | `deleteOne()` | `DELETE /cliente/{id}` | `deleteById()` |

A diferença principal é onde os dados ficam guardados. No MongoDB eles ficam no servidor do banco, e aqui ficam em uma lista na memória da aplicação. Como o Service e o Controller só conhecem o Repository, trocar a lista por um banco real muda apenas essa camada.

---

## Fluxo Completo

```
Postman -> Controller -> Service -> Repository (lista na memória)
```

1. O Postman envia a requisição para `/cliente`.
2. O **Controller** recebe e chama o **Service**.
3. O **Service** aplica a regra (por exemplo, conferir se o cliente existe) e chama o **Repository**.
4. O **Repository** guarda ou busca na lista.
5. A resposta volta pelo mesmo caminho, com o status HTTP correspondente.
