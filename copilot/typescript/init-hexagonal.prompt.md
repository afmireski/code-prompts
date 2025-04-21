# Prompt de Inicialização

## Contexto do Projeto

Esse é um projeto de API Rest em NodeJS, simples, que servirá de base para projetos mais complexos.

### Observações
- **O OBJETIVO DESSE PROMPT É ESPECIFICAMENTE ESTRUTURAR O PROJETO, NÃO IMPLEMENTAR NENHUMA REGRA DE NEGÓCIO.**
- **NÃO CRIE ARQUIVOS DESNECESSÁRIOS AS CONFIGURAÇÕES ESPECIFICADAS**
- **PRESTE ATENÇÃO AOS EXEMPLOS DA MELHOR FORMA POSSÍVEL, MAS, SEJA ADAPTATIVO.**
- **SE PRECISAR CRIAR UM ARQUIVO PARA CRIAR UM DIRETÓRIO, CRIE UM ARQUIVO `x.*` VAZIO, NÃO INVENTE LÓGICAS.**

## Estrutura do Projeto
- **Organização de Diretórios**:
    - `/src`: Código-fonte principal do projeto.
        - `main.ts`: **Arquivo principal do projeto.**
        - `/@types`: Diretório dos tipos do projeto.
        - `/services`: regras de negócio.
            - `/ports`: Interfaces dos adaptadores.
            - `/adapters`: Implementações dos adaptadores.
        - `/controllers`: handlers para as regras de negócio.
        - `/routes`: rotas da api.
            - `/validators`: schemas de validação do Zod.        
        - `/middlewares`: middlewares das rotas.
        - `/errors`: recursos do tratamento de erros.
        - `/models`: reflexo das tabelas do banco.        
    - `/__tests__`: Scripts de teste.
    - `/docs`: Documentação do projeto.
- **Nomenclaturas**:
    - Nomes de arquivos serão em letras minúsculas separados por hífen: `nome-do-arquivo`.
    - Identificar o tipo do arquivo na sua extensão: `*.service.ts`, `*.route.ts`, etc.
    - Usar snake case para nomes de variáveis.
    - Classes, métodos e funções, camel case.
- **Arquitetura**:
    - Seguir os princípios de Ports e Adapters.
     - `services` modulares e independentes.
     - Criar adaptadores para conexões com o banco, por exemplo.


## Configuração

- **Use as versões mais recentes, estáveis, de tudo que especificarei abaixo.**
- Usar Node.js na versão 22.14.0, com typescript.
    - O Typescript deve ser configurado para NÃO GERAR arquivos provenientes da compilação.
    - O NodeJS deve estar configurado para usar os módulos ES6.
- Usar Bun na versão necessária para suportar a versão do node e do typescript requeridas.
    - Configurar hot reload no projeto ao atualizar o código fonte.
- Configurar Eslint e Prettier para formatação.
    - Usar aspas simples em strings e ponto e vírgula (;) ao fim da linha.
    - Tabulação padrão de um tab.
    - Configurar o eslint para remover imports não utilizados quando executado, gosto do plugin no-unused-imports.
    - O ESLint também deve dar erro em variáveis não utilizadas.
        - Ignorar checagem para variáveis iniciando com underscore (_).
- Utilizar o [`mise-en-place`](https://mise.jdx.dev) para gerenciar versão do node e bun dentro do projeto.
- Configurar o Vitest como framework de testes.
- Configurar o [`ZOD`](https://zod.dev) como validador de schemas.
- Montar um ConfigService, singleton, para carregar a env dinamicamente e obter suas variáveis.

### Variáveis de Ambiente

```env
NODE_ENV= #local, development, production
```

## Exemplos

### Repositórios
- `Template de Servidor express`: `afmireski/express-server-starter`

### Mise Config File
```.toml
[tools]
node = "20.18"

[env]
_.file = '.env'
```

### Serviço de Configuração

```ts
import { config } from 'dotenv';
import { InternalError } from '../errors/internal.error';

config();

export class ConfigService {
  private static instance: ConfigService;

  private env;

  private constructor() {
    switch (process.env.NODE_ENV) {
      case 'production':
        config({ path: '.env.production' });
      case 'development':
        config({ path: '.env.development' });
      case 'local':
      default:
        config({ path: '.env' });
    }

    this.env = process.env;
  }

  static create(): ConfigService {
    if (!ConfigService.instance) {
      ConfigService.instance = new ConfigService();
    }
    return ConfigService.instance;
  }

  get(key: string): any {
    const value = this.env[key];
    try {
      if (!value) {
        throw new InternalError(0, [`Missing environment variable: ${key}`]);
      }

      return JSON.parse(value);
    } catch (error) {
      return value; // Unnecessary parse
    }
  }
}

```


### Scripts do package.json
```json
"scripts": {
  "start:dev": "bun main.ts",
  "test": "vitest run",
  "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
  "migrate": "knex migrate:latest",
  "rollback": "knex migrate:rollback"
}
```