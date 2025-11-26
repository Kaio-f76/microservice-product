# Exercício Prático: Análise de Qualidade de Software

## Disciplina: Qualidade de Software

---

## Objetivos de Aprendizagem

Ao final deste exercício, o aluno será capaz de:
- Identificar características de código de qualidade em aplicações reais
- Analisar arquiteturas de software e seus benefícios
- Compreender a importância de testes, documentação e boas práticas
- Implementar melhorias de performance em sistemas existentes

---

## Parte 1 – Análise da Aplicação

### Backend – Análise Inicial

#### 1. Linguagem de Programação
- **Linguagem:** TypeScript  
- **Vantagens:**  
  - Tipagem estática, garantindo maior confiabilidade na construção e definição de tipos  
  - Detecção de erros em tempo de compilação  
  - Melhor manutenção e legibilidade do código  

#### 2. Configuração e Execução
- Não utiliza arquivo `.env`.  
- **Como clonar o repositório:**
```bash
git clone https://github.com/leonardorsolar/microservice-product.git
cd microservice-product/backend
````

* **Instalar dependências:**

````bash
npm install
# ou
yarn install
````

* **Executar aplicação em desenvolvimento:**

````bash
npm run dev
# ou
yarn dev
````

#### 3. Arquitetura de Software

* **Padrão arquitetural:** Clean Architecture / arquitetura em camadas
* **Camadas:**

  * `domain` → regras de negócio, entidades e interfaces do repositório
  * `application` → casos de uso e lógica de aplicação
  * `infra` → implementação de repositórios, conexão com banco e detalhes de infraestrutura
* **Importância:** separação de preocupações facilita manutenção, testabilidade e escalabilidade
* **Endpoints da API:** `/api/products` (GET, POST, PUT, DELETE)
* **Inversão de dependências:** a camada `application` depende de interfaces do `domain` e não da implementação concreta do `infra`.

#### 4. Banco de Dados

* **Banco utilizado:** SQLite por padrão
* **Desacoplamento:** a lógica de negócio não depende do banco, apenas das interfaces de repositório
* **Migração de dados:** não implementada por padrão

#### 5. Funcionalidades

* Listagem de produtos
* Visualização de detalhes
* Criação de produtos
* Atualização de produtos
* Remoção de produtos
* Operações CRUD implementadas: **Create, Read, Update, Delete**

#### 6. Testes Automatizados

* **Tipos de testes:** unitários e integração (backend)
* **Executar testes:**

````bash
npm test
# ou
yarn test
````

* **Coverage:** pode ser gerado via `npm run test:coverage` ou `yarn test:coverage`

#### 7. Qualidade de Código – Linting

* **O que é:** linting verifica estilo de código, erros e inconsistências
* **Ferramenta:** ESLint
* **Como executar:**

````bash
npm run lint
# ou
yarn lint
````

#### 8. Pergunta Avançada – PostgreSQL

* **Para usar PostgreSQL:**

  1. Instalar driver `pg`
  2. Configurar conexão na camada `infra`
  3. Ajustar queries e tipos de dados
  4. Implementar migrações (TypeORM ou Knex)
  5. Configurar variáveis de ambiente para conexão segura
* **Princípio demonstrado:**

  * **Portabilidade** e **desacoplamento**

---

### Frontend – Análise Inicial

#### 1. Linguagem e Framework

* **Linguagem:** TypeScript
* **Framework:** React
* **Vantagens:**

  * Componentização e reatividade
  * Tipagem estática com TypeScript
  * Escalabilidade e manutenção facilitada
  * Ecossistema amplo de bibliotecas

#### 2. Configuração e Execução

* **Clonagem:**

````bash
git clone https://github.com/leonardorsolar/microservice-product.git
cd microservice-product/frontend
````

* **Instalar dependências:**

````bash
npm install
# ou
yarn install
````

* **Executar em desenvolvimento:**

````bash
npm run dev
# ou
yarn dev
````

* **Build para produção:**

````bash
npm run build
# ou
yarn build
````

#### 3. Arquitetura e Estrutura

* **Padrão:** Modular e componentizado
* **Pastas principais:**

  * `components/` → componentes reutilizáveis
  * `modules/` → módulos específicos da aplicação
  * `lib/` → funções/utilitários compartilhados

#### 4. Design UI/UX

* **Estratégia de design:** CSS puro, estilos locais ou CSS Modules
* **Responsividade:** flexbox, grid e media queries
* **Componentes reutilizáveis:** botões, inputs, cards, tabelas, pagination

#### 5. Integração com Backend

* **Comunicação:** HTTP requests (`fetch` ou Axios)
* **URLs da API:** configuradas em arquivos centrais (`lib/api.ts`)
* **Tratamento de erros:** mensagens amigáveis, try/catch, loading states

#### 6. Funcionalidades

* Listar produtos
* Visualizar detalhes
* Criar, atualizar e deletar produtos
* Paginação (quando implementada)
* **Gerenciamento de estado:** React Hooks (`useState`, `useEffect`), sincronização com backend, parâmetros de paginação na URL

#### 7. Testes

* **Tipos:** testes de componente, testes de comportamento/integrados, cobertura de código
* **Execução:**

````bash
npm test
# ou
yarn test
````

* **Cobertura:**

````bash
npm run test:coverage
# ou
yarn test:coverage
````

#### 8. Qualidade de Código

* **Ferramentas:** ESLint e Prettier
* **Execução:**

````bash
npm run lint
# ou
yarn lint
npm run lint:fix
# ou
yarn lint:fix
````

* **Padrões:** PascalCase para componentes, camelCase para variáveis, tipagem consistente, arquivos pequenos, módulos organizados

---

## Parte 2 – Implementação de Melhoria: Sistema de Paginação

### Backend – Paginação

#### 1. Modificações na API

* Endpoint `GET /api/products` atualizado para aceitar query parameters:

  * `page` (número da página, padrão: 1, mínimo: 1)
  * `limit` (itens por página, padrão: 10, valores permitidos: 10, 20, 50)
* Validação de parâmetros de entrada:

  * Valores inválidos retornam **erro 400**
* Resposta JSON com metadados de paginação:

````json
{
  "data": [...],
  "pagination": {
    "currentPage": 1,
    "totalPages": 10,
    "totalItems": 95,
    "itemsPerPage": 10,
    "hasNextPage": true,
    "hasPreviousPage": false
  }
}
````

#### 2. Implementação

* Modificação do **ProductRepositoryDatabase.ts** para incluir `LIMIT` e `OFFSET`
* `GetProducts` use case atualizado para receber `page` e `limit`
* Testes unitários e de integração adicionados

---

### Frontend – Paginação

#### 1. Componente `<Pagination />`

* Localização: `components/ui/Pagination.tsx`
* Funcionalidades:

  * Botões "Anterior" e "Próxima"
  * Numeração de páginas com elipses
  * Seletor de itens por página (10, 20, 50)
  * Exibe informações: "Mostrando X-Y de Z produtos"

#### 2. Gerenciamento de estado

* React Hooks (`useState`, `useEffect`) ou `useReducer`
* Estado sincronizado com **URL**
* Loading states e mensagens de erro amigáveis

#### 3. Testes

* Testes de componente e comportamento
* Coverage mantido ou aumentado

---

### Critérios de Aceitação

1. Exibe apenas os primeiros 10 produtos e controles de paginação quando há mais de 10 itens
2. Navegação entre páginas funciona sem recarregar a aplicação inteira
3. Informações de paginação exibidas corretamente
4. Alterar limite de itens por página retorna à página 1 e atualiza listagem
5. Reload da página mantém o usuário na página atual
6. Mensagem de erro amigável em caso de falha
7. Todos os testes de paginação passam
8. Parâmetros inválidos retornam erro 400

---

### Checklist de Implementação

**Backend**

* [x] Modificar ProductRepository para suportar paginação
* [x] Implementar lógica de paginação em ProductRepositoryDatabase
* [x] Atualizar GetProducts use case
* [x] Adicionar validação de parâmetros
* [x] Criar testes unitários e de integração
* [x] Atualizar documentação da API

**Frontend**

* [x] Criar componente Pagination reutilizável
* [x] Atualizar módulo de produtos para usar paginação
* [x] Gerenciar estado e sincronizar com URL
* [x] Adicionar loading states e tratamento de erros
* [x] Criar testes de componente e comportamento
* [x] Garantir acessibilidade

**Qualidade**

* [x] Código passa no lint
* [x] Coverage mantido ou aumentado
* [x] README atualizado


---

## Parte 3 – Avaliação de Qualidade

### 1. Manutenibilidade

* Arquitetura em camadas facilita manutenção
* Código legível, modular e organizado

### 2. Testabilidade

* Camadas desacopladas permitem testes unitários e de integração
* Componentes frontend e backend testáveis separadamente

### 3. Escalabilidade

* Clean Architecture e modularidade permitem crescimento da aplicação
* Novas funcionalidades podem ser adicionadas com mínimo impacto

### 4. Reusabilidade

* Componentes e funções reutilizáveis
* Evita duplicação de código

### 5. Portabilidade

* Lógica desacoplada de banco e frameworks
* Fácil trocar SQLite por PostgreSQL ou outro banco

### 6. Performance

* Paginação reduz carregamento de todos os produtos
* Backend otimizado (`LIMIT`, `OFFSET`)
* Frontend com loading states e debounce

### 7. Segurança

* Validação de dados e tratamento de erros
* Mensagens amigáveis no frontend

### 8. Documentação

* Código comentado e bem estruturado
* README detalhado com instruções de instalação, execução e uso da API


