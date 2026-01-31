---
name: mongodb-mongoose
description: Expert guidance for MongoDB and Mongoose in scalable backend systems. Use this skill when designing MongoDB schemas, data models, indexes, queries, or integrating MongoDB with NestJS applications. Triggers on schema design, embedding vs referencing decisions, indexing strategies, pagination, transactions, repository patterns, query optimization, and production database best practices. Covers Mongoose ODM patterns, performance tuning, and data integrity.
---

# MongoDB & Mongoose Development Skill

Design efficient, secure, and maintainable data models optimized for high-performance applications.

## Design Principles

- Design schemas based on **access patterns**, not theoretical normalization
- Prefer explicit schema design over flexible "schemaless" usage
- Optimize for read performance first, then writes
- Model data to avoid excessive joins (`$lookup`) in hot paths

**Golden Rule:** If MongoDB feels slow, your schema or indexes are wrong—not MongoDB.

## Schema Design (Mongoose)

Every collection MUST have a defined schema with explicit configuration.

```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { Document, Types } from 'mongoose';

@Schema({
  timestamps: true,                    // Enable only if needed
  versionKey: false,                   // Disable __v unless required
  collection: 'users',                 // Explicit collection name
  toJSON: {
    virtuals: true,
    transform: (_, ret) => {
      ret.id = ret._id;
      delete ret._id;
      delete ret.password;             // Never expose sensitive fields
      return ret;
    },
  },
})
export class User extends Document {
  @Prop({ required: true, trim: true })
  name: string;

  @Prop({ 
    required: true, 
    unique: true, 
    lowercase: true, 
    trim: true,
    index: true,
  })
  email: string;

  @Prop({ required: true, select: false })  // Exclude by default
  password: string;

  @Prop({ 
    type: String, 
    enum: ['active', 'inactive', 'suspended'],
    default: 'active',
  })
  status: string;

  @Prop({ type: Types.ObjectId, ref: 'Organization', index: true })
  organizationId: Types.ObjectId;

  // Bounded embedded data (1:few)
  @Prop({
    type: [{
      street: String,
      city: String,
      country: String,
    }],
    default: [],
    _id: false,
  })
  addresses: Array<{ street: string; city: string; country: string }>;
}

export const UserSchema = SchemaFactory.createForClass(User);

// Compound indexes for common queries
UserSchema.index({ organizationId: 1, status: 1 });
UserSchema.index({ createdAt: -1 });
```

### Schema Rules

- Disable versioning (`versionKey: false`) unless optimistic locking needed
- Disable auto timestamps unless needed
- Use explicit field types, required flags, default values
- Avoid deeply nested documents (>3 levels)

## Data Modeling: Embed vs Reference

### Embed When

- Data is tightly coupled
- Data is read together
- Cardinality is bounded (1:few)

```typescript
// EMBED: User addresses (bounded, read together)
@Schema()
export class User {
  @Prop({
    type: [{
      label: String,
      street: String,
      city: String,
    }],
    validate: [arr => arr.length <= 5, 'Max 5 addresses'],
  })
  addresses: Address[];
}
```

### Reference When

- Data grows unbounded
- Data is shared across domains
- Independent lifecycle exists

```typescript
// REFERENCE: User orders (unbounded, independent lifecycle)
@Schema()
export class Order {
  @Prop({ type: Types.ObjectId, ref: 'User', required: true, index: true })
  userId: Types.ObjectId;

  @Prop({ type: Types.ObjectId, ref: 'Product', required: true })
  productId: Types.ObjectId;

  @Prop({ required: true })
  quantity: number;
}
```

### Decision Matrix

| Scenario | Pattern | Reason |
|----------|---------|--------|
| User → Addresses (max 5) | Embed | Bounded, read together |
| User → Orders | Reference | Unbounded growth |
| Post → Comments (many) | Reference | Unbounded, independent |
| Post → Author info | Embed subset | Read together, denormalize |
| Product → Categories | Reference | Shared across products |

## Indexing (CRITICAL)

Create indexes for all frequently queried fields. **If a query runs often, it must be indexed.**

```typescript
// Single field indexes
UserSchema.index({ email: 1 }, { unique: true });
UserSchema.index({ organizationId: 1 });
UserSchema.index({ createdAt: -1 });

// Compound indexes (order matters!)
// Supports: find by org, find by org+status, sort by createdAt
UserSchema.index({ organizationId: 1, status: 1, createdAt: -1 });

// Text search index
ProductSchema.index({ name: 'text', description: 'text' });

// TTL index for auto-expiring documents
SessionSchema.index({ expiresAt: 1 }, { expireAfterSeconds: 0 });

// Partial index (index only matching documents)
UserSchema.index(
  { email: 1 },
  { partialFilterExpression: { status: 'active' } }
);
```

### Indexing Rules

- Index all frequently queried fields
- Index foreign reference fields
- Index sort + filter combinations together
- Use compound indexes deliberately (order matters: equality → sort → range)
- Never rely on collection scans in production
- Periodically audit unused indexes

## ObjectId Usage

```typescript
import { Types, isValidObjectId } from 'mongoose';

// Validate at API boundaries
function validateObjectId(id: string): Types.ObjectId {
  if (!isValidObjectId(id)) {
    throw new BadRequestException('Invalid ID format');
  }
  return new Types.ObjectId(id);
}

// In DTOs
import { IsMongoId } from 'class-validator';

export class GetUserDto {
  @IsMongoId()
  readonly id: string;
}

// In repository
async findById(id: string): Promise<User | null> {
  const objectId = validateObjectId(id);
  return this.userModel.findById(objectId).lean().exec();
}
```

### ObjectId Rules

- Always use `ObjectId` for `_id`
- Never use strings where `ObjectId` is expected
- Validate ObjectId inputs at API boundaries
- Avoid exposing raw ObjectIds directly to clients

## Query Best Practices

```typescript
@Injectable()
export class UsersRepository {
  constructor(@InjectModel(User.name) private userModel: Model<User>) {}

  // Always use projections
  async findByEmail(email: string): Promise<User | null> {
    return this.userModel
      .findOne({ email })
      .select('name email status organizationId')  // Explicit projection
      .lean()                                       // Return plain object
      .exec();
  }

  // Never find() without filters
  async findByOrganization(orgId: string): Promise<User[]> {
    return this.userModel
      .find({ organizationId: new Types.ObjectId(orgId) })
      .select('name email status')
      .sort({ createdAt: -1 })
      .limit(100)                                   // Always limit
      .lean()
      .exec();
  }

  // Bulk operations for batch updates
  async deactivateUsers(userIds: string[]): Promise<void> {
    await this.userModel.updateMany(
      { _id: { $in: userIds.map(id => new Types.ObjectId(id)) } },
      { $set: { status: 'inactive', updatedAt: new Date() } }
    );
  }
}
```

### Query Rules

- Always use projections (`select`)
- Limit returned fields explicitly
- Never use `find()` without filters
- Paginate all list queries
- Avoid `$where` and regex-heavy queries on large collections
- Use `.lean()` for read-heavy queries

## Pagination

Prefer cursor-based pagination for large datasets.

```typescript
// Cursor-based pagination (recommended)
async findWithCursor(
  cursor?: string,
  limit = 20,
): Promise<{ items: User[]; nextCursor: string | null }> {
  const query: FilterQuery<User> = {};
  
  if (cursor) {
    query._id = { $lt: new Types.ObjectId(cursor) };
  }

  const items = await this.userModel
    .find(query)
    .sort({ _id: -1 })
    .limit(limit + 1)
    .lean()
    .exec();

  const hasMore = items.length > limit;
  if (hasMore) items.pop();

  return {
    items,
    nextCursor: hasMore ? items[items.length - 1]._id.toString() : null,
  };
}

// Skip-based only for small datasets
async findPaginated(page: number, limit: number): Promise<PaginatedResult<User>> {
  const [items, total] = await Promise.all([
    this.userModel
      .find()
      .skip((page - 1) * limit)
      .limit(limit)
      .lean()
      .exec(),
    this.userModel.countDocuments(),
  ]);

  return { items, total, page, totalPages: Math.ceil(total / limit) };
}
```

### Pagination Rules

- Use cursor-based for large datasets
- Use `_id` or indexed fields for cursors
- Avoid skip-based pagination on large collections (>10k docs)

## Transactions

Use transactions only when required. Keep them short-lived.

```typescript
async transferCredits(fromId: string, toId: string, amount: number): Promise<void> {
  const session = await this.connection.startSession();
  
  try {
    await session.withTransaction(async () => {
      // Atomic operations within transaction
      await this.userModel.updateOne(
        { _id: fromId, credits: { $gte: amount } },
        { $inc: { credits: -amount } },
        { session }
      );

      await this.userModel.updateOne(
        { _id: toId },
        { $inc: { credits: amount } },
        { session }
      );
    });
  } finally {
    await session.endSession();
  }
}
```

### Transaction Rules

- Keep transactions short-lived
- Never perform external calls inside transactions
- Prefer eventual consistency when possible
- Use atomic operators (`$set`, `$inc`, `$push`) outside transactions when possible

## Repository Pattern (NestJS)

Access MongoDB ONLY via repositories.

```typescript
// users.repository.ts
@Injectable()
export class UsersRepository {
  constructor(
    @InjectModel(User.name) private readonly model: Model<User>,
  ) {}

  async create(data: CreateUserData): Promise<User> {
    const user = new this.model(data);
    return user.save();
  }

  async findById(id: string): Promise<User | null> {
    return this.model.findById(id).lean().exec();
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.model.findOne({ email }).exec();
  }

  async updateStatus(id: string, status: string): Promise<User | null> {
    return this.model
      .findByIdAndUpdate(id, { $set: { status } }, { new: true })
      .lean()
      .exec();
  }
}

// users.service.ts - no Mongoose internals
@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  async getUser(id: string): Promise<UserResponseDto> {
    const user = await this.usersRepository.findById(id);
    if (!user) throw new NotFoundException('User not found');
    return this.toResponseDto(user);
  }
}
```

### Repository Rules

- Repositories contain database queries only
- Do NOT contain business logic
- Services must not know Mongoose internals
- Never use models directly in controllers

## Error Handling

Handle MongoDB errors and map to domain exceptions.

```typescript
import { MongoError } from 'mongodb';

@Injectable()
export class UsersRepository {
  async create(data: CreateUserData): Promise<User> {
    try {
      const user = new this.model(data);
      return await user.save();
    } catch (error) {
      if (error instanceof MongoError && error.code === 11000) {
        throw new ConflictException('Email already exists');
      }
      if (error.name === 'ValidationError') {
        throw new BadRequestException('Invalid user data');
      }
      if (error.name === 'CastError') {
        throw new BadRequestException('Invalid ID format');
      }
      throw error;
    }
  }
}
```

### Error Rules

- Handle duplicate key errors (code 11000)
- Handle validation errors
- Handle cast errors
- Never leak raw MongoDB errors to clients
- Map DB errors to domain-level exceptions

## Atomic Operators

Use atomic operators to avoid race conditions.

```typescript
// GOOD: Atomic increment
await this.model.updateOne(
  { _id: userId },
  { $inc: { loginCount: 1 } }
);

// GOOD: Atomic push with limit
await this.model.updateOne(
  { _id: postId },
  {
    $push: {
      recentComments: {
        $each: [newComment],
        $slice: -10,  // Keep last 10 only
      },
    },
  }
);

// GOOD: Atomic conditional update
await this.model.updateOne(
  { _id: productId, stock: { $gte: quantity } },
  { $inc: { stock: -quantity } }
);

// BAD: Read-modify-write race condition
const user = await this.model.findById(id);
user.loginCount += 1;  // Another request could increment between read and write
await user.save();
```

## Quick Reference

| Operation | Pattern | Avoid |
|-----------|---------|-------|
| Read-heavy queries | `.lean()` | Returning full documents |
| Batch updates | `updateMany`, `bulkWrite` | Loop with individual saves |
| Counter increment | `$inc` | Read-modify-write |
| Array append | `$push` | Read array, push, save |
| Large lists | Cursor pagination | Skip pagination |
| Related data (bounded) | Embed | Over-reference |
| Related data (unbounded) | Reference | Over-embed |
| Frequent queries | Index | Collection scan |

## Anti-Patterns ❌

- Unindexed production queries
- Over-embedded documents (unbounded arrays)
- God collections (>50 fields)
- Skip-based pagination on large datasets
- Direct model access in controllers
- Storing derived data without justification
- `find()` without filters or limits
- Read-modify-write without atomic operators
- Exposing raw MongoDB errors to clients