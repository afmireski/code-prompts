# Prompt sobre Tratamento de Erros

## Contexto

Os erros em projetos NodeJS projeto serão tratados de forma customizada, usando uma classe especial, mensagens especiais e middlewares.

### Estrutura

- `/src/errors`: recursos do tratamento de erros, exceto middleware.
- `/src/middlewares`: local dos middlewares.


### Tratamento de Erros

#### Classe personalizada
```ts
import errors from './error-mapping/0-example.errors';

export type InternalMessage = {
  httpCode: number;
  message: string;
};

export class InternalError {
  httpCode: number;
  message: string;

  constructor(
    public code: number,
    private readonly details?: string[],
  ) {
    const handling = this.errorHandler();

    this.httpCode = handling?.httpCode;
    this.message = handling?.message;
  }

  private errorHandler(): InternalMessage {
    if (this.code === 0) {
      return {
        httpCode: 500,
        message: 'Um erro inesperado ocorreu',
      };
    } else if (this.code < 100) {
      return errors[this.code];
    }

    this.code = -1;
    return {
      httpCode: 500,
      message: 'Um erro não mapeado ocorreu',
    };
  }
}
```

#### Mensagens de erro:

```ts
import { InternalMessage } from '../internal.error';

const errors: Record<number, InternalMessage> = {};

export default errors;
```

#### Middleware
```ts
import { NextFunction, Request, Response } from 'express';
import { InternalError } from '../errors/internal.error';

export const internalErrorsMiddleware = (
  err: any,
  _request: Request,
  response: Response,
  _next: NextFunction,
): void => {
  if (err instanceof InternalError) {
    response.status(err.httpCode).json(err);
    return;
  } else if (err instanceof Error) {
    response.status(500).json(new InternalError(0, [err.message]));
    return;
  }
  response.status(500).json(new InternalError(-1, [JSON.stringify(err)]));
  return;
};
```

