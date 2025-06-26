# ArkTS Database ORM Patterns

## Introduction

Object-Relational Mapping (ORM) patterns simplify database interactions in ArkTS applications. This guide explores advanced ORM implementations, query builders, migration systems, and connection pooling for robust data management.

## Core ORM Foundation

### Base Entity System

```typescript
interface EntityMetadata {
  tableName: string;
  primaryKey: string;
  columns: Map<string, ColumnMetadata>;
  relations: Map<string, RelationMetadata>;
}

interface ColumnMetadata {
  name: string;
  type: "string" | "number" | "boolean" | "date";
  nullable: boolean;
  unique: boolean;
  default?: any;
}

interface RelationMetadata {
  type: "one-to-one" | "one-to-many" | "many-to-one" | "many-to-many";
  target: string;
  foreignKey: string;
  cascadeDelete: boolean;
}

abstract class BaseEntity {
  private static metadata = new Map<string, EntityMetadata>();

  static getMetadata(): EntityMetadata {
    const className = this.name;
    if (!this.metadata.has(className)) {
      throw new Error(`No metadata found for entity: ${className}`);
    }
    return this.metadata.get(className)!;
  }

  static setMetadata(metadata: EntityMetadata): void {
    this.metadata.set(this.name, metadata);
  }

  getPrimaryKeyValue(): any {
    const metadata = (this.constructor as typeof BaseEntity).getMetadata();
    return (this as any)[metadata.primaryKey];
  }

  toJSON(): Record<string, any> {
    const metadata = (this.constructor as typeof BaseEntity).getMetadata();
    const result: Record<string, any> = {};

    metadata.columns.forEach((column, property) => {
      result[column.name] = (this as any)[property];
    });

    return result;
  }
}

// Decorators for entity definition
function Entity(tableName: string) {
  return function <T extends typeof BaseEntity>(constructor: T) {
    const metadata: EntityMetadata = {
      tableName,
      primaryKey: "",
      columns: new Map(),
      relations: new Map(),
    };
    constructor.setMetadata(metadata);
  };
}

function PrimaryKey() {
  return function (target: any, propertyKey: string) {
    const constructor = target.constructor as typeof BaseEntity;
    const metadata = constructor.getMetadata();
    metadata.primaryKey = propertyKey;
  };
}

function Column(options: Partial<ColumnMetadata> = {}) {
  return function (target: any, propertyKey: string) {
    const constructor = target.constructor as typeof BaseEntity;
    const metadata = constructor.getMetadata();

    metadata.columns.set(propertyKey, {
      name: options.name || propertyKey,
      type: options.type || "string",
      nullable: options.nullable || false,
      unique: options.unique || false,
      default: options.default,
    });
  };
}

function Relation(options: Partial<RelationMetadata>) {
  return function (target: any, propertyKey: string) {
    const constructor = target.constructor as typeof BaseEntity;
    const metadata = constructor.getMetadata();

    metadata.relations.set(propertyKey, {
      type: options.type!,
      target: options.target!,
      foreignKey: options.foreignKey!,
      cascadeDelete: options.cascadeDelete || false,
    });
  };
}
```

### Query Builder

```typescript
class QueryBuilder<T> {
  private selectFields: string[] = ["*"];
  private whereConditions: WhereCondition[] = [];
  private joinClauses: JoinClause[] = [];
  private orderByFields: OrderByField[] = [];
  private limitValue?: number;
  private offsetValue?: number;
  private tableName: string;

  constructor(private entityClass: typeof BaseEntity) {
    this.tableName = entityClass.getMetadata().tableName;
  }

  select(...fields: string[]): QueryBuilder<T> {
    this.selectFields = fields.length > 0 ? fields : ["*"];
    return this;
  }

  where(field: string, operator: string, value: any): QueryBuilder<T> {
    this.whereConditions.push({ field, operator, value, connector: "AND" });
    return this;
  }

  orWhere(field: string, operator: string, value: any): QueryBuilder<T> {
    this.whereConditions.push({ field, operator, value, connector: "OR" });
    return this;
  }

  whereIn(field: string, values: any[]): QueryBuilder<T> {
    this.whereConditions.push({
      field,
      operator: "IN",
      value: values,
      connector: "AND",
    });
    return this;
  }

  join(table: string, on: string): QueryBuilder<T> {
    this.joinClauses.push({ type: "INNER", table, on });
    return this;
  }

  leftJoin(table: string, on: string): QueryBuilder<T> {
    this.joinClauses.push({ type: "LEFT", table, on });
    return this;
  }

  orderBy(field: string, direction: "ASC" | "DESC" = "ASC"): QueryBuilder<T> {
    this.orderByFields.push({ field, direction });
    return this;
  }

  limit(count: number): QueryBuilder<T> {
    this.limitValue = count;
    return this;
  }

  offset(count: number): QueryBuilder<T> {
    this.offsetValue = count;
    return this;
  }

  toSQL(): { sql: string; parameters: any[] } {
    const parameters: any[] = [];
    let sql = `SELECT ${this.selectFields.join(", ")} FROM ${this.tableName}`;

    // Add JOINs
    this.joinClauses.forEach((join) => {
      sql += ` ${join.type} JOIN ${join.table} ON ${join.on}`;
    });

    // Add WHERE clause
    if (this.whereConditions.length > 0) {
      sql += " WHERE ";
      this.whereConditions.forEach((condition, index) => {
        if (index > 0) {
          sql += ` ${condition.connector} `;
        }

        if (condition.operator === "IN") {
          const placeholders = (condition.value as any[])
            .map(() => "?")
            .join(", ");
          sql += `${condition.field} ${condition.operator} (${placeholders})`;
          parameters.push(...condition.value);
        } else {
          sql += `${condition.field} ${condition.operator} ?`;
          parameters.push(condition.value);
        }
      });
    }

    // Add ORDER BY
    if (this.orderByFields.length > 0) {
      sql += " ORDER BY ";
      sql += this.orderByFields
        .map((field) => `${field.field} ${field.direction}`)
        .join(", ");
    }

    // Add LIMIT and OFFSET
    if (this.limitValue) {
      sql += ` LIMIT ${this.limitValue}`;
    }
    if (this.offsetValue) {
      sql += ` OFFSET ${this.offsetValue}`;
    }

    return { sql, parameters };
  }

  async execute(): Promise<T[]> {
    const query = this.toSQL();
    const repository = new Repository(this.entityClass);
    return repository.query(query.sql, query.parameters);
  }

  async first(): Promise<T | null> {
    const results = await this.limit(1).execute();
    return results[0] || null;
  }

  async count(): Promise<number> {
    const query = new QueryBuilder(this.entityClass).select(
      "COUNT(*) as count"
    );

    // Copy conditions
    query.whereConditions = [...this.whereConditions];
    query.joinClauses = [...this.joinClauses];

    const result = await query.execute();
    return (result[0] as any).count;
  }
}

interface WhereCondition {
  field: string;
  operator: string;
  value: any;
  connector: "AND" | "OR";
}

interface JoinClause {
  type: "INNER" | "LEFT" | "RIGHT";
  table: string;
  on: string;
}

interface OrderByField {
  field: string;
  direction: "ASC" | "DESC";
}
```

### Repository Pattern

```typescript
class Repository<T extends BaseEntity> {
  constructor(private entityClass: typeof BaseEntity) {}

  async findById(id: any): Promise<T | null> {
    const metadata = this.entityClass.getMetadata();
    const query = new QueryBuilder(this.entityClass).where(
      metadata.primaryKey,
      "=",
      id
    );

    return (await query.first()) as T | null;
  }

  async findAll(): Promise<T[]> {
    const query = new QueryBuilder(this.entityClass);
    return (await query.execute()) as T[];
  }

  async findBy(criteria: Partial<T>): Promise<T[]> {
    const query = new QueryBuilder(this.entityClass);

    Object.entries(criteria).forEach(([field, value]) => {
      query.where(field, "=", value);
    });

    return (await query.execute()) as T[];
  }

  async save(entity: T): Promise<T> {
    const primaryKeyValue = entity.getPrimaryKeyValue();

    if (primaryKeyValue) {
      return this.update(entity);
    } else {
      return this.create(entity);
    }
  }

  async create(entity: T): Promise<T> {
    const metadata = this.entityClass.getMetadata();
    const data = entity.toJSON();

    const fields = Object.keys(data);
    const values = Object.values(data);
    const placeholders = fields.map(() => "?").join(", ");

    const sql = `INSERT INTO ${metadata.tableName} (${fields.join(
      ", "
    )}) VALUES (${placeholders})`;

    const result = await DatabaseManager.execute(sql, values);

    // Set generated primary key
    if (result.insertId) {
      (entity as any)[metadata.primaryKey] = result.insertId;
    }

    return entity;
  }

  async update(entity: T): Promise<T> {
    const metadata = this.entityClass.getMetadata();
    const data = entity.toJSON();
    const primaryKeyValue = entity.getPrimaryKeyValue();

    const fields = Object.keys(data).filter(
      (key) => key !== metadata.primaryKey
    );
    const values = fields.map((field) => data[field]);

    const setClause = fields.map((field) => `${field} = ?`).join(", ");
    const sql = `UPDATE ${metadata.tableName} SET ${setClause} WHERE ${metadata.primaryKey} = ?`;

    await DatabaseManager.execute(sql, [...values, primaryKeyValue]);
    return entity;
  }

  async delete(entity: T): Promise<void> {
    const metadata = this.entityClass.getMetadata();
    const primaryKeyValue = entity.getPrimaryKeyValue();

    const sql = `DELETE FROM ${metadata.tableName} WHERE ${metadata.primaryKey} = ?`;
    await DatabaseManager.execute(sql, [primaryKeyValue]);
  }

  query(): QueryBuilder<T> {
    return new QueryBuilder(this.entityClass);
  }

  async query(sql: string, parameters: any[] = []): Promise<T[]> {
    const rows = await DatabaseManager.query(sql, parameters);
    return rows.map((row) => this.mapRowToEntity(row));
  }

  private mapRowToEntity(row: any): T {
    const entity = new (this.entityClass as any)();
    const metadata = this.entityClass.getMetadata();

    metadata.columns.forEach((column, property) => {
      entity[property] = row[column.name];
    });

    return entity;
  }
}
```

## Advanced ORM Features

### Migration System

```typescript
interface Migration {
  version: string;
  up(): Promise<void>;
  down(): Promise<void>;
}

abstract class BaseMigration implements Migration {
  abstract version: string;

  abstract up(): Promise<void>;
  abstract down(): Promise<void>;

  protected async createTable(
    name: string,
    callback: (table: TableBuilder) => void
  ): Promise<void> {
    const table = new TableBuilder(name);
    callback(table);
    await DatabaseManager.execute(table.toSQL());
  }

  protected async dropTable(name: string): Promise<void> {
    await DatabaseManager.execute(`DROP TABLE IF EXISTS ${name}`);
  }

  protected async addColumn(
    table: string,
    column: string,
    type: string
  ): Promise<void> {
    await DatabaseManager.execute(
      `ALTER TABLE ${table} ADD COLUMN ${column} ${type}`
    );
  }

  protected async dropColumn(table: string, column: string): Promise<void> {
    await DatabaseManager.execute(`ALTER TABLE ${table} DROP COLUMN ${column}`);
  }
}

class TableBuilder {
  private columns: string[] = [];
  private constraints: string[] = [];

  constructor(private tableName: string) {}

  id(name = "id"): TableBuilder {
    this.columns.push(`${name} INTEGER PRIMARY KEY AUTOINCREMENT`);
    return this;
  }

  string(name: string, length = 255): TableBuilder {
    this.columns.push(`${name} VARCHAR(${length})`);
    return this;
  }

  integer(name: string): TableBuilder {
    this.columns.push(`${name} INTEGER`);
    return this;
  }

  boolean(name: string): TableBuilder {
    this.columns.push(`${name} BOOLEAN DEFAULT FALSE`);
    return this;
  }

  timestamp(name: string): TableBuilder {
    this.columns.push(`${name} TIMESTAMP`);
    return this;
  }

  nullable(): TableBuilder {
    const lastColumn = this.columns[this.columns.length - 1];
    this.columns[this.columns.length - 1] = lastColumn + " NULL";
    return this;
  }

  unique(): TableBuilder {
    const lastColumn = this.columns[this.columns.length - 1];
    this.columns[this.columns.length - 1] = lastColumn + " UNIQUE";
    return this;
  }

  foreignKey(column: string, references: string): TableBuilder {
    this.constraints.push(`FOREIGN KEY (${column}) REFERENCES ${references}`);
    return this;
  }

  toSQL(): string {
    const allDefinitions = [...this.columns, ...this.constraints];
    return `CREATE TABLE ${this.tableName} (\n  ${allDefinitions.join(
      ",\n  "
    )}\n)`;
  }
}

class MigrationManager {
  private migrations: Migration[] = [];

  addMigration(migration: Migration): void {
    this.migrations.push(migration);
    this.migrations.sort((a, b) => a.version.localeCompare(b.version));
  }

  async migrate(): Promise<void> {
    await this.createMigrationTable();
    const appliedMigrations = await this.getAppliedMigrations();

    for (const migration of this.migrations) {
      if (!appliedMigrations.includes(migration.version)) {
        console.log(`Running migration: ${migration.version}`);
        await migration.up();
        await this.recordMigration(migration.version);
      }
    }
  }

  async rollback(steps = 1): Promise<void> {
    const appliedMigrations = await this.getAppliedMigrations();
    const toRollback = appliedMigrations.slice(-steps);

    for (const version of toRollback.reverse()) {
      const migration = this.migrations.find((m) => m.version === version);
      if (migration) {
        console.log(`Rolling back migration: ${version}`);
        await migration.down();
        await this.removeMigration(version);
      }
    }
  }

  private async createMigrationTable(): Promise<void> {
    const sql = `
      CREATE TABLE IF NOT EXISTS migrations (
        version VARCHAR(255) PRIMARY KEY,
        applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `;
    await DatabaseManager.execute(sql);
  }

  private async getAppliedMigrations(): Promise<string[]> {
    const rows = await DatabaseManager.query(
      "SELECT version FROM migrations ORDER BY version"
    );
    return rows.map((row) => row.version);
  }

  private async recordMigration(version: string): Promise<void> {
    await DatabaseManager.execute(
      "INSERT INTO migrations (version) VALUES (?)",
      [version]
    );
  }

  private async removeMigration(version: string): Promise<void> {
    await DatabaseManager.execute("DELETE FROM migrations WHERE version = ?", [
      version,
    ]);
  }
}
```

### Connection Pool

```typescript
interface ConnectionConfig {
  database: string;
  maxConnections: number;
  acquireTimeout: number;
  idleTimeout: number;
}

class ConnectionPool {
  private availableConnections: DatabaseConnection[] = [];
  private activeConnections = new Set<DatabaseConnection>();
  private waitingQueue: ((connection: DatabaseConnection) => void)[] = [];
  private config: ConnectionConfig;

  constructor(config: ConnectionConfig) {
    this.config = config;
    this.initializePool();
  }

  async acquire(): Promise<DatabaseConnection> {
    return new Promise((resolve, reject) => {
      const timeout = setTimeout(() => {
        const index = this.waitingQueue.indexOf(resolve);
        if (index >= 0) {
          this.waitingQueue.splice(index, 1);
          reject(new Error("Connection acquire timeout"));
        }
      }, this.config.acquireTimeout);

      const tryAcquire = () => {
        if (this.availableConnections.length > 0) {
          clearTimeout(timeout);
          const connection = this.availableConnections.pop()!;
          this.activeConnections.add(connection);
          resolve(connection);
        } else if (this.getTotalConnections() < this.config.maxConnections) {
          clearTimeout(timeout);
          const connection = this.createConnection();
          this.activeConnections.add(connection);
          resolve(connection);
        } else {
          this.waitingQueue.push(resolve);
        }
      };

      tryAcquire();
    });
  }

  release(connection: DatabaseConnection): void {
    this.activeConnections.delete(connection);

    if (this.waitingQueue.length > 0) {
      const waiter = this.waitingQueue.shift()!;
      this.activeConnections.add(connection);
      waiter(connection);
    } else {
      this.availableConnections.push(connection);
      this.scheduleIdleCleanup(connection);
    }
  }

  async close(): Promise<void> {
    const allConnections = [
      ...this.availableConnections,
      ...this.activeConnections,
    ];

    await Promise.all(allConnections.map((conn) => conn.close()));

    this.availableConnections = [];
    this.activeConnections.clear();
    this.waitingQueue = [];
  }

  getStats(): PoolStats {
    return {
      available: this.availableConnections.length,
      active: this.activeConnections.size,
      waiting: this.waitingQueue.length,
      total: this.getTotalConnections(),
    };
  }

  private initializePool(): void {
    const initialConnections = Math.min(2, this.config.maxConnections);
    for (let i = 0; i < initialConnections; i++) {
      this.availableConnections.push(this.createConnection());
    }
  }

  private createConnection(): DatabaseConnection {
    return new DatabaseConnection(this.config.database);
  }

  private getTotalConnections(): number {
    return this.availableConnections.length + this.activeConnections.size;
  }

  private scheduleIdleCleanup(connection: DatabaseConnection): void {
    setTimeout(() => {
      const index = this.availableConnections.indexOf(connection);
      if (index >= 0 && this.availableConnections.length > 1) {
        this.availableConnections.splice(index, 1);
        connection.close();
      }
    }, this.config.idleTimeout);
  }
}

interface PoolStats {
  available: number;
  active: number;
  waiting: number;
  total: number;
}

class DatabaseConnection {
  constructor(private database: string) {}

  async query(sql: string, parameters: any[] = []): Promise<any[]> {
    // Database-specific implementation
    console.log(`Executing: ${sql} with params:`, parameters);
    return [];
  }

  async execute(
    sql: string,
    parameters: any[] = []
  ): Promise<{ insertId?: number; affectedRows: number }> {
    // Database-specific implementation
    console.log(`Executing: ${sql} with params:`, parameters);
    return { affectedRows: 1 };
  }

  async close(): Promise<void> {
    // Clean up connection
  }
}

// Global database manager
class DatabaseManager {
  private static pool: ConnectionPool;

  static initialize(config: ConnectionConfig): void {
    this.pool = new ConnectionPool(config);
  }

  static async query(sql: string, parameters: any[] = []): Promise<any[]> {
    const connection = await this.pool.acquire();
    try {
      return await connection.query(sql, parameters);
    } finally {
      this.pool.release(connection);
    }
  }

  static async execute(
    sql: string,
    parameters: any[] = []
  ): Promise<{ insertId?: number; affectedRows: number }> {
    const connection = await this.pool.acquire();
    try {
      return await connection.execute(sql, parameters);
    } finally {
      this.pool.release(connection);
    }
  }

  static async transaction<T>(
    callback: (connection: DatabaseConnection) => Promise<T>
  ): Promise<T> {
    const connection = await this.pool.acquire();
    try {
      await connection.execute("BEGIN TRANSACTION");
      const result = await callback(connection);
      await connection.execute("COMMIT");
      return result;
    } catch (error) {
      await connection.execute("ROLLBACK");
      throw error;
    } finally {
      this.pool.release(connection);
    }
  }

  static getStats(): PoolStats {
    return this.pool.getStats();
  }
}
```

## Usage Examples

### Entity Definition

```typescript
@Entity("users")
class User extends BaseEntity {
  @PrimaryKey()
  @Column({ type: "number" })
  id: number = 0;

  @Column({ type: "string", unique: true })
  email: string = "";

  @Column({ type: "string" })
  name: string = "";

  @Column({ type: "date" })
  createdAt: Date = new Date();

  @Relation({ type: "one-to-many", target: "Post", foreignKey: "userId" })
  posts: Post[] = [];
}

@Entity("posts")
class Post extends BaseEntity {
  @PrimaryKey()
  @Column({ type: "number" })
  id: number = 0;

  @Column({ type: "string" })
  title: string = "";

  @Column({ type: "string" })
  content: string = "";

  @Column({ type: "number" })
  userId: number = 0;

  @Relation({ type: "many-to-one", target: "User", foreignKey: "userId" })
  user?: User;
}

// Usage
const userRepository = new Repository(User);
const users = await userRepository
  .query()
  .where("name", "LIKE", "%John%")
  .orderBy("createdAt", "DESC")
  .limit(10)
  .execute();
```

## Conclusion

Advanced ORM patterns in ArkTS provide:

- Type-safe database operations
- Flexible query building capabilities
- Automated schema migrations
- Connection pooling for performance
- Repository pattern for data access
- Transaction management support

These patterns enable robust, maintainable data layer architectures in HarmonyOS applications.
