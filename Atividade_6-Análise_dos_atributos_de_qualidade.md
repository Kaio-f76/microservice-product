# Relatório de Análise de Qualidade de Software (Backend/Frontend)

## Resumo Geral

### Por que esta aplicação demonstra qualidade de software?

A aplicação demonstra qualidade principalmente pela arquitetura bem definida e organizada no backend, favorecendo **manutenibilidade**, **testabilidade** e **performance**.

* **Backend (TypeScript + Clean Architecture):** Utiliza separação clara entre **Controller**, **UseCase** e **Repository**, seguindo o princípio de Separação de Preocupações. O uso de TypeScript adiciona robustez.
* **Frontend (React/Vite + TypeScript):** Estrutura modular, com componentes reutilizáveis, boa gestão de estado e foco em simplicidade, performance e escalabilidade.

### Por que esta aplicação **não** demonstra qualidade de software?

Os pontos mais fracos são:

* **Testes limitados:** o backend possui testes de integração/uniários, mas nem todos os UseCases e Repositories estão cobertos.
* **Documentação incompleta:** READMEs básicos e ausência de Swagger/OpenAPI.
* **Acoplamento ao ORM:** TypeORM e decoradores específicos dificultam troca de tecnologia sem refatoração.

---

## 1. Manutenibilidade

### A arquitetura facilita manutenção futura?

Sim. O backend segue o padrão **Controller → UseCase → Repository**, isolando responsabilidades e favorecendo substituição de implementações.

**Exemplo de Controller:**
`backend/checkout/src/infra/http/HttpController.ts`

```typescript
httpServer.on("post", "/checkout", async (params, body, headers) => {
  body.token = headers.token;
  const checkout = usecaseFactory.createCheckout();
  const output = await checkout.execute(body);
  return output;
});
```

**Exemplo de Repository:**
`backend/auth/src/infra/repository/UserRepositoryDatabase.ts`

```typescript
async get(email: string): Promise<User> {
  const [userData] = await this.connection.query(
    "SELECT * FROM cccat11.user WHERE email = $1", [email]
  );
  return User.restore(
    userData.email, 
    userData.password, 
    userData.salt, 
    userData.password_type
  );
}
```

### O código é legível e padronizado?

Sim. A modularização e o uso de TypeScript garantem clareza, tipagem consistente e responsabilidade única por arquivo.

### Os arquivos seguem responsabilidade única?

Sim.
Exemplo: `backend/auth/src/domain/entity/Password.ts`

```typescript
export default interface Password {
  value: string;
  salt?: string;
  validate(password: string): boolean;
}
```

---

## 2. Testabilidade

### A arquitetura facilita criação de testes?

Sim. UseCases recebem dependências por construtor, permitindo substituição por mocks.

**Exemplo de UseCase:**
`backend/auth/src/application/usecase/Signup.ts`

```typescript
export default class Signup {
  constructor (
    readonly signupUserRepository: SignupUserRepository,
    readonly encrypter: Encrypter
  ) {}

  async execute(input: { email: string; password: string }): Promise<void> {
    const password = this.encrypter.encrypt(input.password);
    const user = new User(input.email, password);
    await this.signupUserRepository.save(user);
  }
}
```

### É fácil criar testes unitários?

Sim. A arquitetura facilita mocks de repositórios e serviços, mas **nem todos os casos de uso estão cobertos**, ponto negativo.

---

## 3. Escalabilidade

### A arquitetura suporta crescimento?

Sim. O backend é modular e novos módulos/casos de uso podem ser adicionados com baixo impacto.

**Exemplo:**
`backend/catalog/src/application/usecase/GetProduct.ts`

```typescript
import Presenter from "../../infra/presenter/Presenter";
import ProductRepository from "../repository/ProductRepository";
import RepositoryFactory from "../factory/RepositoryFactory";
import Product from "../../domain/entity/Product";

export default class GetProduct {
	productRepository: ProductRepository;

	constructor (repositoryFactory: RepositoryFactory) {
		this.productRepository = repositoryFactory.createProductRepository();
	}

	async execute (idProduct: number): Promise<Output> {
		const product = await this.productRepository.get(idProduct);
		return Object.assign(product, {
			volume: product.getVolume(),
			density: product.getDensity()
		});
	}
}

type Output = {
	idProduct: number,
	description: string,
	price: number,
	width: number,
	height: number,
	length: number,
	weight: number,
	volume: number,
	density: number
}
```

### O backend suporta escalabilidade horizontal?

Sim. O serviço é **stateless**, suporta múltiplas instâncias e utiliza paginação para reduzir carga no banco.

---

## 4. Reusabilidade

### Existem componentes reutilizáveis no frontend?

Sim.
Exemplo: `frontend/src/components/ui/button.jsx`

```tsx
/* eslint-disable react/prop-types */
import * as React from 'react'
import { cva } from 'class-variance-authority'
import { cn } from '../../lib/utils'

const buttonVariants = cva(
  'inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 ring-offset-background',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:opacity-90',
        secondary: 'bg-secondary text-secondary-foreground hover:opacity-90',
        outline: 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        destructive: 'bg-destructive text-destructive-foreground hover:opacity-90',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 rounded-md px-3',
        lg: 'h-11 rounded-md px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
)

const Button = React.forwardRef(({ className, variant, size, ...props }, ref) => {
  return (
    <button
      ref={ref}
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  )
})
Button.displayName = 'Button'

export default Button
```

### O backend possui elementos reutilizáveis?

Sim. DTOs e UseCases podem ser aplicados em múltiplas rotas.

**Exemplo:**
`backend/auth/src/application/usecase/Login.ts`

```typescript
import UserRepository from "../repository/UserRepository";
import TokenGenerator from "../../domain/entity/TokenGenerator";


export default class Login {

	constructor (readonly userRepository: UserRepository) {
	}

	async execute (input: Input): Promise<Output> {
		const user = await this.userRepository.get(input.email);
		if (user.validatePassword(input.password)) {
			const tokenGenerator = new TokenGenerator("secret");
			return {
				token: tokenGenerator.sign(user, input.date)
			};
		} else {
			throw new Error("Authentication failed");
		}
	}

}

type Input = {
	email: string,
	password: string,
	date: Date
}

type Output = {
	token: string
}
```

---

## 5. Portabilidade

### É fácil trocar tecnologias (DB, framework)?

Parcialmente. O uso de **interfaces de repositório** ajuda, mas decoradores TypeORM acoplam a implementação ao ORM.

**Exemplo:**
`backend/catalog/src/domain/entity/Product.ts`

```typescript
// Entity - Aggregate Root - Quanto menor melhor
export default class Product {

	constructor (readonly idProduct: number, readonly description: string, readonly price: number, readonly width: number, readonly height: number, readonly length: number, readonly weight: number) {
		if (width <= 0 || height <= 0 || length <= 0) throw new Error("Invalid dimensions");
		if (weight <= 0) throw new Error("Invalid weight");
	}

	getVolume (): number {
		const volume = this.width/100 * this.height/100 * this.length/100;
		return volume;
	}
	
	getDensity (): number {
		const density = this.weight/this.getVolume();
		return density;
	}
}
```

### A aplicação roda em outros ambientes?

Sim. Variáveis de ambiente e `package.json` facilitam configuração em Docker ou servidores distintos.

**Exemplo:**
`backend/checkout/package.json`

```json
{
  "name": "cccat11_refactoring",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "dependencies": {
    "@hapi/hapi": "^21.3.2",
    "@types/amqplib": "^0.10.1",
    "@types/cors": "^2.8.13",
    "@types/express": "^4.17.17",
    "@types/jest": "^29.5.0",
    "@types/sinon": "^10.0.14",
    "amqplib": "^0.10.3",
    "axios": "^1.3.5",
    "cors": "^2.8.5",
    "express": "^4.18.2",
    "jest": "^29.5.0",
    "nodemon": "^2.0.22",
    "pg-promise": "^11.4.3",
    "sinon": "^15.0.4",
    "ts-jest": "^29.1.0",
    "ts-node": "^10.9.1",
    "typescript": "^5.0.4"
  }
}
```

---

## 6. Performance

### Existem otimizações?

Sim.

* Backend com paginação usando: Arquitetura em camadas e Clean Architecture, Injeção de dependência via factory, Tipagem com TypeScript e Código limpo e legível.


  Exemplo: `backend/catalog/src/application/usecase/GetProduct.ts`

```typescript
import Presenter from "../../infra/presenter/Presenter";
import ProductRepository from "../repository/ProductRepository";
import RepositoryFactory from "../factory/RepositoryFactory";
import Product from "../../domain/entity/Product";

export default class GetProduct {
	productRepository: ProductRepository;

	constructor (repositoryFactory: RepositoryFactory) {
		this.productRepository = repositoryFactory.createProductRepository();
	}

	async execute (idProduct: number): Promise<Output> {
		const product = await this.productRepository.get(idProduct);
		return Object.assign(product, {
			volume: product.getVolume(),
			density: product.getDensity()
		});
	}
}

type Output = {
	idProduct: number,
	description: string,
	price: number,
	width: number,
	height: number,
	length: number,
	weight: number,
	volume: number,
	density: number
}
```

* Frontend com Vite otimiza carregamento e build; componentes reutilizáveis reduzem re-render desnecessário.

---

## 7. Segurança

### Os dados são validados antes de salvar?

Sim. validações garantem entradas corretas.

**Exemplo:**
`backend/auth/src/domain/entity/Email.ts`

```typescript
export default class Email {
	value: string

	constructor (email: string) {
		if (!this.isValid(email)) throw new Error("Invalid email");
		this.value = email;
	}

	isValid (email: string) {
		return String(email)
			.toLowerCase()
			.match(
			/^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
			);
	}
}
```

### Existem proteções básicas?

Sim.validações e tratamento de erros reduzem riscos de SQL Injection.

### A API trata erros corretamente?

não totalmente,O código não trata erros explicitamente.

- Por que:
    -> O async function chama verify.execute(body.token) e retorna output.

    -> Se verify.execute lançar um erro (por exemplo, token inválido ou problema interno), ele vai propagar a exceção e provavelmente quebrar o servidor ou retornar um erro genérico.

    -> Não há try/catch nem padronização da resposta de erro.

**Exemplo:**
`backend/auth/src/infra/http/HttpController.ts`

```typescript
import HttpServer from "./HttpServer";
import UsecaseFactory from "../factory/UsecaseFactory";

// interface adapter
export default class HttpController {

	constructor (httpServer: HttpServer, usecaseFactory: UsecaseFactory) {

		httpServer.on("post", "/verify", async function (params: any, body: any, headers: any) {
			const verify = usecaseFactory.createVerify();
			const output = await verify.execute(body.token);
			return output;
		});

	}
}
```

---

## 8. Documentação

### O código possui comentários úteis?

Parcialmente. Código legível, mas faltam comentários explicativos em pontos críticos.

### O projeto inclui README?

Ainda básico. Não há documentação Swagger/OpenAPI para rotas.

### As rotas são documentadas?

Não.

---

## Sugestões de Melhoria

1. **Implementar testes automatizados**

   * Unitários para UseCases
   * Integração para Controllers e Repositories

2. **Documentação completa**

   * Preencher READMEs
   * Adicionar Swagger/OpenAPI para rotas

3. **Abstração do ORM**

   * Criar interfaces de repositório
   * Fazer UseCases dependerem de abstrações, não diretamente do TypeORM

4. **Segurança**

   * Implementar autenticação e autorização (NestJS Guards)
   * Proteger rotas sensíveis (POST/PUT/DELETE)

5. **Melhoria de performance**

   * Indexação em banco
   * Cache de consultas frequentes
   * Debounce em requisições frontend


