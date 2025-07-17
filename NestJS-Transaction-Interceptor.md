# 🚀 NestJS Transaction Interceptor (TypeORM)

A plug-and-play solution for automatically managing per-request database transactions in NestJS using TypeORM and `EntityManager`.

---

## 📌 Why Use This?

Manually wrapping database logic in transactions is tedious and error-prone. This interceptor pattern:

- ✅ Automatically starts and manages a transaction per request.
- ✅ Commits on success, rolls back on failure.
- ✅ Injects `EntityManager` via a decorator for DRY service calls.
- ✅ Ensures consistency and atomicity across service layers.

---

## 📦 Installation

Make sure you have the required packages installed:

```ts
npm install @nestjs/common typeorm rxjs
```

---

## 🧱 Implementation

### `transaction.interceptor.ts`

```ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { DataSource, EntityManager } from 'typeorm';
import { Observable, catchError, finalize, tap } from 'rxjs';

@Injectable()
export class TransactionInterceptor implements NestInterceptor {
  constructor(private readonly dataSource: DataSource) {}

  async intercept(context: ExecutionContext, next: CallHandler): Promise<Observable<any>> {
    const req = context.switchToHttp().getRequest();

    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();

    req.manager = queryRunner.manager;

    return next.handle().pipe(
      tap(async () => {
        await queryRunner.commitTransaction();
      }),
      catchError(async (err) => {
        await queryRunner.rollbackTransaction();
        throw err;
      }),
      finalize(async () => {
        await queryRunner.release();
      }),
    );
  }
}
```

### `transactional.decorator.ts`

```ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { EntityManager } from 'typeorm';

export const Transactional = createParamDecorator(
  (_: unknown, ctx: ExecutionContext): EntityManager => {
    const req = ctx.switchToHttp().getRequest();
    return req.manager as EntityManager;
  },
);
```

### Controller Usage
```ts
import { Controller, Post, Body, UseInterceptors } from '@nestjs/common';
import { TransactionInterceptor } from '../transaction/transaction.interceptor';
import { Transactional } from '../transaction/transactional.decorator';
import { EntityManager } from 'typeorm';
import { UserService } from './user.service';
import { ProfileService } from '../profile/profile.service';

@Controller('users')
export class UserController {
  constructor(
    private readonly userService: UserService,
    private readonly profileService: ProfileService,
  ) {}

  @Post()
  @UseInterceptors(TransactionInterceptor)
  async createUser(
    @Body() dto: any,
    @Transactional() manager: EntityManager,
  ) {
    await this.userService.create(dto.user, manager);
    await this.profileService.create(dto.profile, manager);
    return { message: 'User and Profile created successfully.' };
  }
}
```

### Service Usage (Need modification)
import { Injectable } from '@nestjs/common';
import { EntityManager } from 'typeorm';
import { User } from './user.entity';
```ts
@Injectable()
export class UserService {
  async create(data: Partial<User>, manager: EntityManager) {
    const user = manager.create(User, data);
    return await manager.save(user);
  }
}
```

### Optional: Global Interceptor Registration

## ✅ Benefits
| Feature       | Description                                              |
| ------------- | -------------------------------------------------------- |
| 🔁 DRY        | No repetitive `transaction()` boilerplate code needed    |
| 🎯 Atomicity  | All service calls share the same `EntityManager` context |
| 🧩 Composable | Easy to plug in logging, metrics, tracing, etc.          |
| ⚡ Performance | Uses a single transaction per request for efficiency     |



---

MIT © Hasnain Nisan
