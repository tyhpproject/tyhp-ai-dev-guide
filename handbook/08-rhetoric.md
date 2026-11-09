## 8. `tyhp/rhetoric` (Laravel database access)

Package `tyhp/rhetoric`, namespace `\Tyhp\Rhetoric`. PHP 8.3–8.5, Laravel 13. Composer pin `80N.X.Y` for the PHP you emit (`803` / `804` / `805`). Requires `tyhp/core`, `tyhp/lambda`, `illuminate/database`, `illuminate/support`, `illuminate/console`. Provider `\Tyhp\Rhetoric\Laravel\RhetoricServiceProvider` (package discovery). `config/rhetoric.php` key `contexts` lists context classes; each is a scoped binding. Publish with `vendor:publish --tag=rhetoric-config` and stubs with `--tag=rhetoric-stubs`.

Two ways to call it. **Tyhp** (the next section) passes inline `fn` lambdas; the compiler builds the tree. **PHP** (the last section) does not read the Tyhp language pages: call a method that needs an explicit type as `name(types)(args)`, call `entry` and `\Tyhp\Expression::of` as ordinary methods, and build trees with `\Tyhp\Expression::of` / `\Tyhp\Expr`.

### Tyhp

**Model.** Plain class. Public properties are columns (camelCase property → snake_case column). `int $id` is a generated primary key. A property typed as another model, or `array<Other>`, is a relation. `#[Timestamps]` writes `created_at` / `updated_at`. Table name is plural snake_case of the class, or `#[Table('users')]`.

**Context.** `class AppContext extends \Tyhp\Rhetoric\Context`. `protected ?string $connection = null` is the Laravel connection name (`null` = default). `public Table<Post> $posts { get => $this->table<Post>(); }`. Override `map(\Tyhp\Rhetoric\Mapping\Mapper $map): void`.

**Mapping precedence.** Fluent mapper > attributes > conventions. `$map->model<Post>()` returns `ModelMapper<Post>`. Paths are `fn ($p) => $p->slug`. `$map->register(new PostMapping())` where `PostMapping extends ModelMapping<Post>` implements `configure(ModelMapper<Post> $model)`. `$map->conventions($c)` replaces `Conventions`. `$map->intercept($q)` adds `QueryInterceptor::beforeExecuting(string $sql, array $bindings, Connection $connection): void`.

`ModelMapper`: `table`, `comment`, `key`, `keyless`, `fromView`, `softDeletes`, `addGlobalScope`, `discriminator`, `discriminatorValue`, `observedBy`, `extraColumn<TValue>(string $name)`, `alternateKey`, `hasIndex` → `IndexMapper::unique` / `name`, `seed`, `column` → `ColumnMapper` (`name`, `maxLength`, `precision`, `generated`, `comment`, `type`, `text`, `default`, `cast`, `asJson`, `embedded`), `belongsTo` / `hasOne` / `hasMany` / `belongsToMany`. Relation mappers: `inverseOf`, `foreignKey`, `onDelete(\Tyhp\Rhetoric\OnDelete::Cascade|SetNull|Restrict)`, `referencedColumn`. Many-to-many also `pivotTable`, `pivotColumns`, `using<TPivot>()`.

Attributes in `\Tyhp\Rhetoric\Attributes`: class `Table`, `Timestamps`, `SoftDeletes`, `Keyless`, `Index<T>`, `ObservedBy`, `Discriminator`, `DiscriminatorValue`. Property `Column`, `MaxLength`, `Precision`, `Text`, `DefaultValue`, `Generated`, `Comment`, `Key`, `NotMapped`, `Cast`, `AsJson`, `Embedded`, `Deferred`, `ConcurrencyCheck`, `RowVersion`, `ForeignKey`, `InverseOf<T>`. On PHP 8.3, attributes on a hooked property are not read; set them in `map()`. `Cast<TValue, TColumn>` is `get` / `set`; `null` passes through. Built-in casts: `int`, `bool`, `\DateTimeImmutable`, backed enums. A class-typed property with a cast is a column.

**Query** (`Table<T>` extends `Query<T>`). Builders return a new query. Terminals run it. `all(): array<T>`. `get(): \Illuminate\Support\Collection<int, T>`. Untranslatable `where` / `orderBy` throws `\Tyhp\Rhetoric\Query\QueryTranslationException`. A `select` that cannot be translated runs in PHP after load.

`where(Expression<callable(T): bool>)`, `withoutGlobalScopes`, `withTrashed`, `onlyTrashed`, `with<TValue>(PropertyPath, ?Expression $constrain = null)`, `thenWith`, `singleQuery`, `select<TResult>(Expression)`, `orderBy` / `orderByDesc` / `thenBy` / `thenByDesc`, `latest(?Expression)`, `oldest(?Expression)`, `skip`, `take`, `distinct`, `tag`, `lockForUpdate`, `sharedLock`, `fromRaw(string $sql, array $bindings = [])`, `untracked`, `untrackedWithIdentity`, `join` / `leftJoin` (other query may only have `where`), `groupBy` → `Query<Group<T>>` (`$group->key`, `count`, `sum`, `min`, `max`, `avg` are SQL), `having`, `union`, `unionAll`, `intersect`, `except`. Set ops associate left to right.

Terminals: `all`, `get`, `first(): ?T`, `firstOrFail`, `sole`, `exists()`, `doesntExist()`, `count()` (no arguments; narrow with `where`), `every(fn)`, `sum`, `min`, `max`, `avg`, `toSql`, `toRawSql`, `bindings`, `paginate`, `simplePaginate`, `cursorPaginate`, `cursor`, `lazy(int $count = 1000)` (chunk generator, not lazy loading), `chunk`, `chunkById`. `find` / `findOrFail` / `reference` live on `Table`. `delete(): int`, `restore(): int`, `update(callable(Assignments<T>): mixed): int`, `increment` / `decrement`. Bulk writes throw if the query has `skip`, `take`, `distinct`, `groupBy`, `having`, a join, or a set op. Soft-delete `delete()` sets `deleted_at`.

`skip` / `take` apply before aggregates and `exists` / `every`. `latest()` / `oldest()` with no lambda use the timestamp column. `$post->blog->name` in a predicate is a join. Captured variables are bindings. `=== null` is `IS NULL`. `$col !== 'a'` matches SQL `NULL`. `tag()` must not open a nested comment on PostgreSQL.

**Load.** `with` is a separate query per relation (identity map). `singleQuery()` uses one statement, except together with `first` / `sole` / `skip` / `take`. `$db->load` / `loadMissing`. `lazyLoading(true)` enables ghost loads. `preventLazyLoading()` throws and leaves the object unloaded. `reference($id)` queries on first non-key read and loads every pending model of that class. `#[Deferred]` columns load the same way. `close()` rejects a later load. `Query::lazy` is the chunk generator.

**Save.** `add` / `addMany` / `track` / `trackAsDirty` / `remove` / `removeMany`. `save(): int` in one transaction, joining an open Laravel transaction. Parents insert before children; children delete before parents. `$db->entry<T>($model)->state` is `TrackingState` (`Added`, `Clean`, `Dirty`, `Deleted`, `Untracked`). `isDirty(?PropertyPath)`, `getOriginal(PropertyPath)`, `refresh()`, `expire()`. `$db->tracker->clear()`. `#[ConcurrencyCheck]` / `#[RowVersion]` throw `\Tyhp\Rhetoric\ConcurrencyException` on a mismatch. Observer methods: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`. `transaction<TReturn>(Closure, int $attempts = 1)`, `statement`, `affectingStatement`, `select<T>(string $sql, array $bindings = [])`. `createSchema` / `dropSchema` / `schemaHas(PropertyPath)`.

**Migrations.** `rhetoric:migration {name} --context=` writes an anonymous PHP migration (`return new class extends Migration`) under `database/migrations` and a JSON snapshot `database/rhetoric/<Context>.snapshot.json`. `--rename=table.old:new` (repeatable), `--path`, `--snapshot`, `--defer`, `--independent` (`public bool $deferred` / `$independent`). `rhetoric:remove-migration`, `rhetoric:pending` (non-zero when the map differs), `rhetoric:show {context}`, `rhetoric:baseline`. `php artisan migrate` runs the files.

**Auto-migrate.** `RHETORIC_AUTO_MIGRATE` or `auto_migrate.enabled`. Every environment except `testing`, unless `environments` is set. Trigger `boot`. `except_paths` default `['up']`. `console` is an allow-list of artisan commands; empty migrates none. `migrate`, `db:`, `schema:`, `rhetoric:`, `optimize`, `config:`, and `package:discover` are always skipped. `queue` and `octane` default true. Lock `database` (advisory, write connection on MySQL and PostgreSQL) or `cache`. Marker file `storage/framework/rhetoric`. HTTP wait 10s then 503 `Retry-After` when `on_timeout` is `throw`. `rhetoric:auto-migrate` (`--status`, `--reset`, `--connection`, `--set`), `rhetoric:migrate-deferred` (`--only`), `rhetoric:cache`, `rhetoric:clear` (also `artisan optimize` / `optimize:clear`).

**Scaffold.** `rhetoric:scaffold --context=` (required) `--namespace= --connection= --schema= --tables= --exclude= --output= --attributes --fluent --no-pluralize --force --baseline --snapshot= --eloquent`. Default mapping style is attributes unless `--fluent`. `--eloquent` needs PHP ≥ 8.4 and writes Eloquent attribute properties. Generated members live in a `Generated/` trait; the model class is rewritten only with `--force`.

**Eloquent queries.** PHP ≥ 8.4. `EloquentQuery::query<Article>()` and `on<Article>($connection)`. `where`, `orderBy`, `orderByDesc`, `get(): Eloquent\Collection`, `first(): ?T`, `builder()`. Column is `#[Column]` or snake_case property.

**`Sql::fn`.** `MethodTranslatorRegistry::shared()->registerFunction('soundex', ['mysql' => 'soundex({0})', ...])`. Keys: `mysql`, `mariadb`, `pgsql`, `sqlite`, `sqlsrv`. `{0}`… are arguments after the name, each used once. In a lambda: `Sql::fn('soundex', $post->title)`. `register` / `registerStatic` replace a method. `clear()` resets the seed (`contains`, `startsWith`, `endsWith`, `toLower`, `toUpper`, `trim`, `length`, and `Sql::like|raw|dateDiff|year|month|day|abs|round|floor|ceiling|fn`).

**`ContextFactory<T>`.** `new ContextFactory<AppContext>()->create(): AppContext`. One context per job or unit of work. `$db->tracker->clear()` if you reuse one.

### PHP

Open this section on its own. Models are plain PHP classes. The context subclasses `\Tyhp\Rhetoric\Context`. A type argument is the first call. Value arguments are the second. `use Tyhp\Type as T` when writing `T::`. `T::int` is a `\Tyhp\BuiltIn` constant. `T::int()` is a `\Tyhp\Type`. A class-string is a type argument. `\Tyhp\Generic::bind($target, T::int)` is for a class or callable chosen at runtime. `\_gnrc($target, T::int)` is the same and does not construct.

A context exposes each table as a method that calls `table`:

```php
public function posts(): \Tyhp\Rhetoric\Table
{
    return $this->table(\App\Data\Post::class)();
}
```

Call it as `$db->posts()->where(...)`. A collection relation on a plain PHP model names its model with `#[\Tyhp\PropertyType('array<App\\Data\\Post>')]`, because the `array` hint does not. A generic class is `\Vendor\Box::new(T::int)(42)` and `\Vendor\Box::type(T::int)`.

`\Tyhp\Expression::of` is an ordinary call. The class-string supplies `T`. `\_expr` is that function.

```php
\Tyhp\Expression::of(\App\Data\User::class)->age->gte(18)->and()->isActive->same(true);
\Tyhp\Expr::param('post', \App\Data\Post::class)->title->call('startsWith', 'Tyhp');
```

`Expression::of(...)->title` is a property path (`gte` / `gt` / `lte` / `lt` / `eq` / `neq` / `same` / `notSame`). `call`, `staticCall`, `plus`, `minus`, `times`, `dividedBy`, `mod`, `concat`, `not`, `neg`, `pos`, `then`, `else`, `coalesce`, `cast`, `is`, `looseEq`, `looseNeq` are `\Tyhp\Expr`. `same` / `eq` are `===`.

`map()` calls `model`: `$map->model(\App\Data\Post::class)()->belongsTo(\App\Data\Blog::class)(\Tyhp\Expression::of(\App\Data\Post::class)->blog)`. `hasOne`, `hasMany`, and `belongsToMany` take the related class the same way. `using(\App\Data\PostTag::class)()` names the pivot. `extraColumn(T::string)('legacy')` takes the value type, then the column name. `column`, `key`, `alternateKey`, `hasIndex`, `discriminator`, `inverseOf`, and `foreignKey` take the path directly. Attributes are the same classes; `#[Index(properties: ['slug'], unique: true)]` takes property-name strings.

Queries are the methods in the Tyhp section. `with`, `thenWith`, `load`, `loadMissing`, `join`, and `leftJoin` are ordinary calls. `select($resultType)($expression)` projects. `$db->select(\App\Data\Post::class)($sql, $bindings)` and `$db->transaction(T::int)($callback)` take the type first. `schemaHas` takes the property path. `all()` is a list; `get()` is a Collection. `update` takes `function (\Tyhp\Rhetoric\Query\Assignments $set): void` and `$set->assign($columnExpr, $valueExpr)`.

```php
$entry = $db->entry($post);
$factory = \Tyhp\Rhetoric\ContextFactory::new(\App\Data\ChirpContext::class)();
$query = \Tyhp\Rhetoric\EloquentQuery::query(\App\Models\Article::class)();
$onMysql = \Tyhp\Rhetoric\EloquentQuery::on(\App\Models\Article::class)('mysql');
\Tyhp\Rhetoric\Query\MethodTranslatorRegistry::shared()->registerFunction('soundex', ['sqlite' => 'soundex({0})']);
```

`entry` reads the type from the model. `EloquentQuery::query` takes no value arguments after the type. `on` takes the connection. `Context::entry` and `Tracker::entry` are `#[\Tyhp\InferGeneric]`. Artisan commands and config keys are the same as the Tyhp section. Scaffold still writes `.tyhp` files.
