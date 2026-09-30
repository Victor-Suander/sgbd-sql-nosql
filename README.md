# Seminário — SGBD SQL vs NoSQL

## Tema

**SGBD SQL vs NoSQL: modelos de dados, características, aplicações e diferenças**

### Objetivo

Apresentar as principais características dos bancos de dados relacionais e NoSQL, demonstrando suas diferenças de modelagem, consultas, transações, escalabilidade e aplicações práticas.

---

# 1. Introdução

Hoje praticamente todos os sistemas de software trabalham com dados.

Um aplicativo de banco precisa armazenar:

- clientes;
- contas;
- transferências;
- pagamentos;
- transações.

Um e-commerce precisa armazenar:

- produtos;
- clientes;
- pedidos;
- pagamentos;
- avaliações;
- estoque.

Uma rede social precisa armazenar:

- usuários;
- publicações;
- comentários;
- curtidas;
- seguidores;
- relacionamentos.

Para armazenar, organizar, consultar e proteger esses dados utilizamos um **SGBD — Sistema de Gerenciamento de Banco de Dados**.

Existem diferentes modelos de SGBD.

Dois dos principais grupos são:

**SQL / relacionais**

e

**NoSQL / não relacionais.**

---

# 2. O que é um SGBD?

SGBD significa **Sistema de Gerenciamento de Banco de Dados**.

É o software responsável por permitir que aplicações armazenem, consultem, alterem e removam informações de maneira controlada.

Exemplos:

### Relacionais

- MySQL
- PostgreSQL
- Oracle Database
- Microsoft SQL Server

### NoSQL

- MongoDB
- Redis
- Apache Cassandra
- Neo4j

O SGBD fica entre a aplicação e os dados.

```text
             APLICAÇÃO
                 │
                 ↓
                SGBD
                 │
                 ↓
              DADOS
```

Por exemplo:

```text
Aplicação Java
      │
      ↓
   MySQL
      │
      ↓
Banco de dados
```

---

# 3. O que significa SQL?

SQL significa:

**Structured Query Language**

ou

**Linguagem de Consulta Estruturada.**

É uma linguagem utilizada principalmente para trabalhar com bancos de dados relacionais.

O conceito central é organizar os dados em **tabelas relacionadas entre si**.

Exemplo:

```text
CLIENTE
--------------------------------
id | nome       | telefone
--------------------------------
1  | João       | 99999-1111
2  | Maria      | 99999-2222
```

Outra tabela:

```text
PEDIDO
--------------------------------
id | cliente_id | valor
--------------------------------
10 | 1          | 150.00
11 | 2          | 80.00
```

Nesse exemplo:

```text
CLIENTE
   │
   │ 1:N
   ↓
PEDIDO
```

Um cliente pode possuir vários pedidos.

---

# 4. Principais características do SQL

Os bancos relacionais normalmente trabalham com:

- tabelas;
- linhas;
- colunas;
- chave primária;
- chave estrangeira;
- relacionamentos;
- restrições;
- esquema definido;
- SQL;
- transações.

Um dos pontos importantes é a **integridade referencial**.

No MySQL, por exemplo, uma `FOREIGN KEY` permite relacionar dados entre tabelas e ajuda o SGBD a manter a consistência desses relacionamentos.

---

# 5. Chave primária

A chave primária identifica de maneira única um registro.

Exemplo:

```sql
CREATE TABLE cliente (
    id INT PRIMARY KEY,
    nome VARCHAR(100),
    telefone VARCHAR(20)
);
```

Nesse caso:

```text
id
↓
1 → João
2 → Maria
3 → Carlos
```

O `id` não deve identificar dois clientes diferentes.

---

# 6. Chave estrangeira

Agora podemos relacionar tabelas.

```sql
CREATE TABLE pedido (
    id INT PRIMARY KEY,
    cliente_id INT NOT NULL,

    FOREIGN KEY (cliente_id)
        REFERENCES cliente(id)
);
```

Aqui:

```text
cliente.id
     ↑
     │
cliente_id
     │
     ↓
pedido
```

Isso permite representar relacionamentos entre entidades.

---

# 7. Exemplo real — sistema de açougue

Vamos utilizar um sistema de pedidos de um açougue.

Precisamos armazenar:

```text
CLIENTE
PRODUTO
PEDIDO
ITEM_PEDIDO
```

Modelo:

```text
CLIENTE
   │
   │ 1:N
   ↓
PEDIDO
   │
   │ 1:N
   ↓
ITEM_PEDIDO
   │
   │ N:1
   ↓
PRODUTO
```

Essa modelagem representa:

> Um cliente pode realizar vários pedidos.

> Um pedido pode possuir vários produtos.

> Um produto pode aparecer em vários pedidos.

---

# 8. Criando o banco SQL

```sql
CREATE DATABASE acougue;

USE acougue;
```

Criando cliente:

```sql
CREATE TABLE cliente (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    telefone VARCHAR(20)
);
```

Produto:

```sql
CREATE TABLE produto (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2) NOT NULL,
    estoque INT NOT NULL
);
```

Pedido:

```sql
CREATE TABLE pedido (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT NOT NULL,
    data_pedido DATETIME DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (cliente_id)
        REFERENCES cliente(id)
);
```

Item do pedido:

```sql
CREATE TABLE item_pedido (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT NOT NULL,
    produto_id INT NOT NULL,
    quantidade INT NOT NULL,

    FOREIGN KEY (pedido_id)
        REFERENCES pedido(id),

    FOREIGN KEY (produto_id)
        REFERENCES produto(id)
);
```

Esse é um exemplo real de modelagem relacional.

O MySQL utiliza o InnoDB como mecanismo de armazenamento padrão no MySQL 8.4, e o InnoDB oferece recursos transacionais, recuperação após falhas e bloqueio em nível de linha.

---

# 9. Inserindo dados

```sql
INSERT INTO cliente (nome, telefone)
VALUES
('João Silva', '11999991111'),
('Maria Souza', '11999992222');
```

Produtos:

```sql
INSERT INTO produto (nome, preco, estoque)
VALUES
('Picanha', 79.90, 20),
('Alcatra', 59.90, 30),
('Linguiça', 29.90, 50);
```

---

# 10. Consultando dados

```sql
SELECT *
FROM cliente;
```

Resultado:

```text
1 | João Silva
2 | Maria Souza
```

Podemos filtrar:

```sql
SELECT *
FROM produto
WHERE preco > 50;
```

---

# 11. JOIN

Agora chegamos a uma das características importantes dos bancos relacionais.

Queremos descobrir:

> Qual cliente fez cada pedido?

Podemos utilizar:

```sql
SELECT
    cliente.nome,
    pedido.id,
    pedido.data_pedido
FROM cliente
INNER JOIN pedido
    ON pedido.cliente_id = cliente.id;
```

O banco relaciona as tabelas utilizando os valores correspondentes.

```text
CLIENTE
   │
   │ JOIN
   ↓
PEDIDO
```

---

# 12. Transações

Imagine uma compra.

O cliente compra:

```text
2 kg de picanha
```

Precisamos:

1. registrar o pedido;
2. registrar o item;
3. diminuir o estoque.

Essas operações estão relacionadas.

Podemos utilizar uma transação:

```sql
START TRANSACTION;

INSERT INTO pedido (cliente_id)
VALUES (1);

INSERT INTO item_pedido
    (pedido_id, produto_id, quantidade)
VALUES
    (1, 1, 2);

UPDATE produto
SET estoque = estoque - 2
WHERE id = 1;

COMMIT;
```

Se houver um problema:

```sql
ROLLBACK;
```

O objetivo é evitar que o sistema fique em um estado parcialmente atualizado.

---

# 13. O que é ACID?

ACID representa quatro propriedades:

### A — Atomicidade

A transação acontece completamente ou não acontece.

```text
Pedido criado
       +
Estoque atualizado
       +
Item registrado
```

Ou tudo funciona:

```text
COMMIT
```

Ou desfazemos:

```text
ROLLBACK
```

### C — Consistência

O banco deve respeitar suas regras e restrições.

Por exemplo:

```text
pedido.cliente_id
```

não deveria apontar para um cliente inexistente quando uma chave estrangeira está sendo usada.

### I — Isolamento

Transações concorrentes não devem interferir indevidamente umas nas outras.

O InnoDB oferece diferentes níveis de isolamento, incluindo `READ COMMITTED`, `REPEATABLE READ` e `SERIALIZABLE`.

### D — Durabilidade

Depois que uma transação é confirmada:

```sql
COMMIT;
```

os dados devem permanecer persistidos mesmo diante de uma falha, dentro das garantias do sistema.

---

# 14. O que é NoSQL?

NoSQL normalmente é utilizado para designar bancos que não seguem o modelo relacional tradicional.

O termo não deve ser entendido simplesmente como:

> "Banco que não utiliza SQL."

A ideia é mais ampla.

NoSQL engloba diferentes modelos de armazenamento.

Entre eles:

- documentos;
- chave-valor;
- colunar;
- grafos.

---

# 15. Tipos de NoSQL

| Modelo | Exemplo | Representação |
|---|---|---|
| Documento | MongoDB | JSON/BSON |
| Chave-valor | Redis | chave → valor |
| Colunar | Cassandra | famílias de colunas |
| Grafo | Neo4j | nós + relacionamentos |

O Apache Cassandra, por exemplo, é um banco NoSQL distribuído baseado em um modelo de armazenamento de colunas largas.

O Neo4j utiliza o modelo de grafos, trabalhando com nós e relacionamentos.

---

# 16. MongoDB

Para nosso exemplo, vamos utilizar o MongoDB.

O MongoDB é um banco orientado a documentos.

Em vez de:

```text
TABELA
   ↓
LINHAS
   ↓
COLUNAS
```

podemos trabalhar com:

```text
COLEÇÃO
   ↓
DOCUMENTOS
   ↓
CAMPOS
```

Os documentos são armazenados em BSON, uma representação binária de documentos semelhantes a JSON.

---

# 17. O mesmo pedido no MongoDB

Podemos representar um pedido assim:

```javascript
db.pedidos.insertOne({
    cliente: {
        id: 1,
        nome: "João Silva"
    },

    data: new Date(),

    itens: [
        {
            produto: "Picanha",
            quantidade: 2,
            preco: 79.90
        },
        {
            produto: "Linguiça",
            quantidade: 1,
            preco: 29.90
        }
    ],

    pagamento: {
        metodo: "PIX",
        status: "aprovado"
    }
});
```

Observe a diferença.

No SQL:

```text
cliente
pedido
item_pedido
produto
```

No MongoDB:

```text
pedido
 ├── cliente
 ├── itens[]
 │    ├── produto
 │    ├── quantidade
 │    └── preço
 └── pagamento
```

---

# 18. Por que isso é interessante?

Imagine que um produto possua características diferentes.

Produto 1:

```javascript
{
    nome: "Picanha",
    peso: 1.2,
    origem: "Brasil"
}
```

Produto 2:

```javascript
{
    nome: "Linguiça artesanal",
    peso: 0.8,
    sabor: "Apimentada",
    fabricante: "Casa do Churrasco"
}
```

O modelo de documentos permite que documentos de uma mesma coleção tenham estruturas diferentes. O MongoDB chama isso de modelo de esquema flexível.

Isso não significa ausência completa de regras: o MongoDB também oferece **schema validation** quando a aplicação precisa impor tipos ou restrições específicas.

---

# 19. Consultando MongoDB

Podemos buscar pedidos:

```javascript
db.pedidos.find({
    "cliente.nome": "João Silva"
});
```

Buscar pedidos que possuem picanha:

```javascript
db.pedidos.find({
    "itens.produto": "Picanha"
});
```

Atualizar:

```javascript
db.pedidos.updateOne(
    { "cliente.id": 1 },
    {
        $set: {
            "pagamento.status": "confirmado"
        }
    }
);
```

Excluir:

```javascript
db.pedidos.deleteOne({
    _id: ObjectId("...")
});
```

---

# 20. SQL × NoSQL — diferença visual

### SQL

```text
CLIENTE
┌──────────────┐
│ id           │
│ nome         │
│ telefone     │
└──────┬───────┘
       │
       ↓
PEDIDO
┌──────────────┐
│ id           │
│ cliente_id   │
│ data         │
└──────┬───────┘
       │
       ↓
ITEM_PEDIDO
```

### NoSQL

```text
PEDIDO
┌─────────────────────────────┐
│ cliente                     │
│  ├── id                     │
│  └── nome                   │
│                             │
│ itens                       │
│  ├── produto                │
│  ├── quantidade             │
│  └── preço                  │
│                             │
│ pagamento                   │
│  ├── método                 │
│  └── status                 │
└─────────────────────────────┘
```

---

# 21. Schema: rígido × flexível

No modelo relacional, normalmente definimos a estrutura das tabelas antecipadamente:

```sql
CREATE TABLE cliente (
    id INT,
    nome VARCHAR(100),
    email VARCHAR(150)
);
```

Existe uma estrutura definida.

No modelo documental:

```javascript
{
    nome: "João",
    email: "joao@email.com"
}
```

Outro documento pode possuir:

```javascript
{
    nome: "Maria",
    email: "maria@email.com",
    telefone: "11999999999"
}
```

Essa flexibilidade é uma característica importante de bancos documentais.

---

# 22. Normalização × desnormalização

No SQL, é comum separar informações para evitar redundância.

Exemplo:

```text
CLIENTE
   ↓
PEDIDO
   ↓
ITEM
   ↓
PRODUTO
```

Isso pode exigir `JOINs`.

No NoSQL, dependendo do padrão de acesso, podemos armazenar juntos dados que são consultados juntos.

```text
PEDIDO
 ├── cliente
 ├── produtos
 └── pagamento
```

O próprio MongoDB recomenda projetar o modelo de acordo com os padrões de acesso da aplicação e considera a possibilidade de incorporar dados relacionados.

---

# 23. Escalabilidade

Outro ponto importante é a escalabilidade.

## Escalabilidade vertical

Aumentamos os recursos de uma máquina:

```text
        SERVIDOR
       ┌─────────┐
       │ CPU     │
       │ RAM     │
       │ SSD     │
       └─────────┘
           ↑
      mais recursos
```

## Escalabilidade horizontal

Adicionamos máquinas:

```text
             APLICAÇÃO
                 │
        ┌────────┼────────┐
        ↓        ↓        ↓
     SERVIDOR SERVIDOR SERVIDOR
        01       02       03
```

Alguns sistemas NoSQL foram projetados especificamente para ambientes distribuídos e grandes volumes de dados.

O Cassandra, por exemplo, é documentado como um banco NoSQL distribuído, voltado a grandes volumes e cargas de consulta.

---

# 24. Um erro comum: "NoSQL não suporta ACID"

Essa afirmação está incorreta.

O MongoDB, por exemplo, suporta transações ACID, inclusive transações envolvendo múltiplos documentos, coleções e shards.

Portanto:

```text
NoSQL ≠ ausência de transações
```

A diferença está no **modelo de dados e na arquitetura**, e não simplesmente na existência ou ausência de transações.

---

# 25. Comparação

| Característica | SQL | NoSQL |
|---|---|---|
| Modelo | Relacional | Não relacional |
| Estrutura | Tabelas | Documentos, chave-valor, grafos etc. |
| Schema | Geralmente definido | Frequentemente flexível |
| Relacionamentos | Muito fortes | Dependem do modelo |
| JOIN | Recurso central | Geralmente menos central em documentos |
| Transações | Muito utilizadas | Também podem existir |
| ACID | Comum | Depende do SGBD, alguns suportam ACID |
| Escalabilidade | Vertical e também horizontal dependendo do SGBD | Frequentemente projetada para distribuição horizontal |
| Flexibilidade | Menor na estrutura | Maior em modelos documentais |
| Consultas | SQL | Depende do SGBD |
| Exemplos | MySQL, PostgreSQL | MongoDB, Cassandra, Redis, Neo4j |

Essa tabela é uma **comparação de características**, não uma classificação de qual tecnologia é superior.

---

# 26. Quando SQL pode fazer sentido?

Imagine um sistema bancário.

Temos:

```text
CLIENTE
CONTA
TRANSAÇÃO
TRANSFERÊNCIA
PAGAMENTO
```

Existem muitos relacionamentos e regras de integridade.

Uma transferência pode envolver:

```text
Conta A
   ↓
- R$ 500
   ↓
Conta B
   ↓
+ R$ 500
```

Precisamos garantir que as operações relacionadas sejam tratadas corretamente.

Um banco relacional com transações é uma possibilidade natural para esse tipo de requisito.

---

# 27. Quando NoSQL pode fazer sentido?

Agora imagine uma plataforma que recebe milhões de eventos:

```text
usuário abriu página
usuário clicou
usuário pesquisou
usuário assistiu
usuário adicionou produto
usuário removeu produto
```

Os eventos podem possuir estruturas diferentes.

```text
EVENTO 1
{
    usuario: 10,
    tipo: "clique",
    pagina: "/produtos"
}
```

Outro:

```text
EVENTO 2
{
    usuario: 10,
    tipo: "compra",
    produto: 50,
    valor: 199.90,
    pagamento: "pix"
}
```

Um modelo documental ou outro NoSQL pode ser adequado dependendo dos requisitos de armazenamento, consulta, distribuição e consistência.

---

# 28. Banco de grafos

Existe ainda um caso em que tabelas não são necessariamente o modelo mais natural.

Imagine uma rede social:

```text
        João
       /    \
   segue    segue
     ↓        ↓
   Maria    Carlos
     ↓
   segue
     ↓
   Pedro
```

Aqui o principal elemento são os **relacionamentos**.

É justamente o tipo de problema para o qual bancos de grafos são projetados.

O Neo4j trabalha com nós, relacionamentos e propriedades e possui a linguagem declarativa Cypher para consultar grafos.

Exemplo:

```cypher
MATCH (p:Pessoa)-[:SEGUE]->(outra:Pessoa)
RETURN p, outra;
```

---

# 29. Arquitetura híbrida

Um sistema moderno não necessariamente precisa escolher apenas um banco.

Podemos ter:

```text
                 APLICAÇÃO
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
        MySQL                MongoDB
          │                     │
          ↓                     ↓
   Dados transacionais    Documentos/eventos
```

Por exemplo:

### MySQL

```text
Clientes
Pedidos
Pagamentos
Estoque
```

### MongoDB

```text
Logs
Eventos
Catálogo flexível
Dados de sessão
```

Essa abordagem é chamada de **polyglot persistence**: utilizar diferentes tecnologias de armazenamento conforme as necessidades dos diferentes tipos de dados.

---

# 30. Estudo de caso — açougue

Vamos imaginar que estamos desenvolvendo um sistema para um açougue.

## Requisitos

O sistema precisa:

- cadastrar clientes;
- cadastrar produtos;
- controlar estoque;
- registrar pedidos;
- registrar pagamentos;
- armazenar informações do pedido;
- gerar relatórios.

### Possível arquitetura

```text
                 SISTEMA
                    │
        ┌───────────┴───────────┐
        ↓                       ↓
      MySQL                  MongoDB
        │                       │
        ↓                       ↓
   Dados críticos          Eventos/logs
   e relacionamentos       e informações
```

---

# 31. Exemplo de fluxo

Cliente:

```text
João
```

Compra:

```text
2 kg de Picanha
1 kg de Linguiça
```

Pagamento:

```text
PIX
```

No SQL:

```text
CLIENTE
   ↓
PEDIDO
   ↓
ITEM_PEDIDO
   ↓
PRODUTO
```

No MongoDB:

```text
PEDIDO
 ├── cliente
 ├── itens[]
 └── pagamento
```

A escolha depende de **como o sistema precisa armazenar, consultar, atualizar e escalar esses dados**.

---

# 32. Demonstração prática para apresentar

Se você tiver um computador durante o seminário, essa seria uma demonstração interessante.

## Parte 1 — MySQL

Execute:

```sql
SELECT
    c.nome,
    p.id AS pedido
FROM cliente c
INNER JOIN pedido p
    ON p.cliente_id = c.id;
```

Explique:

> "Aqui estou relacionando duas tabelas através da chave estrangeira."

Depois mostre:

```sql
START TRANSACTION;

UPDATE produto
SET estoque = estoque - 1
WHERE id = 1;

COMMIT;
```

Explique:

> "Aqui estou utilizando uma transação para controlar uma alteração relacionada ao estoque."

---

## Parte 2 — MongoDB

Mostre:

```javascript
db.pedidos.find({
    "cliente.nome": "João Silva"
});
```

Depois:

```javascript
db.pedidos.find({
    "itens.produto": "Picanha"
});
```

E explique:

> "Aqui os dados do pedido estão organizados dentro de um documento."

Essa comparação prática deixa muito mais evidente a diferença de modelagem.

---

# 33. Pergunta importante para a apresentação

Você pode colocar um slide apenas com:

# "Qual é melhor: SQL ou NoSQL?"

E deixar a pergunta no slide por alguns segundos.

Depois responder:

> **Não existe uma resposta universal.**

A decisão depende dos requisitos do sistema.

Devemos analisar:

```text
Volume de dados
      +
Estrutura dos dados
      +
Relacionamentos
      +
Consultas
      +
Consistência
      +
Transações
      +
Escalabilidade
      +
Disponibilidade
      +
Custo operacional
      ↓
ESCOLHA DO SGBD
```

---

# 34. Conclusão

SQL e NoSQL representam diferentes formas de modelar e gerenciar dados.

O SQL trabalha tradicionalmente com:

```text
Tabelas
Relacionamentos
Chaves
SQL
Transações
Integridade
```

Enquanto o universo NoSQL apresenta diferentes modelos, como:

```text
Documentos
Chave-valor
Colunas largas
Grafos
```

Um ponto importante é que **NoSQL não significa simplesmente "melhor para muitos dados"**, assim como SQL não significa simplesmente "melhor para sistemas pequenos".

A tecnologia deve ser escolhida considerando os requisitos da aplicação.

Em determinados sistemas, um banco relacional pode ser a escolha adequada.

Em outros, um banco NoSQL pode atender melhor determinado padrão de acesso ou necessidade de distribuição.

E em sistemas complexos, **SQL e NoSQL podem coexistir**.

---

# 35. Frase final para o seminário

> **"A pergunta correta não é qual banco é melhor. A pergunta correta é: qual modelo de dados atende melhor aos requisitos do sistema que estamos construindo?"**

---

# 36. Perguntas que o professor pode fazer

### 1. SQL e MySQL são a mesma coisa?

**Resposta:**

Não.

SQL é uma linguagem.

MySQL é um SGBD que utiliza SQL.

```text
SQL
 ↓
linguagem

MySQL
 ↓
SGBD
```

### 2. MongoDB é NoSQL?

Sim.

MongoDB é um banco NoSQL orientado a documentos e utiliza documentos BSON semelhantes a JSON.

### 3. NoSQL não possui relacionamentos?

Depende do modelo.

Alguns NoSQL trabalham com estruturas diferentes dos relacionamentos tradicionais do modelo relacional.

Bancos de grafos, por exemplo, são justamente especializados em representar relacionamentos.

### 4. NoSQL não suporta transações?

Não é correto afirmar isso.

MongoDB suporta transações ACID, inclusive envolvendo múltiplos documentos e coleções.

### 5. SQL sempre escala verticalmente?

Não.

Bancos relacionais podem utilizar diferentes estratégias de escalabilidade, dependendo do SGBD e da arquitetura.

A ideia de que:

```text
SQL = vertical
NoSQL = horizontal
```

é uma simplificação.

### 6. Por que utilizar MongoDB em vez de MySQL?

Não existe uma resposta única.

Um dos motivos pode ser a necessidade de trabalhar com documentos de estrutura flexível e modelar os dados de acordo com os padrões de acesso da aplicação.

### 7. Podemos utilizar os dois?

Sim.

Uma arquitetura pode utilizar diferentes bancos para diferentes necessidades.

### 8. O que é ACID?

```text
A = Atomicidade
C = Consistência
I = Isolamento
D = Durabilidade
```

É um conjunto de propriedades para transações.

---

# 37. Referências técnicas

- MySQL 8.4 Reference Manual — InnoDB  
  https://dev.mysql.com/doc/refman/8.4/en/innodb-introduction.html

- MySQL 8.4 — Foreign Keys  
  https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html

- MySQL 8.4 — Transaction Isolation Levels  
  https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html

- MongoDB Documentation  
  https://www.mongodb.com/docs/

- MongoDB — Data Modeling  
  https://www.mongodb.com/docs/manual/data-modeling/

- MongoDB — Transactions  
  https://www.mongodb.com/docs/manual/core/transactions/

- Apache Cassandra Documentation  
  https://cassandra.apache.org/doc/stable/

- Apache Cassandra — Architecture Overview  
  https://cassandra.apache.org/doc/stable/cassandra/architecture/overview.html

- Neo4j Documentation  
  https://neo4j.com/docs/

- Neo4j — Cypher Manual  
  https://neo4j.com/docs/cypher-manual/current/introduction/

---
