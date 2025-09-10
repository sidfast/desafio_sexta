## 📚 Construindo uma API RESTful CRUD Completa

(O conteúdo abordado aqui é uma parte do que já foi visto na aula anterior, com o complemento dos demais métodos HTTP)

Agora, vamos aprofundar nossa API de produtos, implementando as operações completas de **CRUD (Create, Read, Update, Delete)**. Você aprenderá a lidar com requisições **POST**, **PUT** e **DELETE**, e a usar ferramentas como Postman ou Insomnia para testar sua API.

-----
### Objetivos da Aula

Ao final deste conteúdo, você será capaz de:

  * Implementar as operações **CREATE (POST)**, **UPDATE (PUT)** e **DELETE** para gerenciar recursos.
  * Utilizar o middleware `express.json()` para processar corpos de requisição JSON.
  * Gerenciar dados em memória para simular um banco de dados.
  * Testar sua API utilizando ferramentas como Postman ou Insomnia.

-----

### 1\. Relembrando o CRUD e Métodos HTTP

Vimos na Aula 02 que **CRUD** é um acrônimo para as operações básicas de qualquer sistema que gerencia dados. Agora, vamos mapeá-las com os métodos HTTP que usaremos:

  * **C**reate (Criar) ➡️ **`POST`**
  * **R**ead (Ler) ➡️ **`GET`** (Já implementamos na Aula 03)
  * **U**pdate (Atualizar) ➡️ **`PUT`** / `PATCH`
  * **D**elete (Deletar) ➡️ **`DELETE`**

### 2\. Preparação: Nosso "Banco de Dados" em Memória

Para simular o armazenamento de dados, continuaremos usando um array JavaScript. Vamos adicionar uma variável `nextId` para gerar IDs automaticamente para novos produtos.

**Modifique seu `app.js`**:

```javascript
// ... importações e configurações iniciais ...

// Nosso "banco de dados" em memória para simular dados.
// Use 'let' para 'produtos' e 'nextId' pois eles serão modificados.
let produtos = [
  { id: 1, nome: 'Teclado Mecânico', preco: 450.00 },
  { id: 2, nome: 'Mouse Gamer', preco: 150.00 },
  { id: 3, nome: 'Monitor UltraWide', preco: 1200.00 }
];
let nextId = 4; // Começa com o próximo ID disponível

// ... suas rotas GET existentes ...

// Inicia o servidor
app.listen(PORT, () => {
  console.log(`Servidor rodando em http://localhost:${PORT}`);
  console.log('Para parar o servidor, pressione Ctrl+C no terminal.');
});
```

### 3\. Habilitando o Processamento de JSON (`express.json()`)

Quando enviamos dados para o servidor via requisições `POST` ou `PUT`, eles geralmente vêm no corpo da requisição em formato **JSON**. Para que o Express consiga "ler" e parsear esse JSON para um objeto JavaScript (`req.body`), precisamos usar um **middleware** específico.

Adicione a seguinte linha no seu `app.js`, **antes de qualquer definição de rota**:

```javascript
const express = require('express');
const app = express();
const PORT = 3000;

// Middleware para processar requisições com corpo em JSON.
// Isso faz com que 'req.body' esteja disponível com o JSON parseado.
app.use(express.json());

// ... produtos e nextId ...
// ... suas rotas GET existentes ...

// ... app.listen() ...
```

### 4\. Operação CREATE: `POST /api/produtos`

Esta rota permitirá que clientes enviem dados de um novo produto para serem adicionados à nossa lista.

  * **Método HTTP:** `POST`
  * **Endpoint:** `/api/produtos`
  * **Corpo da Requisição (exemplo JSON):**
    ```json
    {
      "nome": "Webcam Full HD",
      "preco": 300.00
    }
    ```

**Adicione a rota `POST` ao seu `app.js`:**

```javascript
// ... após as rotas GET e antes do app.listen() ...

// Rota POST para criar um novo produto.
app.post('/api/produtos', (req, res) => {
  // O corpo da requisição (JSON) é acessado via req.body.
  const { nome, preco } = req.body;

  // Validação simples: verifica se nome e preco foram fornecidos.
  if (!nome || preco === undefined) {
    return res.status(400).json({ message: 'Nome e preço são obrigatórios.' });
  }

  const novoProduto = {
    id: nextId++, // Atribui um novo ID e incrementa nextId
    nome,        // Equivale a nome: nome
    preco        // Equivale a preco: preco
  };

  produtos.push(novoProduto); // Adiciona o novo produto ao array

  // Retorna o produto criado com status 201 (Created).
  res.status(201).json(novoProduto);
});

// ... app.listen() ...
```

#### Testando com Postman/Insomnia:

1.  **Inicie/Reinicie** seu servidor (`node app.js`).
2.  Abra o **Postman** ou **Insomnia**.
3.  Crie uma nova requisição:
      * **Método:** `POST`
      * **URL:** `http://localhost:3000/api/produtos`
      * Vá na aba **Body** (Corpo) e selecione **raw** (bruto) e **JSON**.
      * Cole o JSON de exemplo:
        ```json
        {
          "nome": "Webcam Full HD",
          "preco": 300.00
        }
        ```
      * Clique em **Send** (Enviar).
      * **Verifique a resposta:** Você deve receber o novo produto com um ID e o status `201 Created`.
4.  Faça um `GET` para `http://localhost:3000/api/produtos` para confirmar que o produto foi adicionado.

### 5\. Operação UPDATE: `PUT /api/produtos/:id`

Esta rota permitirá que clientes atualizem os dados de um produto existente.

  * **Método HTTP:** `PUT`
  * **Endpoint:** `/api/produtos/:id`
  * **Corpo da Requisição (exemplo JSON):**
    ```json
    {
      "nome": "Monitor UltraWide 34 Polegadas",
      "preco": 1350.00
    }
    ```

**Adicione a rota `PUT` ao seu `app.js`:**

```javascript
// ... após a rota POST e antes do app.listen() ...

// Rota PUT para atualizar um produto existente por ID.
app.put('/api/produtos/:id', (req, res) => {
  const id = parseInt(req.params.id);
  // Encontra o índice do produto no array.
  const produtoIndex = produtos.findIndex(p => p.id === id);

  // Verifica se o produto foi encontrado.
  if (produtoIndex !== -1) {
    const { nome, preco } = req.body;

    // Validação simples: Pelo menos um campo deve ser fornecido.
    if (!nome && preco === undefined) { // Permite que preco seja 0
      return res.status(400).json({ message: 'Pelo menos um campo (nome ou preco) deve ser fornecido para atualização.' });
    }

    // Atualiza o produto existente no array.
    // Usamos o spread operator (...) para manter as propriedades existentes
    // e sobrescrever apenas as que foram fornecidas no req.body.
    produtos[produtoIndex] = {
      ...produtos[produtoIndex], // Mantém o ID e outras propriedades se não forem fornecidas
      nome: nome !== undefined ? nome : produtos[produtoIndex].nome, // Atualiza nome se fornecido
      preco: preco !== undefined ? preco : produtos[produtoIndex].preco // Atualiza preco se fornecido
    };

    // Retorna o produto atualizado.
    res.json(produtos[produtoIndex]);
  } else {
    // Se não encontrou, retorna 404 (Not Found).
    res.status(404).json({ message: 'Produto não encontrado para atualização.' });
  }
});

// ... app.listen() ...
```

#### Testando com Postman/Insomnia:

1.  **Inicie/Reinicie** seu servidor.
2.  Faça um `GET` em `http://localhost:3000/api/produtos` para ver os IDs disponíveis. Escolha um ID existente (ex: `3`).
3.  Crie uma nova requisição:
      * **Método:** `PUT`
      * **URL:** `http://localhost:3000/api/produtos/3` (use o ID do produto que você quer atualizar)
      * **Body:** `raw` e `JSON`. Cole o JSON de exemplo:
        ```json
        {
          "nome": "Monitor UltraWide 34 Polegadas",
          "preco": 1350.00
        }
        ```
      * Clique em **Send**.
      * **Verifique a resposta:** Você deve receber o produto atualizado e o status `200 OK`.
4.  Faça um `GET` novamente em `http://localhost:3000/api/produtos/3` para confirmar a atualização.
5.  Teste com um ID que não existe (ex: `http://localhost:3000/api/produtos/99`) para ver o erro `404 Not Found`.

### 6\. Operação DELETE: `DELETE /api/produtos/:id`

Esta rota permitirá que clientes removam um produto existente.

  * **Método HTTP:** `DELETE`
  * **Endpoint:** `/api/produtos/:id`

**Adicione a rota `DELETE` ao seu `app.js`:**

```javascript
// ... após a rota PUT e antes do app.listen() ...

// Rota DELETE para excluir um produto por ID.
app.delete('/api/produtos/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const initialLength = produtos.length; // Guarda o tamanho original do array

  // Filtra o array, removendo o produto com o ID especificado.
  produtos = produtos.filter(p => p.id !== id);

  // Verifica se a quantidade de produtos diminuiu (se um produto foi removido).
  if (produtos.length < initialLength) {
    // Se o produto foi removido, retorna status 204 (No Content).
    res.status(204).send(); // send() sem corpo para 204
  } else {
    // Se o produto não foi encontrado para exclusão, retorna 404.
    res.status(404).json({ message: 'Produto não encontrado para exclusão.' });
  }
});

// ... app.listen() ...
```

#### Testando com Postman/Insomnia:

1.  **Inicie/Reinicie** seu servidor.
2.  Faça um `GET` em `http://localhost:3000/api/produtos` para ver os IDs disponíveis. Escolha um ID existente (ex: `1`).
3.  Crie uma nova requisição:
      * **Método:** `DELETE`
      * **URL:** `http://localhost:3000/api/produtos/1`
      * Clique em **Send**.
      * **Verifique a resposta:** Você deve receber o status `204 No Content` (e nenhum corpo na resposta).
4.  Faça um `GET` novamente em `http://localhost:3000/api/produtos` para confirmar que o produto foi removido.
5.  Teste com um ID que não existe para ver o erro `404 Not Found`.

-----

### 7\. Estado Atual Completo do Projeto (`app.js`)

Ao final desta aula, seu arquivo `app.js` deve estar com a API CRUD completa:

```javascript
const express = require('express');
const app = express();
const PORT = 3000;

// Middleware para processar requisições com corpo em JSON.
app.use(express.json());

// Nosso "banco de dados" em memória para simular dados.
let produtos = [
  { id: 1, nome: 'Teclado Mecânico', preco: 450.00 },
  { id: 2, nome: 'Mouse Gamer', preco: 150.00 },
  { id: 3, nome: 'Monitor UltraWide', preco: 1200.00 }
];
let nextId = 4; // Para gerar IDs únicos para novos produtos

// Rota GET para a URL raiz
app.get('/', (req, res) => {
  res.send('Bem-vindo à nossa primeira API Back-End com Express!');
});

// Rota GET para a URL '/sobre'
app.get('/sobre', (req, res) => {
  res.send('Esta é a página "Sobre" da nossa aplicação.');
});

// Rota GET para listar todos os produtos
app.get('/api/produtos', (req, res) => {
  res.json(produtos);
});

// Rota GET para buscar um produto específico por ID
app.get('/api/produtos/:id', (req, res) => {
  const idProduto = parseInt(req.params.id);
  const produtoEncontrado = produtos.find(p => p.id === idProduto);

  if (produtoEncontrado) {
    res.json(produtoEncontrado);
  } else {
    res.status(404).send('Produto não encontrado.');
  }
});

// Rota POST para criar um novo produto
app.post('/api/produtos', (req, res) => {
  const { nome, preco } = req.body;

  if (!nome || preco === undefined) {
    return res.status(400).json({ message: 'Nome e preço são obrigatórios.' });
  }

  const novoProduto = {
    id: nextId++,
    nome,
    preco
  };

  produtos.push(novoProduto);
  res.status(201).json(novoProduto);
});

// Rota PUT para atualizar um produto existente por ID
app.put('/api/produtos/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const produtoIndex = produtos.findIndex(p => p.id === id);

  if (produtoIndex !== -1) {
    const { nome, preco } = req.body;

    if (!nome && preco === undefined) {
      return res.status(400).json({ message: 'Pelo menos um campo (nome ou preco) deve ser fornecido para atualização.' });
    }

    produtos[produtoIndex] = {
      ...produtos[produtoIndex],
      nome: nome !== undefined ? nome : produtos[produtoIndex].nome,
      preco: preco !== undefined ? preco : produtos[produtoIndex].preco
    };

    res.json(produtos[produtoIndex]);
  } else {
    res.status(404).json({ message: 'Produto não encontrado para atualização.' });
  }
});

// Rota DELETE para excluir um produto por ID
app.delete('/api/produtos/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const initialLength = produtos.length;

  produtos = produtos.filter(p => p.id !== id);

  if (produtos.length < initialLength) {
    res.status(204).send();
  } else {
    res.status(404).json({ message: 'Produto não encontrado para exclusão.' });
  }
});

// Inicia o servidor
app.listen(PORT, () => {
  console.log(`Servidor rodando em http://localhost:${PORT}`);
  console.log('Para parar o servidor, pressione Ctrl+C no terminal.');
});
```

-----

### 8\. Aplicação nos Projetos

Com base nos modelos descritos, certifique-se de aplicar estes métodos básicos nos seus respectivos projetos, individualmente.

Ao final, é essencial registrar o progresso de suas implementações. Lembre-se de salvar seu trabalho e subir para o github:

```bash
git add .
git commit -m "feat: implementacao completa das operacoes CRUD na API do projeto..."
```

-----

### Conclusão e Próximos Passos

Parabéns\! Você construiu uma API RESTful completa com operações CRUD utilizando Node.js e Express. Agora você tem o conhecimento para criar serviços básicos que interagem com dados.

-----

**Próxima Aula de Conteúdo:** Em nossa próxima aula abordaremos a **análise de sistemas Server-Side e a concepção de micro e macro services**.
