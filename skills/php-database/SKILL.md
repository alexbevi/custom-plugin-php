---
name: php-database
version: "2.1.0"
description: PHP database mastery - PDO, Eloquent, Doctrine, MongoDB (PHP Library + Laravel integration), query optimization, and migrations
sasmp_version: "1.3.0"
bonded_agent: 05-php-database
bond_type: PRIMARY_BOND
atomic: true
category: data
---

# PHP Database Skill

> Atomic skill for mastering database operations in PHP (relational + MongoDB)

## Overview

Comprehensive skill for PHP database interactions covering:
- **Relational**: PDO, SQL query patterns, migrations, transaction patterns
- **Framework ORMs**: Laravel Eloquent, Symfony Doctrine
- **MongoDB**:
  - Vanilla PHP via the **MongoDB PHP extension** + **MongoDB PHP Library**
  - Laravel via the official **mongodb/laravel-mongodb** integration (Eloquent + Query Builder, Aggregation Builder, Schema Builder)

## Skill Parameters

### Input Validation
```typescript
interface SkillParams {
  topic:
    | "pdo"              // Native PHP relational access (SQL)
    | "eloquent"         // Laravel ORM (SQL + MongoDB via mongodb/laravel-mongodb)
    | "doctrine"         // Symfony ORM (SQL)
    | "optimization"     // Query tuning, indexing (SQL + MongoDB)
    | "migrations"       // Schema versioning (SQL + MongoDB schema/index management)
    | "transactions";    // ACID, locking (SQL + MongoDB)

  level: "beginner" | "intermediate" | "advanced";

  // NOTE: "mongodb" routes guidance to MongoDB PHP Library + Laravel MongoDB integration.
  database?: "mysql" | "postgresql" | "sqlite" | "mongodb";

  // ORM selection is optional; for MongoDB+Laravel this is typically "eloquent".
  orm?: "eloquent" | "doctrine" | "none";

  // Optional hint so the skill can tailor examples.
  framework?: "laravel" | "symfony" | "none";
}
```

### Validation Rules
```yaml
validation:
  topic:
    required: true
    allowed: [pdo, eloquent, doctrine, optimization, migrations, transactions]
  level:
    required: true
  database:
    default: "mysql"
  orm:
    default: "none"
  framework:
    default: "none"

compatibility_notes:
  - when: { database: "mongodb", topic: "pdo" }
    note: "PDO is SQL-only. Use MongoDB PHP Library (MongoDB\\Client) or Laravel MongoDB integration instead."
  - when: { database: "mongodb", topic: "doctrine" }
    note: "Doctrine here refers to SQL ORM. Prefer MongoDB PHP Library or mongodb/laravel-mongodb for MongoDB."
```

## Learning Modules

### Module 1: PDO Fundamentals (Relational)
```yaml
beginner:
  - Connection and DSN
  - Prepared statements
  - Fetching results

intermediate:
  - Error handling modes
  - Transactions basics
  - Named placeholders

advanced:
  - Connection pooling
  - Stored procedures
  - Batch operations
```

### Module 2: Query Optimization (SQL + MongoDB)
```yaml
beginner:
  - SQL: Basic indexing
  - SQL: EXPLAIN basics
  - SQL: WHERE clause optimization
  - MongoDB: Index the fields you filter/sort on
  - MongoDB: Use projections to limit returned fields

intermediate:
  - SQL: Composite indexes
  - SQL: Join optimization
  - SQL: Query profiling
  - MongoDB: Compound indexes for filter+sort patterns
  - MongoDB: Aggregation pipeline ordering (match early, project late)

advanced:
  - SQL: Execution plan analysis
  - SQL: Partitioning
  - SQL: Query caching
  - MongoDB: Explain plans + index coverage checks
  - MongoDB: Aggregation performance tuning ($match early, reduce cardinality, avoid unbounded $lookup)
```

### Module 3: ORM Patterns (Eloquent + Doctrine + MongoDB Eloquent)
```yaml
beginner:
  - Model basics
  - CRUD operations
  - Simple relationships

intermediate:
  - Eager loading (N+1 prevention)
  - Query scopes
  - Lifecycle hooks

advanced:
  - Custom repositories
  - Result caching
  - Batch processing

mongodb_extensions:
  - Eloquent model base class for MongoDB (MongoDB\\Laravel\\Eloquent\\Model)
  - Collection mapping via $table
  - Optional schema versioning patterns at the model layer
```

### Module 4: MongoDB Fundamentals (PHP Library)
```yaml
beginner:
  - Install MongoDB PHP extension + mongodb/mongodb library
  - Connect via URI (mongodb:// or mongodb+srv://)
  - CRUD: insertOne/findOne/updateOne/deleteOne
  - Filters, projections, limits

intermediate:
  - Update operators ($set, $inc, $push)
  - Bulk writes
  - Index creation (single + compound)
  - Pagination patterns (range queries vs skip/limit)

advanced:
  - Aggregation pipelines
  - Transactions (requirements + retry strategy)
  - Write concerns / read preferences (when relevant)
```

### Module 5: Laravel + MongoDB (mongodb/laravel-mongodb)
```yaml
beginner:
  - Install mongodb/laravel-mongodb and configure config/database.php with driver= mongodb and dsn env var
  - Define models extending MongoDB\\Laravel\\Eloquent\\Model
  - Basic Eloquent CRUD + query builder filters

intermediate:
  - Aggregation Builder (Model::aggregate() + stages)
  - Raw driver access for advanced operations
  - Multi-connection setups (mix SQL + MongoDB connections)

advanced:
  - Schema Builder migrations for indexes + validation
  - Atlas Search / Vector Search index management via migrations (when applicable)
  - Transaction usage patterns + limitations
```

### Module 6: Migrations & Schema Management (SQL + MongoDB)
```yaml
sql:
  - create/alter tables
  - add indexes and constraints
  - rollbacks and data backfills

mongodb:
  - Manage indexes via migrations (index, unique, sparse, TTL)
  - Schema validation via jsonSchema() in migrations
  - Drop indexes safely by name
```

### Module 7: Transactions (SQL + MongoDB)
```yaml
sql:
  - Begin/commit/rollback
  - Isolation levels (where supported)
  - Deadlock detection and retry

mongodb:
  - Single-document operations are atomic by default
  - Multi-document transactions require replica set / sharded cluster
  - No nested transactions; avoid parallel ops in a single tx
  - Prefer DB::transaction(callback, attempts: N) where supported
```

## Error Handling & Retry Logic

```yaml
errors:
  CONNECTION_ERROR:
    code: "DB_001"
    recovery: "Check connection string, credentials, network; retry with backoff"

  DEADLOCK_ERROR:
    code: "DB_002"
    recovery: "Retry transaction with exponential backoff (SQL deadlock / Mongo transient tx errors)"

  QUERY_ERROR:
    code: "DB_003"
    recovery: "Validate query syntax, parameters, and schema assumptions"

  MONGODB_EXTENSION_MISSING:
    code: "MDB_001"
    recovery: "Install/enable the mongodb PHP extension in both CLI + web runtime; verify phpinfo()/php -m"

  MONGODB_TX_UNSUPPORTED:
    code: "MDB_002"
    recovery: "Ensure MongoDB is a replica set or sharded cluster; standalone does not support transactions"

retry:
  max_attempts: 3
  backoff:
    type: exponential
    initial_delay_ms: 100
    max_delay_ms: 2000
  retryable: [CONNECTION_ERROR, DEADLOCK_ERROR]
```

## Code Examples

### PDO with Prepared Statements (Relational)
```php
<?php
declare(strict_types=1);

final class Database
{
    private \PDO $pdo;

    public function __construct(string $dsn, string $user, string $pass)
    {
        $this->pdo = new \PDO($dsn, $user, $pass, [
            \PDO::ATTR_ERRMODE => \PDO::ERRMODE_EXCEPTION,
            \PDO::ATTR_DEFAULT_FETCH_MODE => \PDO::FETCH_ASSOC,
            \PDO::ATTR_EMULATE_PREPARES => false,
        ]);
    }

    public function findById(int $id): ?array
    {
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
        $stmt->execute(['id' => $id]);
        return $stmt->fetch() ?: null;
    }
}
```

### MongoDB PHP Library: Connect + CRUD
```php
<?php
declare(strict_types=1);

use MongoDB\Client;

$uri = getenv('MONGODB_URI') ?: 'mongodb://127.0.0.1:27017';
$dbName = getenv('MONGODB_DATABASE') ?: 'app';

$client = new Client($uri);
$collection = $client->selectDatabase($dbName)->selectCollection('users');

// Create
$insertResult = $collection->insertOne([
    'email' => 'alex@example.com',
    'name' => 'Alex',
    'createdAt' => new MongoDB\BSON\UTCDateTime(),
]);

// Read (projection to limit fields)
$user = $collection->findOne(
    ['_id' => $insertResult->getInsertedId()],
    ['projection' => ['email' => 1, 'name' => 1]]
);

// Update (atomic per document)
$collection->updateOne(
    ['_id' => $insertResult->getInsertedId()],
    ['$set' => ['name' => 'Alex B.']]
);

// Delete
$collection->deleteOne(['_id' => $insertResult->getInsertedId()]);
```

### Laravel MongoDB: Connection Configuration (dsn in config/database.php)
```php
// config/database.php
'connections' => [
    'mongodb' => [
        'driver' => 'mongodb',
        'dsn' => env('DB_URI'),      // e.g. mongodb+srv://...
        'database' => env('DB_NAME', 'app'),
        // 'options' => ['w' => 'majority'],
        // 'driver_options' => ['serverApi' => 1],
    ],
],
'default' => env('DB_CONNECTION', 'mongodb'),
```

### Laravel MongoDB: Model Class + Collection Mapping
```php
<?php
namespace App\Models;

use MongoDB\Laravel\Eloquent\Model;

final class User extends Model
{
    // Optional: set collection name (defaults to snake_case plural)
    protected $table = 'users';

    protected $fillable = ['email', 'name'];
}
```

### Laravel MongoDB: Aggregation Builder (pipeline-style)
```php
<?php
use App\Models\User;
use MongoDB\Builder\Query\Query;
use MongoDB\Builder\Expression\Expression;
use MongoDB\Builder\Accumulator\Accumulator;
use MongoDB\Builder\Type\Sort;

// Example: group users by occupation and count
$results = User::aggregate()
    ->match(Query::query(active: Query::eq(true)))
    ->group(
        _id: Expression::fieldPath('occupation'),
        count: Accumulator::sum(1),
    )
    ->sort(count: Sort::Desc)
    ->get();
```

### Laravel MongoDB: Migrations for Indexes + Schema Validation
```php
<?php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Support\Facades\Schema;
use MongoDB\Laravel\Schema\Blueprint;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('users', function (Blueprint $collection) {
            // Indexes
            $collection->index('email');
            $collection->unique('email', options: ['name' => 'unique_email_idx']);
            $collection->expire('expiresAt', 3600); // TTL

            // Schema validation (server-enforced)
            $collection->jsonSchema(
                schema: [
                    'bsonType' => 'object',
                    'required' => ['email'],
                    'properties' => [
                        'email' => ['bsonType' => 'string'],
                        'expiresAt' => ['bsonType' => 'date'],
                    ],
                ],
                validationLevel: 'strict',
                validationAction: 'error',
            );
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

### Transaction with Retry (Laravel MongoDB)
```php
<?php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    $sender = Account::where('number', 223344)->first();
    $receiver = Account::where('number', 776655)->first();

    $sender->balance -= 200;
    $sender->save();

    $receiver->balance += 200;
    $receiver->save();
}, attempts: 5);
```

## Troubleshooting

| Problem | Detection | Solution |
|---------|-----------|----------|
| N+1 queries | Debugbar shows 100+ queries | Use eager loading with() |
| Slow queries (SQL) | Query > 1 second | Add indexes, run EXPLAIN |
| Slow queries (MongoDB) | High latency, large docs scanned | Add appropriate indexes; reduce fields via projections; move $match early in pipelines |
| Missing MongoDB extension | Class "MongoDB\\Client" not found | Install/enable mongodb extension in both CLI + web runtime |
| Laravel MongoDB DSN misconfig | Connection errors at boot | Ensure config/database.php uses driver=mongodb and dsn from env |
| Unsupported Laravel connection settings | Prefix settings not applied | Avoid prefix/prefix_indexes for MongoDB connection |
| Transactions failing | Runtime errors / unsupported deployment | Use replica set / sharded cluster; avoid nested tx; prefer DB::transaction(..., attempts:N) |
| Memory exhaustion | Memory limit error | Use chunk()/cursor(), paginate, stream results |

### Memory-Efficient Processing (Framework-agnostic)
```php
<?php
// GOOD: Chunk processing
User::chunk(1000, function ($users) {
    foreach ($users as $user) {
        // Process
    }
});

// BETTER: Cursor for minimal memory
foreach (User::cursor() as $user) {
    // Process one at a time
}
```

## Quality Metrics

| Metric | Target |
|--------|--------|
| Prepared statements (SQL) | 100% |
| N+1 prevention | 100% |
| Index recommendations (SQL + MongoDB) | ≥95% |
| Transaction safety | 100% |
| MongoDB schema validation for critical collections (when appropriate) | ≥80% |

## Usage

```
Skill("php-database", {topic: "eloquent", level: "intermediate", database: "mongodb", orm: "eloquent", framework: "laravel"})
Skill("php-database", {topic: "migrations", level: "advanced", database: "mongodb", framework: "laravel"})
Skill("php-database", {topic: "optimization", level: "advanced", database: "mongodb"})
```
