# Laravel Components Cheat Sheet

> **Riferimento rapido** per le componentistiche principali di Laravel e le best practices per utilizzarle correttamente

## Indice Rapido

### 18 Componentistiche Principali
1. [Service Container (IoC Container)](#1-service-container-ioc-container)
2. [Service Provider](#2-service-provider)
3. [Service (Business Logic)](#3-service-business-logic)
4. [Repository](#4-repository)
5. [Model (Eloquent)](#5-model-eloquent)
6. [Controller](#6-controller)
7. [Middleware](#7-middleware)
8. [Form Request](#8-form-request)
9. [Resource (API Resource)](#9-resource-api-resource)
10. [Event & Listener](#10-event--listener)
11. [Job (Queue)](#11-job-queue)
12. [Policy](#12-policy)
13. [Blade Templates](#13-blade-templates)
14. [Artisan Commands](#14-artisan-commands)
15. [Migration](#15-migration)
16. [Seeder](#16-seeder)
17. [Factory](#17-factory)
18. [Test](#18-test)

### Sezioni Speciali
- [Regole Generali](#regole-generali)
- [Il Service Layer: Il Core Logico di Laravel](#il-service-layer-il-core-logico-di-laravel)
- [Best Practices Generali](#best-practices-generali)
- [Quick Reference](#quick-reference)

---

## Regole Generali

### Principi Fondamentali

#### 🎯 **Separation of Concerns**
- **Controller**: Coordina HTTP (riceve → delega → risponde)
- **Service**: Logica business, regole del dominio
- **Repository**: Accesso dati, astrazione database
- **Model**: Rappresentazione entità, relazioni

#### 🔄 **Dependency Injection**
- **Sempre** usa dependency injection per risolvere dipendenze
- **Mai** `new` o `app()` nel codice applicativo
- **Interfacce** per i binding nel container

#### 🚫 **Logica Business**
- **Solo nel Service Layer** - centralizza regole del dominio
- **Non in Controller, Middleware, Form Request, Resource, Policy, Blade, Command, Migration, Seeder, Factory**
- **Eccezioni**: Accessor/Mutator, Scopes, Eventi Model, Validation

#### 📊 **Accesso ai Dati**
- **Default**: Usa Eloquent (sufficiente per 80% dei casi)
- **Repository**: Solo per query complesse o multi-database
- **Service** → usa Model Eloquent o Repository
- **Controller** → delega al Service (eccezione: CRUD base senza logica business)
- **Altri componenti** → usa Model Eloquent o Repository

#### ⚡ **Single Responsibility**
- **Una classe, una responsabilità**
- **Un metodo, una azione**
- **Un file, un concetto**

---

## 1. Service Container (IoC Container)

### Cosa fa
- **Gestisce le dipendenze** dell'applicazione
- **Risolve automaticamente** le dipendenze tramite dependency injection
- **Registra e risolve** servizi, binding e istanze

### Best Practices
- ✅ **Registra servizi nei Service Provider** con `$this->app->bind()`
- ✅ **Usa interfacce** per i binding per maggiore flessibilità
- ❌ **Non fare binding** nel codice applicativo → **usa Service Provider**

#### Eccezioni alla Dependency Injection
- **Trait**: Non hanno costruttore → usa `app()` o metodi statici
- **Closure/Anonymous functions**: Usa `app()` per risolvere dipendenze
- **Static methods**: Usa `app()` quando necessario
- **Global helpers**: Possono usare `app()` per servizi Laravel

### Esempi pratici
```php
// ✅ CORRETTO - Dependency injection
class UserController extends Controller
{
    public function __construct(private UserService $userService) {}
}

// ❌ SBAGLIATO - Uso di new (accoppiamento forte)
class UserController extends Controller
{
    public function index()
    {
        $userService = new UserService(); // NO! Accoppiamento forte
    }
}

// ❌ SBAGLIATO - Uso di app() (accoppiamento debole)
class UserController extends Controller
{
    public function index()
    {
        $userService = app(UserService::class); // NO! Accoppiamento debole
    }
}
```

---

## 2. Service Provider

### Cosa fa
- **Configurazione per la IoC** - registra servizi nel container
- **Bootstrap dell'applicazione** - inizializza servizi all'avvio
- **Binding di interfacce** - collega contratti alle implementazioni

### Best Practices
- ✅ **Un provider per funzionalità** o package specifico
- ✅ **Usa `register()` per binding** e `boot()` per inizializzazione
- ✅ **Registra sempre interfacce** invece di classi concrete
- ✅ **Usa `singleton()` per servizi costosi** che non cambiano stato
- ❌ **Non registrare servizi** che non servono sempre → **usa lazy loading**

#### Eccezioni
- **Configurazione**: Può fare setup di configurazioni e binding complessi
- **Eventi**: Può registrare eventi e listener nel metodo `boot()`
- **View Composers**: Può registrare view composers per Blade
- **Macro**: Può registrare macro per Collection, Builder, etc.

### Esempi pratici
```php
// ✅ CORRETTO - Service Provider
class PaymentServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->bind(PaymentGatewayInterface::class, StripeGateway::class);
        $this->app->singleton(PaymentService::class);
    }
    
    public function boot()
    {
        // Inizializzazione servizi
    }
}
```

---

## 3. Service (Business Logic)

### Cosa fa
- **Implementazione della logica business** dell'applicazione
- **Orchestrazione** di repository, eventi e altri servizi
- **Separazione delle responsabilità** dal controller

### Best Practices
- ✅ **Una responsabilità per servizio** - Single Responsibility Principle
- ✅ **Lancia eccezioni specifiche** per errori business
- ✅ **Usa eventi** per comunicare con altri servizi
- ❌ **Non fare logica di presentazione** → **usa DTO o API Resources**

### Esempi pratici
```php
// ✅ CORRETTO - Service con logica business
class UserService
{
    public function __construct(
        private UserRepository $userRepository,
        private EventDispatcher $eventDispatcher
    ) {}
    
    public function createUser(array $data): User
    {
        $user = $this->userRepository->create($data);
        $this->eventDispatcher->dispatch(new UserCreated($user));
        return $user;
    }
}
```

---

## 4. Repository

### Cosa fa
- **Astrazione dell'accesso ai dati** - nasconde la complessità del database
- **Interfaccia unificata** per operazioni CRUD
- **Testabilità** - facile da mockare per i test
- **Indipendenza dall'implementazione** - puoi cambiare da Eloquent a MongoDB, API, file, etc.

### Quando usare Repository?

#### ✅ **USA Repository quando:**
- **Query business complesse** (es: `findActiveUsersWithRecentOrders()`)
- **Multi-database** (Eloquent + MongoDB + API)
- **Testing intensivo** (mocking delle query)
- **Team grandi** (separazione responsabilità)
- **Legacy systems** (integrazione sistemi esistenti)

#### ❌ **NON usare Repository quando:**
- **CRUD semplice** (create, read, update, delete)
- **Query standard** (where, orderBy, paginate)
- **Progetto piccolo** (over-engineering)
- **Solo Eloquent** (nessun multi-database)

#### ✅ **Esempi di quando serve Repository:**
```php
// ✅ CORRETTO - Query business complesse
interface UserRepositoryInterface
{
    public function findActiveUsersWithRecentOrders(): Collection;
    public function findUsersByRoleAndLocation(string $role, string $location): Collection;
}

class EloquentUserRepository implements UserRepositoryInterface
{
    public function findActiveUsersWithRecentOrders(): Collection
    {
        return User::where('is_active', true)
            ->whereHas('orders', function ($query) {
                $query->where('created_at', '>=', now()->subDays(30));
            })
            ->with(['orders' => function ($query) {
                $query->latest()->limit(5);
            }])
            ->get();
    }
}

// ✅ CORRETTO - Multi-database
class MongoUserRepository implements UserRepositoryInterface
{
    public function findById(int $id): ?User
    {
        $userData = $this->mongoCollection->findOne(['_id' => $id]);
        return $userData ? User::fromArray($userData) : null;
    }
}
```

### Best Practices
- ✅ **Un repository per entità** o aggregate
- ✅ **Usa interfacce** per i contratti
- ✅ **Implementa query specifiche** come metodi del repository
- ❌ **Non accedere a più tabelle** non correlate → **usa Unit of Work Pattern**

### Perché Laravel privilegia Eloquent?

**Filosofia Laravel**: "Convention over Configuration" e semplicità. Eloquent fornisce già astrazione sufficiente per l'80% dei casi d'uso reali.

**Vantaggi Eloquent:**
- **Produttività** - sviluppo rapido senza configurazioni
- **Leggibilità** - `$user->posts()` è più chiaro di `$userRepository->getPosts($user)`
- **Ecosistema** - integrazione perfetta con Migration, Seeder, Factory
- **Manutenibilità** - meno codice da mantenere

**Repository aggiunge valore solo quando:**
- **Query complesse** che non si adattano bene a Eloquent
- **Multi-database** con logiche diverse
- **Testing intensivo** con mocking specifico
- **Team grandi** con separazione netta delle responsabilità

---

## 5. Model (Eloquent)

### Cosa fa
- **Rappresentazione delle entità** del database
- **Active Record pattern** - ogni istanza rappresenta una riga
- **Gestione delle relazioni** tra entità

### Best Practices
- ✅ **Un model per tabella** del database
- ✅ **Usa `$fillable` o `$guarded`** per la mass assignment protection
- ✅ **Definisci relazioni** come metodi
- ✅ **Usa accessors e mutators** per trasformare dati
- ✅ **Usa scopes** per query riutilizzabili

#### Eccezioni alla Logica Business
- **Accessor/Mutator**: Può fare logica di trasformazione dati
- **Scopes**: Può fare query complesse per il model stesso
- **Relazioni**: Può accedere a tabelle correlate tramite relazioni
- **Eventi Model**: Può fare logica in `creating`, `updating`, `deleting`
- **Validation**: Può fare validazione specifica del model

### Esempi pratici
```php
// ✅ CORRETTO - Model con relazioni e scopes
class User extends Model
{
    protected $fillable = ['name', 'email', 'password'];
    
    protected $casts = [
        'email_verified_at' => 'datetime',
    ];
    
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
    
    public function scopeActive($query)
    {
        return $query->where('is_active', true);
    }
}
```

### Perché Laravel usa Eloquent?

**Eloquent è il cuore dell'accesso ai dati in Laravel** per diverse ragioni strategiche:

#### **Active Record Pattern**
- **Semplicità** - Ogni model rappresenta una riga del database
- **Intuitività** - `$user->name` è più chiaro di `$user['name']`
- **Produttività** - Sviluppo rapido senza configurazioni complesse

#### **Query Builder integrato**
- **Fluent interface** - `User::where('active', true)->get()`
- **Relazioni eleganti** - `$user->posts()->where('published', true)`
- **Eager loading** - `User::with('posts')->get()`

#### **Filosofia Laravel**
- **Convention over Configuration** - Convenzioni sensate di default
- **Developer Experience** - Focus sulla produttività del developer
- **Ecosistema** - Integrazione perfetta con Migration, Seeder, Factory

#### **Vantaggi rispetto ad altri ORM**
- **Laravel-first** - Progettato specificamente per Laravel
- **Documentazione eccellente** - Esempi chiari e completi
- **Community** - Supporto vasto e risorse abbondanti
- **Evoluzione** - Aggiornamenti costanti e miglioramenti

**Eloquent bilancia perfettamente semplicità e potenza**, rendendo l'accesso ai dati naturale e produttivo senza sacrificare flessibilità.

---

## 6. Controller

### Cosa fa
- **Coordina le richieste HTTP** - riceve e delega
- **Costruisce le risposte HTTP** - formatta output
- **Gestisce logica HTTP pura** - redirect, headers, status codes

### Best Practices
- ✅ **Controller = Coordinatore HTTP** - solo riceve e delega
- ✅ **Usa Resource Controllers** per operazioni CRUD standard
- ✅ **Usa Form Request** per validazione
- ✅ **Restituisci Resource** per API consistenti
- ✅ **Un controller per risorsa** - gestisce una entità specifica
- ❌ **Non fare logica business** nei controller → **usa Service Layer**

#### Eccezioni (Logica HTTP pura)
- **CRUD base**: Operazioni banali senza regole business (es: `User::paginate()`)
- **Logica HTTP**: Redirect, response headers, status codes
- **File Upload**: Validazione tecnica (tipo, dimensione, estensione)
- **Paginazione semplice**: Limit, offset, filtri base
- **Autenticazione HTTP**: Login/logout senza logica complessa

### Esempi pratici
```php
// ✅ CORRETTO - Controller magro (delega al Service)
class UserController extends Controller
{
    public function __construct(private UserService $userService) {}
    
    public function store(CreateUserRequest $request)
    {
        $user = $this->userService->createUser($request->validated());
        return new UserResource($user);
    }
}

// ✅ CORRETTO - Eccezione: CRUD base senza logica business
class UserController extends Controller
{
    public function index(Request $request)
    {
        $users = User::paginate(15); // Operazione banale
        return UserResource::collection($users);
    }
}

// ❌ SBAGLIATO - Logica business nel Controller
class UserController extends Controller
{
    public function store(Request $request)
    {
        // NO! Logica business nel Controller
        $role = $request->email === 'admin@example.com' ? 'admin' : 'user';
        $user = User::create([...]);
        Mail::to($user)->send(new WelcomeEmail());
    }
}
```

---

## 7. Middleware

### Cosa fa
- **Filtra le richieste HTTP** prima che raggiungano il controller
- **Applica logica cross-cutting** (auth, CORS, logging)
- **Modifica request/response** in modo trasparente

### Best Practices
- ✅ **Una responsabilità per middleware** - Single Responsibility
- ✅ **Usa middleware groups** per raggruppare logica correlata
- ✅ **Applica middleware** a route specifiche quando possibile
- ✅ **Restituisci response** o chiama `$next()`
- ❌ **Non fare logica business** nel middleware → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Model Eloquent o Repository**

#### Eccezioni (Middleware che possono accedere al database)
- **Authentication**: Può accedere al database per verificare utenti
- **Authorization**: Può fare query per verificare permessi
- **Logging**: Può accedere al database per log delle richieste
- **Rate Limiting**: Può accedere a cache/database per contatori
- **CORS**: Può gestire headers e configurazioni complesse
- **Complex Middleware**: Può gestire più aspetti correlati (auth + logging) se ben progettato

### Esempi pratici
```php
// ✅ CORRETTO - Middleware per autenticazione
class AuthenticateMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        if (!Auth::check()) {
            return redirect()->route('login');
        }
        
        return $next($request);
    }
}
```

---

## 8. Form Request

### Cosa fa
- **Validazione centralizzata** dei dati di input
- **Autorizzazione** delle richieste
- **Sanitizzazione** dei dati

### Best Practices
- ✅ **Una Form Request per endpoint** specifico
- ✅ **Usa regole di validazione** appropriate
- ✅ **Implementa `authorize()`** per controlli di accesso
- ✅ **Usa `messages()`** per messaggi personalizzati
- ❌ **Non fare logica business** nella Form Request → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Model Eloquent o Repository**

### Esempi pratici
```php
// ✅ CORRETTO - Form Request con validazione
class CreateUserRequest extends FormRequest
{
    public function authorize()
    {
        return Auth::user()->can('create', User::class);
    }
    
    public function rules()
    {
        return [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:8|confirmed',
        ];
    }
}
```

---

## 9. Resource (API Resource)

### Cosa fa
- **Trasforma i modelli** in array per API
- **Nasconde i dettagli** interni del database
- **Fornisce API consistenti** e versionabili

### Best Practices
- ✅ **Una Resource per entità** specifica
- ✅ **Usa `with()`** per includere relazioni
- ✅ **Usa `when()`** per dati condizionali
- ✅ **Usa `makeHidden()`** per nascondere campi sensibili
- ❌ **Non fare logica business** nelle Resource → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Model Eloquent o Repository**

### Esempi pratici
```php
// ✅ CORRETTO - Resource per API
class UserResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at->toISOString(),
            'posts' => PostResource::collection($this->whenLoaded('posts')),
        ];
    }
}
```

---

## 10. Event & Listener

### Cosa fa
- **Decoupling** tra componenti dell'applicazione
- **Comunicazione asincrona** tra servizi
- **Logica reattiva** basata su eventi

### Best Practices
- ✅ **Eventi per azioni** che sono successe (UserCreated, OrderPaid)
- ✅ **Listener per reazioni** agli eventi (SendEmail, UpdateInventory)
- ✅ **Usa Job** per listener che richiedono tempo
- ✅ **Eventi immutabili** - non modificare lo stato
- ❌ **Non fare logica business** negli eventi → **usa Service Layer**
- ❌ **Non accedere direttamente** al database nei listener → **usa Repository Pattern**

### Esempi pratici
```php
// ✅ CORRETTO - Event e Listener
class UserCreated
{
    public function __construct(public User $user) {}
}

class SendWelcomeEmail
{
    public function handle(UserCreated $event)
    {
        Mail::to($event->user->email)->send(new WelcomeEmail($event->user));
    }
}
```

---

## 11. Job (Queue)

### Cosa fa
- **Elaborazione asincrona** di task pesanti
- **Gestione delle code** per distribuire il carico
- **Retry automatico** per operazioni fallite

### Best Practices
- ✅ **Job per operazioni** che richiedono tempo
- ✅ **Implementa `failed()`** per gestire errori
- ✅ **Usa `timeout` e `tries`** appropriati
- ✅ **Job idempotenti** - possono essere eseguiti più volte
- ❌ **Non fare operazioni** che richiedono interazione utente → **usa Controller o Livewire**
- ❌ **Non accedere a sessioni** o request → **usa dependency injection**

#### Eccezioni
- **Batch Jobs**: Possono gestire più operazioni correlate
- **Chained Jobs**: Possono dipendere da altri job
- **Scheduled Jobs**: Possono accedere a configurazioni globali
- **Notification Jobs**: Possono gestire più canali di notifica
- **Report Jobs**: Possono generare report complessi
- **Cleanup Jobs**: Possono fare operazioni di pulizia del database

### Esempi pratici
```php
// ✅ CORRETTO - Job per invio email
class SendEmailJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
    public function __construct(private User $user, private string $template) {}
    
    public function handle()
    {
        Mail::to($this->user->email)->send(new $this->template($this->user));
    }
    
    public function failed(Throwable $exception)
    {
        Log::error('Email sending failed', ['user_id' => $this->user->id]);
    }
}
```

---

## 12. Policy

### Cosa fa
- **Autorizzazione granulare** per risorse specifiche
- **Logica di accesso** centralizzata
- **Controllo permessi** basato su contesto

### Best Practices
- ✅ **Una Policy per modello** o risorsa
- ✅ **Metodi specifici** per azioni (view, create, update, delete)
- ✅ **Usa `before()`** per controlli globali
- ✅ **Policy per logica complessa**, Gates per logica semplice
- ❌ **Non fare logica business** nelle Policy → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Model Eloquent o Repository**

### Esempi pratici
```php
// ✅ CORRETTO - Policy per autorizzazione
class PostPolicy
{
    public function view(User $user, Post $post)
    {
        return $post->is_published || $user->id === $post->user_id;
    }
    
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
    
    public function before(User $user, string $ability)
    {
        if ($user->isAdmin()) {
            return true;
        }
    }
}
```

---

## 13. Blade Templates

### Cosa fa
- **Sistema di templating** per le view
- **Ereditarietà** e composizione di template
- **Componenti riutilizzabili** per UI

### Best Practices
- ✅ **Layout principale** per struttura comune
- ✅ **Componenti** per elementi riutilizzabili
- ✅ **Sections** per contenuto specifico
- ✅ **Directives personalizzate** per logica complessa
- ❌ **Non fare logica business** nei template → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Model Eloquent o Repository**

### Esempi pratici
```php
// ✅ CORRETTO - Blade component
class UserCard extends Component
{
    public function __construct(public User $user) {}
    
    public function render()
    {
        return view('components.user-card');
    }
}
```

---

## 14. Artisan Commands

### Cosa fa
- **Comandi CLI** per automatizzare task
- **Generazione codice** e file boilerplate
- **Operazioni di manutenzione** dell'applicazione

### Best Practices
- ✅ **Un comando per task** specifico
- ✅ **Usa `handle()`** per la logica principale
- ✅ **Opzioni e argomenti** per configurazione
- ✅ **Output informativo** per l'utente
- ❌ **Non fare logica business** nei comandi → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Model Eloquent o Repository**

### Esempi pratici
```php
// ✅ CORRETTO - Artisan Command con Service Layer
class CreateUserCommand extends Command
{
    protected $signature = 'user:create {name} {email}';
    protected $description = 'Create a new user';
    
    public function __construct(private UserService $userService)
    {
        parent::__construct();
    }
    
    public function handle()
    {
        $user = $this->userService->createUser([
            'name' => $this->argument('name'),
            'email' => $this->argument('email'),
        ]);
        
        $this->info("User created: {$user->name}");
    }
}
```

---

## 15. Migration

### Cosa fa
- **Versionamento del database** - controlla le modifiche
- **Sincronizzazione** tra ambienti
- **Rollback** delle modifiche

### Best Practices
- ✅ **Una migration per modifica** specifica
- ✅ **Nomi descrittivi** per le migration
- ✅ **Implementa `down()`** per rollback
- ✅ **Usa metodi specifici** (create, add, modify)
- ❌ **Non modificare** migration già eseguite → **crea nuova migration**
- ❌ **Non fare logica business** nelle migration → **usa Seeder o Command**

### Esempi pratici
```php
// ✅ CORRETTO - Migration per creazione tabella
class CreateUsersTable extends Migration
{
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->timestamps();
        });
    }
    
    public function down()
    {
        Schema::dropIfExists('users');
    }
}
```

---

## 16. Seeder

### Cosa fa
- **Popolamento del database** con dati di test
- **Dati di base** per l'applicazione
- **Dati di esempio** per sviluppo

### Best Practices
- ✅ **Un seeder per modello** specifico
- ✅ **Usa Model Factories** per dati casuali
- ✅ **Dati consistenti** e riproducibili
- ✅ **Usa `call()`** per chiamare altri seeder
- ❌ **Non fare logica business** nei seeder → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Model o Repository**

### Esempi pratici
```php
// ✅ CORRETTO - Seeder con Factory
class UserSeeder extends Seeder
{
    public function run()
    {
        User::factory(10)->create();
        
        User::factory()->create([
            'name' => 'Admin User',
            'email' => 'admin@example.com',
        ]);
    }
}
```

---

## 17. Factory

### Cosa fa
- **Generazione dati** per test e seeder
- **Dati realistici** per sviluppo
- **Varianti** di dati per test

### Best Practices
- ✅ **Una factory per modello** specifico
- ✅ **Dati realistici** e variabili
- ✅ **Usa `faker`** per dati casuali
- ✅ **Stati specifici** per scenari diversi
- ❌ **Non fare logica business** nelle factory → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Model o Repository**

### Esempi pratici
```php
// ✅ CORRETTO - Factory con stati
class UserFactory extends Factory
{
    public function definition()
    {
        return [
            'name' => $this->faker->name(),
            'email' => $this->faker->unique()->safeEmail(),
            'password' => Hash::make('password'),
        ];
    }
    
    public function admin()
    {
        return $this->state([
            'role' => 'admin',
        ]);
    }
}
```

---

## 18. Test

### Cosa fa
- **Verifica del comportamento** dell'applicazione
- **Prevenzione regressioni** durante lo sviluppo
- **Documentazione vivente** del codice

### Best Practices
- ✅ **Test per ogni funzionalità** importante
- ✅ **Test isolati** - non dipendenti tra loro
- ✅ **Usa mocks** per dipendenze esterne
- ✅ **Nomi descrittivi** per i test
- ❌ **Non testare** codice di terze parti → **usa integration test**
- ❌ **Non fare test** troppo complessi → **usa Test Data Builder**

### Esempi pratici
```php
// ✅ CORRETTO - Test con mock
class UserServiceTest extends TestCase
{
    public function test_creates_user_successfully()
    {
        $userRepository = Mockery::mock(UserRepositoryInterface::class);
        $userRepository->shouldReceive('create')->once()->andReturn(new User());
        
        $userService = new UserService($userRepository);
        $result = $userService->createUser(['name' => 'John']);
        
        $this->assertInstanceOf(User::class, $result);
    }
}
```

---

## Il Service Layer: Il Core Logico di Laravel

### Perché il Service è il Core dell'Applicazione

Il **Service Layer** rappresenta il **core logico** della tua applicazione Laravel. È qui che vive tutta la logica business, le regole del dominio e l'intelligenza dell'applicazione.

### Architettura a Livelli con Service al Centro

#### Flusso Principale HTTP
```
┌─────────────────┐   ┌───────────┐   ┌───────────────┐   ┌──────────────┐
│ HTTP Request    │ → │   Route   │ → │   Middleware  │ → │ Form Request │
│ (GET/POST/etc)  │   └─────┬─────┘   └───────┬───────┘   └──────┬───────┘
└─────────────────┘         │                 │                  │
                            │                 │                  │
                            └─────────────────┴─────────┬────────┘
                                                        ▼
                                                ┌───────────────┐
                                                │  Controller   │
                                                └───────┬───────┘
                                                        ├────────────────┐
                                                        ▼                │
                                         ┌─────────────────────┐         │
                                         │   Service Layer     │         │
                                         │  (Business Logic)   │         │
                                         └────────┬────────────┘         │
                                                  │                      │
                                                  ├─→ Emette Evento      │
                                                  │   (Dominio)          │
                                                  │        │             │
                                                  │        ▼             │
                                                  │  ┌─────────────┐     │
                                                  │  │  Listener   │     │
                                                  │  └──────┬──────┘     │
                                                  │         ▼            │
                                                  │  ┌─────────────┐     │
                                                  │  │ Altri       │     │
                                                  │  │ Service     │     │
                                                  ▼  └─────────────┘     │
                                         ┌────────▼─────┐                │
                                         │  Data Layer  │                │
                                         │  Model/Repo  │                │
                                         └────────┬─────┘                │
                                                  │                      │
                                                  ├─→ Emette Evento      │
                                                  │   (Dati)             │
                                                  │        │             │
                                                  │        ▼             │
                                                  │  ┌─────────────┐     │
                                                  │  │  Listener   │     │
                                                  │  └──────┬──────┘     │
                                                  │         ▼            │
                                                  │  ┌─────────────┐     │
                                                  │  │ Altri       │     │
                                                  │  │ Service     │     │
                                                  ▼  └─────────────┘     │
                                         ┌──────────────────────────┐    │
                                         │      Response Layer      │←───┘
                                         │ (Resource/View/Redirect) │
                                         └────────┬─────────────────┘
                                                  ▼
                                         ┌──────────────────────────┐
                                         │       HTTP RESPONSE      │
                                         │  (JSON/HTML/StatusCode)  │
                                         └──────────────────────────┘

```
**Legenda:**
- **Evento (Dominio)**: Eventi legati alla business logic (es: OrderShipped, UserRegistered)
- **Evento (Dati)**: Eventi legati a operazioni CRUD (es: UserCreated, PostUpdated)
- **Listener**: Reagiscono agli eventi e coordinano altri service
- **Response Layer**: Trasforma i dati in formato adatto (JSON/HTML)

**Note Importanti:**
1. Il Controller può bypassare il Service Layer per operazioni semplici
2. I Listener devono essere leggeri e delegare la logica complessa ai Service
3. Gli eventi possono essere sincroni o asincroni (configurabili in `EventServiceProvider`)
4. La Response Layer gestisce anche redirect e errori

### Flusso Tipico con Service al Centro

1. **HTTP Request arriva al server**
   - Input: Richiesta HTTP (URL, metodo, headers, body)
   - Output: Richiesta processata

2. **Route definisce URL e metodo**
   - Input: Richiesta HTTP
   - Output: Controller e metodo specifico da chiamare

3. **Middleware filtra la richiesta**
   - Input: Richiesta HTTP + Controller destinazione
   - Output: Richiesta filtrata (auth, CORS, logging, etc.)

4. **Form Request valida i dati** (se presente)
   - Input: Richiesta filtrata + dati da validare
   - Output: Dati validati e sanitizzati

5. **Controller riceve la richiesta**
   - Input: Richiesta validata + parametri
   - Output: Chiamata al Service Layer

6. **Controller chiama Service**
   - Input: Dati validati + parametri business
   - Output: Delegazione logica business

7. **Service applica logica business**
   - Input: Dati business + regole del dominio
   - Output: Risultato business + eventi emessi

8. **Service coordina Repository per i dati**
   - Input: Operazioni CRUD + query business
   - Output: Dati dal database + eventi di dati

9. **Service lancia eventi per comunicare**
   - Input: Eventi di dominio + eventi di dati
   - Output: Notifiche a Listener

10. **Listener gestiscono le conseguenze**
    - Input: Eventi ricevuti
    - Output: Azioni reattive (email, log, cache, etc.)

11. **Service restituisce risultato**
    - Input: Dati processati + logica applicata
    - Output: Entità del dominio o DTO

12. **Controller restituisce Resource**
    - Input: Entità del dominio
    - Output: Dati formattati per API (JSON/XML)

13. **Response HTTP al client**
    - Input: Dati formattati + headers
    - Output: Risposta HTTP completa (status, body, headers)

#### Caratteristiche Chiave

- **Service Layer** = **Core Logico** accessibile da qualsiasi entry point
- **Data Layer** = Accesso ai dati (Model Eloquent o Repository), chiamabile da Service o direttamente
- **Controller** = Solo uno dei possibili entry point, non l'unico
- **Flessibilità** = Ogni entry point può accedere ai layer sottostanti

### Caratteristiche del Service Layer

#### Cosa fa il Service:
- **Centralizza la logica business** - Tutte le regole del dominio
- **Orchestra le operazioni** - Coordina repository, eventi, altri servizi
- **Mantiene la coerenza** - Applica regole uniformemente
- **Gestisce transazioni complesse** - Operazioni multi-step
- **Comunica tramite eventi** - Decoupling con altri servizi
- **È indipendente dal framework** - Testabile e riutilizzabile

#### Cosa NON fa il Service:
- **Non gestisce HTTP** → usa Controller
- **Non accede direttamente al DB** → usa Model o Repository
- **Non formatta dati per API** → usa Resource
- **Non gestisce validazione input** → usa Form Request
- **Non gestisce autenticazione** → usa Middleware/Policy

### Esempio Pratico: Service come Core

```php
// ❌ SBAGLIATO - Logica sparsa nel Controller
class UserController extends Controller
{
    public function createUser(Request $request)
    {
        // Validazione (dovrebbe essere in Form Request)
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:8|confirmed'
        ]);
        
        // Logica business nel Controller (SBAGLIATO!)
        $role = $request->email === 'admin@example.com' ? 'admin' : 'user';
        $status = $request->has('company') ? 'verified' : 'pending';
        
        // Creazione utente
        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
            'role' => $role,
            'status' => $status,
        ]);
        
        // Invio email (logica business)
        if ($status === 'pending') {
            Mail::to($user->email)->send(new VerificationEmail($user));
        }
        
        // Log attività
        Log::info('User created', ['user_id' => $user->id]);
        
        return response()->json($user);
    }
}

// ✅ CORRETTO - Service come core logico
class UserController extends Controller
{
    public function __construct(private UserService $userService) {}
    
    public function createUser(CreateUserRequest $request)
    {
        $user = $this->userService->createUser($request->validated());
        return new UserResource($user);
    }
}

class UserService
{
    public function __construct(
        private UserRepository $userRepository,
        private EventDispatcher $eventDispatcher,
        private LogService $logService
    ) {}
    
    public function createUser(array $data): User
    {
        // Logica business centralizzata
        $data['role'] = $this->determineUserRole($data['email']);
        $data['status'] = $this->determineUserStatus($data);
        $data['password'] = Hash::make($data['password']);
        
        $user = $this->userRepository->create($data);
        
        // Eventi per comunicare con altri servizi
        $this->eventDispatcher->dispatch(new UserCreated($user));
        
        // Log centralizzato
        $this->logService->logUserCreation($user);
        
        return $user;
    }
    
    private function determineUserRole(string $email): string
    {
        return $email === 'admin@example.com' ? 'admin' : 'user';
    }
    
    private function determineUserStatus(array $data): string
    {
        return isset($data['company']) ? 'verified' : 'pending';
    }
}

// Listener per gestire le conseguenze
class SendVerificationEmail
{
    public function handle(UserCreated $event)
    {
        if ($event->user->status === 'pending') {
            Mail::to($event->user->email)->send(new VerificationEmail($event->user));
        }
    }
}
```

### Vantaggi del Service come Core

#### Centralizzazione
- **Una fonte di verità** per la logica business
- **Coerenza** tra Controller, Command, Job
- **Manutenibilità** - modifiche in un posto solo

#### Riusabilità
- **Stesso Service** da Controller, Command, Job
- **API unificata** per operazioni business
- **Testabilità** - facile da testare in isolamento

#### Separazione delle Responsabilità
- **Controller**: Gestisce HTTP
- **Service**: Logica business
- **Repository**: Accesso dati
- **Model**: Rappresentazione entità

#### Flessibilità
- **Cambiamenti** senza impatto su altri layer
- **Evoluzione** del dominio senza rotture
- **Testing** indipendente per ogni layer

### Best Practices per il Service Layer

#### ✅ **DO (Cosa Fare):**
- **Una responsabilità per Service** - Single Responsibility
- **Usa dependency injection** per le dipendenze
- **Lancia eventi** per comunicare con altri servizi
- **Gestisci transazioni** complesse
- **Restituisci entità del dominio** o DTO
- **Usa eccezioni specifiche** per errori business

#### ❌ **DON'T (Cosa Non Fare):**
- **Non gestire HTTP** → usa Controller
- **Non accedere direttamente al DB** → usa Repository
- **Non formattare per API** → usa Resource
- **Non gestire validazione** → usa Form Request
- **Non gestire autenticazione** → usa Middleware/Policy

### Quando Creare un Service

#### ✅ **Crea un Service quando:**
- Hai **logica business** complessa
- Devi **coordinare** più repository
- Hai **regole del dominio** da applicare
- Devi **gestire transazioni** complesse
- Hai **operazioni** che coinvolgono più entità

#### ❌ **Non creare un Service quando:**
- Hai solo **operazioni CRUD** semplici
- Non c'è **logica business** reale
- Stai solo **passando dati** tra layer
- Hai solo **validazione** di input

### Esempio di Service Completo

```php
class OrderService
{
    public function __construct(
        private OrderRepository $orderRepository,
        private PaymentService $paymentService,
        private InventoryService $inventoryService,
        private NotificationService $notificationService,
        private EventDispatcher $eventDispatcher
    ) {}
    
    public function createOrder(array $orderData, User $user): Order
    {
        // Validazione business
        $this->validateOrderData($orderData);
        
        // Calcoli business
        $orderData['total'] = $this->calculateOrderTotal($orderData['items']);
        $orderData['status'] = 'pending_payment';
        $orderData['user_id'] = $user->id;
        
        // Transazione complessa
        DB::transaction(function () use ($orderData, &$order) {
            // Creazione ordine
            $order = $this->orderRepository->create($orderData);
            
            // Riserva inventario
            $this->inventoryService->reserveItems($orderData['items']);
            
            // Lancia evento
            $this->eventDispatcher->dispatch(new OrderCreated($order));
        });
        
        return $order;
    }
    
    public function processPayment(Order $order, PaymentData $paymentData): PaymentResult
    {
        // Logica business per pagamento
        $paymentResult = $this->paymentService->processPayment($paymentData, $order->total);
        
        if ($paymentResult->isSuccessful()) {
            // Aggiorna stato ordine
            $this->orderRepository->update($order->id, ['status' => 'paid']);
            
            // Conferma inventario
            $this->inventoryService->confirmReservation($order->items);
            
            // Notifica utente
            $this->notificationService->sendOrderConfirmation($order);
            
            // Evento
            $this->eventDispatcher->dispatch(new OrderPaid($order));
        }
        
        return $paymentResult;
    }
    
    private function validateOrderData(array $data): void
    {
        if (empty($data['items'])) {
            throw new InvalidOrderException('Order must have at least one item');
        }
        
        if ($data['total'] <= 0) {
            throw new InvalidOrderException('Order total must be greater than zero');
        }
    }
    
    private function calculateOrderTotal(array $items): float
    {
        return collect($items)->sum(function ($item) {
            return $item['price'] * $item['quantity'];
        });
    }
}
```

### Conclusione

Il **Service Layer** è il **core logico** della tua applicazione Laravel. È qui che vive l'intelligenza del dominio, dove si prendono le decisioni importanti e dove si coordina tutto il sistema.

**Ricorda**: Se hai logica business, va nel Service. Se non hai logica business, probabilmente non ti serve un Service.

---

## Best Practices Generali

### 1. Separation of Concerns
- **Controller**: Gestisce HTTP, coordina servizi
- **Service**: Logica business, orchestrazione
- **Repository**: Accesso dati, astrazione database
- **Model**: Rappresentazione entità, relazioni

### 2. Dependency Injection
- **Sempre** usa dependency injection
- **Mai** `new` o `app()` nel codice applicativo
- **Interfacce** per i binding nel container

#### Eccezioni alla Dependency Injection
- **Trait**: Non hanno costruttore → usa `app()` o metodi statici
- **Closure/Anonymous functions**: Usa `app()` per risolvere dipendenze
- **Static methods**: Usa `app()` quando necessario
- **Global helpers**: Possono usare `app()` per servizi Laravel

```php
// ✅ CORRETTO - Trait con app()
trait LogsActivity
{
    protected function logActivity(string $action, $model = null)
    {
        $logger = app(ActivityLogger::class); // OK nei trait
        $logger->log($action, $model);
    }
}

// ✅ CORRETTO - Static method con app()
class Helper
{
    public static function sendEmail(string $email, string $message)
    {
        $mailer = app(MailService::class); // OK nei metodi statici
        return $mailer->send($email, $message);
    }
}
```

### 3. Single Responsibility
- **Una classe, una responsabilità**
- **Un metodo, una azione**
- **Un file, un concetto**

#### Eccezioni alla Single Responsibility
- **Controller**: Può gestire più azioni correlate (CRUD)
- **Model**: Può avere accessor/mutator + relazioni + scopes
- **Service Provider**: Può registrare più servizi correlati
- **Middleware**: Può gestire più aspetti della stessa richiesta

### 4. Don't Repeat Yourself (DRY)
- **Estrai** logica comune in servizi
- **Usa** ereditarietà e composizione
- **Crea** utility e helper riutilizzabili

### 5. Keep It Simple (KISS)
- **Semplice** è meglio di complesso
- **Leggibile** è meglio di intelligente
- **Chiaro** è meglio di compatto

### 6. You Aren't Gonna Need It (YAGNI)
- **Implementa** solo quello che serve ora
- **Non** pre-ottimizzare
- **Non** aggiungere funzionalità "per il futuro"

### 7. Fail Fast
- **Valida** input il prima possibile
- **Lancia** eccezioni specifiche
- **Gestisci** errori in modo esplicito

### 8. Test-Driven Development
- **Scrivi** test prima del codice
- **Refactoring** continuo
- **Copertura** adeguata dei test

---

## Quick Reference

### Flusso tipico di una richiesta
1. **HTTP Request** → arriva al server
2. **Route** → definisce URL e metodo
3. **Middleware** → filtra la richiesta
4. **Form Request** → valida i dati (se presente)
5. **Controller** → riceve la richiesta
6. **Service** → esegue logica business (se necessario)
7. **Model/Repository** → accede ai dati
8. **Resource** → trasforma per API
9. **Response** → restituisce al client

### Componenti per responsabilità
- **HTTP**: Controller, Middleware, Form Request
- **Business**: Service, Event, Listener
- **Data**: Model, Repository, Migration
- **API**: Resource, API Resource
- **Background**: Job, Queue
- **Auth**: Policy, Gate, Middleware
- **UI**: Blade, Component, View
- **CLI**: Command, Schedule

### Pattern di utilizzo
- **CRUD**: Controller + Service + Repository + Model
- **API**: Resource + Form Request + Service
- **Background**: Job + Event + Listener
- **Auth**: Policy + Middleware + Gate
- **UI**: Blade + Component + View

---

*Questo cheat sheet fornisce una panoramica rapida delle componentistiche principali di Laravel e le best practices per utilizzarle correttamente. Per implementazioni dettagliate e esempi completi, consulta i pattern specifici nella documentazione.*
