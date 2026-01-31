---
name: nestjs-backend
description: Expert guidance for NestJS and scalable backend architecture. Use this skill when building NestJS applications, modules, controllers, services, or any backend TypeScript code. Triggers on API development, domain-driven module design, DTOs, validation, database repositories, guards, interceptors, error handling, queues, events, and production-ready backend patterns. Covers security, testing, performance, and maintainability best practices.
---

# NestJS Backend Development Skill

Build secure, testable, performant, and maintainable backend applications following NestJS best practices.

## TypeScript Guidelines (Backend)

- Enable strict type checking in `tsconfig.json`
- Prefer type inference when obvious
- Never use `any`; use `unknown` and narrow explicitly
- Use `readonly` for immutable data
- Prefer enums or union types over magic strings
- Never suppress errors with `@ts-ignore` without justification

```typescript
// Union types over magic strings
type OrderStatus = 'pending' | 'processing' | 'completed' | 'cancelled';

// Readonly for immutable data
interface Config {
  readonly apiKey: string;
  readonly maxRetries: number;
}

// Narrow unknown types explicitly
function processInput(data: unknown): ProcessedData {
  if (!isValidInput(data)) {
    throw new BadRequestException('Invalid input');
  }
  return transform(data);
}
```

## Architecture Principles

- Follow domain-driven module design
- Each module = one business domain
- Controllers are thin; services are focused
- No cross-module imports; use explicit exports
- Never place business logic in controllers

```
src/
├── users/
│   ├── users.module.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── users.repository.ts
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── user-response.dto.ts
│   └── entities/
│       └── user.entity.ts
├── orders/
│   ├── orders.module.ts
│   └── ...
└── config/
    ├── app.config.ts
    ├── database.config.ts
    └── auth.config.ts
```

## Modules

One module per domain. Export only what is required.

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { UsersRepository } from './users.repository';
import { User } from './entities/user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService], // Export only what other modules need
})
export class UsersModule {}
```

### Module Rules

- Do NOT create "shared" modules with business logic
- Use dynamic modules only when configuration truly varies
- Avoid circular dependencies between modules

## Controllers

Controllers handle HTTP concerns only—no business logic.

```typescript
import { Controller, Get, Post, Body, Param, HttpStatus, HttpCode } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UserResponseDto } from './dto/user-response.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(@Body() createUserDto: CreateUserDto): Promise<UserResponseDto> {
    return this.usersService.create(createUserDto);
  }

  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserResponseDto> {
    return this.usersService.findOne(id);
  }
}
```

### Controller Rules

- Validate inputs using DTOs
- Return explicit response DTOs
- Use proper HTTP status codes
- Never access database or repositories directly

## Services

One service = one responsibility. Services orchestrate business logic.

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { UsersRepository } from './users.repository';
import { CreateUserDto } from './dto/create-user.dto';
import { UserResponseDto } from './dto/user-response.dto';

@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  async create(dto: CreateUserDto): Promise<UserResponseDto> {
    const user = await this.usersRepository.create({
      ...dto,
      password: await this.hashPassword(dto.password),
    });
    return this.toResponseDto(user);
  }

  async findOne(id: string): Promise<UserResponseDto> {
    const user = await this.usersRepository.findById(id);
    if (!user) {
      throw new NotFoundException(`User ${id} not found`);
    }
    return this.toResponseDto(user);
  }

  private toResponseDto(user: User): UserResponseDto {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: user.createdAt,
    };
  }

  private async hashPassword(password: string): Promise<string> {
    // Hash implementation
  }
}
```

### Service Rules

- Prefer pure functions for complex logic
- Use `@Injectable()` with explicit scope when needed
- Avoid God services (>300–400 LOC)

## DTOs & Validation

All incoming requests MUST use DTOs with `class-validator` and `class-transformer`.

```typescript
import { IsEmail, IsString, MinLength, IsOptional } from 'class-validator';
import { Transform } from 'class-transformer';

export class CreateUserDto {
  @IsEmail()
  @Transform(({ value }) => value?.toLowerCase().trim())
  readonly email: string;

  @IsString()
  @MinLength(8)
  readonly password: string;

  @IsString()
  @MinLength(2)
  readonly name: string;

  @IsOptional()
  @IsString()
  readonly phone?: string;
}

// Response DTO - separate from request DTO
export class UserResponseDto {
  readonly id: string;
  readonly email: string;
  readonly name: string;
  readonly createdAt: Date;
  // Note: password is never included
}
```

### DTO Rules

- DTOs must be immutable (use `readonly`)
- DTOs represent API contracts only
- Do NOT reuse DTOs as database models
- Never expose sensitive fields in response DTOs

## Database & Repositories

Access database ONLY via repositories. Repositories contain persistence logic only.

```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';

@Injectable()
export class UsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repo: Repository<User>,
  ) {}

  async create(data: Partial<User>): Promise<User> {
    const user = this.repo.create(data);
    return this.repo.save(user);
  }

  async findById(id: string): Promise<User | null> {
    return this.repo.findOne({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.repo.findOne({ where: { email } });
  }

  async findAllPaginated(page: number, limit: number): Promise<[User[], number]> {
    return this.repo.findAndCount({
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });
  }
}
```

### Repository Rules

- Do NOT contain business rules
- Keep schemas/entities isolated from services
- Never leak ORM entities to controllers

## Configuration

Use `@nestjs/config` with typed, validated configuration.

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';
import { IsString, IsNumber, validateSync } from 'class-validator';
import { plainToInstance } from 'class-transformer';

class DatabaseEnv {
  @IsString()
  DB_HOST: string;

  @IsNumber()
  DB_PORT: number;

  @IsString()
  DB_NAME: string;
}

export default registerAs('database', () => {
  const config = {
    host: process.env.DB_HOST,
    port: parseInt(process.env.DB_PORT, 10) || 5432,
    name: process.env.DB_NAME,
  };

  // Validate
  const validatedConfig = plainToInstance(DatabaseEnv, process.env);
  const errors = validateSync(validatedConfig);
  if (errors.length > 0) {
    throw new Error(`Config validation error: ${errors}`);
  }

  return config;
});

// Usage in service
@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {
    const host = this.configService.get<string>('database.host');
  }
}
```

### Configuration Rules

- Centralize by domain: app, database, auth, cache
- Configuration must be typed and validated
- Do NOT access `process.env` directly outside config files

## Security

Enable global validation with whitelisting:

```typescript
// main.ts
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,              // Strip non-whitelisted properties
      forbidNonWhitelisted: true,   // Throw on non-whitelisted properties
      transform: true,              // Transform payloads to DTO instances
      transformOptions: {
        enableImplicitConversion: true,
      },
    }),
  );
  
  await app.listen(3000);
}
```

### Security Rules

- Use guards for authentication & authorization
- Use interceptors for response shaping
- Never trust client input
- Mask sensitive fields everywhere

## Guards, Pipes, Interceptors, Filters

Each serves one concern only.

```typescript
// Guard - Authorization
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<string[]>('roles', context.getHandler());
    if (!requiredRoles) return true;
    
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return requiredRoles.some(role => user.roles?.includes(role));
  }
}

// Interceptor - Response transformation
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}

// Exception Filter - Centralized error handling
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    response.status(status).json({
      success: false,
      statusCode: status,
      message: this.getMessage(exception),
      timestamp: new Date().toISOString(),
    });
  }
}
```

## Error Handling

Use custom exceptions extending `HttpException`. Never throw raw errors.

```typescript
export class UserNotFoundException extends NotFoundException {
  constructor(userId: string) {
    super({
      errorCode: 'USER_NOT_FOUND',
      message: `User with ID ${userId} not found`,
    });
  }
}

export class InvalidCredentialsException extends UnauthorizedException {
  constructor() {
    super({
      errorCode: 'INVALID_CREDENTIALS',
      message: 'Invalid email or password',
    });
  }
}
```

### Error Rules

- Centralize formatting using exception filters
- Avoid leaking internal error details
- Use consistent error response structure

## Async & Background Jobs

Offload heavy tasks to queues. Never block HTTP request lifecycle.

```typescript
import { Process, Processor } from '@nestjs/bull';
import { Job } from 'bull';

@Processor('email')
export class EmailProcessor {
  @Process('send-welcome')
  async handleWelcomeEmail(job: Job<{ userId: string; email: string }>) {
    const { userId, email } = job.data;
    // Send email - must be idempotent
    await this.emailService.sendWelcome(email);
  }
}

// Queue the job from service
@Injectable()
export class UsersService {
  constructor(@InjectQueue('email') private emailQueue: Queue) {}

  async create(dto: CreateUserDto): Promise<UserResponseDto> {
    const user = await this.usersRepository.create(dto);
    
    // Offload email to queue
    await this.emailQueue.add('send-welcome', {
      userId: user.id,
      email: user.email,
    });
    
    return this.toResponseDto(user);
  }
}
```

### Async Rules

- Ensure jobs are idempotent
- Use queues, workers, or cron jobs for heavy tasks

## Testing

Write unit tests for services, E2E tests for controllers.

```typescript
// users.service.spec.ts
describe('UsersService', () => {
  let service: UsersService;
  let repository: jest.Mocked<UsersRepository>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UsersRepository,
          useValue: {
            create: jest.fn(),
            findById: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get(UsersService);
    repository = module.get(UsersRepository);
  });

  describe('findOne', () => {
    it('should return user when found', async () => {
      const mockUser = { id: '1', email: 'test@example.com', name: 'Test' };
      repository.findById.mockResolvedValue(mockUser);

      const result = await service.findOne('1');

      expect(result).toEqual(expect.objectContaining({ id: '1' }));
    });

    it('should throw NotFoundException when user not found', async () => {
      repository.findById.mockResolvedValue(null);

      await expect(service.findOne('1')).rejects.toThrow(NotFoundException);
    });
  });
});
```

### Testing Rules

- Mock external services and databases
- Do NOT mock internal business logic
- Ensure tests are deterministic

## Performance

- Use caching where applicable
- Avoid N+1 database queries
- Paginate all list endpoints
- Use streaming for large payloads

## Quick Reference

| Layer | Responsibility | Avoid |
|-------|---------------|-------|
| Controller | HTTP concerns, validation | Business logic, DB access |
| Service | Business logic orchestration | >400 LOC, direct ORM |
| Repository | Persistence logic only | Business rules |
| DTO | API contract, validation | Reuse as DB model |
| Guard | Auth & authorization | Multiple concerns |
| Interceptor | Response shaping, logging | Business logic |
| Filter | Error handling | Multiple concerns |

## Anti-Patterns ❌

- Business logic in controllers
- Shared "utils" dumping ground
- God services (>400 LOC)
- Circular dependencies
- Direct database access in controllers
- Returning ORM entities to clients
- `@ts-ignore` without justification
- `process.env` outside config files