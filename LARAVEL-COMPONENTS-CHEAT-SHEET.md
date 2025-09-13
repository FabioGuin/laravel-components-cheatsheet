# Laravel Components Cheat Sheet

> **Riferimento rapido** per le componentistiche principali di Laravel e le regole d'oro per utilizzarle correttamente

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
- [Regole d'Oro Generali](#regole-doro-generali)
- [Quick Reference](#quick-reference)

---

## 1. Service Container (IoC Container)

### Cosa fa
- **Gestisce le dipendenze** dell'applicazione
- **Risolve automaticamente** le dipendenze tramite dependency injection
- **Registra e risolve** servizi, binding e istanze

### Regole d'oro
- ✅ **Usa sempre dependency injection** invece di `new` o `app()`
- ✅ **Registra servizi nei Service Provider** con `$this->app->bind()`
- ✅ **Usa interfacce** per i binding per maggiore flessibilità
- ❌ **Non usare mai** `app()` direttamente nei controller o servizi → **usa dependency injection**
- ❌ **Non fare binding** nel codice applicativo, solo nei provider → **usa Service Provider**

#### Eccezioni
- **Trait**: Non hanno costruttore → usa `app()` o metodi statici
- **Closure/Anonymous functions**: Usa `app()` per risolvere dipendenze
- **Static methods**: Usa `app()` quando necessario
- **Global helpers**: Possono usare `app()` per servizi Laravel
- **Service Provider**: Può usare `$this->app` per binding

### Esempi pratici
```php
// ✅ CORRETTO - Dependency injection
class UserController extends Controller
{
    public function __construct(private UserService $userService) {}
}

// ❌ SBAGLIATO - Uso diretto di app()
class UserController extends Controller
{
    public function index()
    {
        $userService = app(UserService::class); // NO!
    }
}
```

---

## 2. Service Provider

### Cosa fa
- **Configurazione per la IoC** - registra servizi nel container
- **Bootstrap dell'applicazione** - inizializza servizi all'avvio
- **Binding di interfacce** - collega contratti alle implementazioni

### Regole d'oro
- ✅ **Un provider per funzionalità** o package specifico
- ✅ **Usa `register()` per binding** e `boot()` per inizializzazione
- ✅ **Registra sempre interfacce** invece di classi concrete
- ✅ **Usa `singleton()` per servizi costosi** che non cambiano stato
- ❌ **Non fare logica business** nei provider → **usa Service Layer**
- ❌ **Non registrare servizi** che non servono sempre → **usa lazy loading**

#### Eccezioni
- **Configurazione**: Può fare setup di configurazioni e binding complessi
- **Eventi**: Può registrare eventi e listener nel metodo `boot()`
- **View Composers**: Può registrare view composers per Blade
- **Macro**: Può registrare macro per Collection, Builder, etc.
- **Package Provider**: Può registrare più servizi correlati al package

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

### Regole d'oro
- ✅ **Una responsabilità per servizio** - Single Responsibility Principle
- ✅ **Usa dependency injection** per le dipendenze
- ✅ **Lancia eccezioni specifiche** per errori business
- ✅ **Usa eventi** per comunicare con altri servizi
- ❌ **Non accedere direttamente** al database → **usa Repository Pattern**
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

### Regole d'oro
- ✅ **Un repository per entità** o aggregate
- ✅ **Usa interfacce** per i contratti
- ✅ **Restituisci DTO o entità di dominio** per vera astrazione
- ✅ **Implementa query specifiche** come metodi del repository
- ❌ **Non fare logica business** nel repository → **usa Service Layer**
- ❌ **Non accedere a più tabelle** non correlate → **usa Unit of Work Pattern**

#### Eccezioni
- **Laravel Standard**: Può restituire modelli Eloquent per semplicità
- **Legacy Code**: Può usare modelli Eloquent per compatibilità
- **Simple Apps**: Per applicazioni semplici, modelli Eloquent sono accettabili
- **API Resources**: Può restituire modelli Eloquent se usi API Resources

### Esempi pratici
```php
// ✅ CORRETTO - Repository con DTO (vera astrazione)
interface UserRepositoryInterface
{
    public function findById(int $id): ?UserDTO;
    public function findByEmail(string $email): ?UserDTO;
    public function create(array $data): UserDTO;
}

class EloquentUserRepository implements UserRepositoryInterface
{
    public function findById(int $id): ?UserDTO
    {
        $user = User::find($id);
        return $user ? UserDTO::fromModel($user) : null;
    }
}

// ✅ ACCETTABILE - Repository con modelli Eloquent (Laravel standard)
interface UserRepositoryInterface
{
    public function findById(int $id): ?User;
    public function findByEmail(string $email): ?User;
    public function create(array $data): User;
}

```

---

## 5. Model (Eloquent)

### Cosa fa
- **Rappresentazione delle entità** del database
- **Active Record pattern** - ogni istanza rappresenta una riga
- **Gestione delle relazioni** tra entità

### Regole d'oro
- ✅ **Un model per tabella** del database
- ✅ **Usa `$fillable` o `$guarded`** per la mass assignment protection
- ✅ **Definisci relazioni** come metodi
- ✅ **Usa accessors e mutators** per trasformare dati
- ✅ **Usa scopes** per query riutilizzabili
- ❌ **Non fare logica business complessa** nei model → **usa Service Layer**
- ❌ **Non accedere direttamente** ad altre tabelle non correlate → **usa Repository Pattern**

#### Eccezioni
- **Accessor/Mutator**: Può fare logica di trasformazione dati
- **Scopes**: Può fare query complesse per il model stesso
- **Relazioni**: Può accedere a tabelle correlate tramite relazioni
- **Eventi Model**: Può fare logica in `creating`, `updating`, `deleting`
- **Validation**: Può fare validazione specifica del model
- **Pivot Models**: Può gestire tabelle pivot con logica specifica

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

---

## 6. Controller

### Cosa fa
- **Gestisce le richieste HTTP** e restituisce risposte
- **Coordina** tra service, repository e view
- **Applica middleware** per autenticazione e autorizzazione

### Regole d'oro
- ✅ **Mantieni i controller magri** - logica business nei service
- ✅ **Usa Resource Controllers** per operazioni CRUD standard
- ✅ **Usa Form Request** per validazione
- ✅ **Restituisci Resource** per API consistenti
- ✅ **Un controller per risorsa** - gestisce una entità specifica
- ❌ **Non fare logica business** nei controller → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Repository Pattern**

#### Eccezioni
- **CRUD Controller**: Può gestire più azioni correlate (index, show, store, update, destroy)
- **Logica HTTP**: Può gestire redirect, response headers, status codes
- **File Upload**: Può gestire upload e validazione file
- **Pagination**: Può gestire paginazione e filtri semplici
- **Authentication**: Può gestire login/logout se non complesso
- **Simple Controllers**: Per operazioni molto semplici può accedere direttamente al model

### Esempi pratici
```php
// ✅ CORRETTO - Controller magro
class UserController extends Controller
{
    public function __construct(private UserService $userService) {}
    
    public function store(CreateUserRequest $request)
    {
        $user = $this->userService->createUser($request->validated());
        return new UserResource($user);
    }
}
```

---

## 7. Middleware

### Cosa fa
- **Filtra le richieste HTTP** prima che raggiungano il controller
- **Applica logica cross-cutting** (auth, CORS, logging)
- **Modifica request/response** in modo trasparente

### Regole d'oro
- ✅ **Una responsabilità per middleware** - Single Responsibility
- ✅ **Usa middleware groups** per raggruppare logica correlata
- ✅ **Applica middleware** a route specifiche quando possibile
- ✅ **Restituisci response** o chiama `$next()`
- ❌ **Non fare logica business** nel middleware → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Repository Pattern**

#### Eccezioni
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

### Regole d'oro
- ✅ **Una Form Request per endpoint** specifico
- ✅ **Usa regole di validazione** appropriate
- ✅ **Implementa `authorize()`** per controlli di accesso
- ✅ **Usa `messages()`** per messaggi personalizzati
- ❌ **Non fare logica business** nella Form Request → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Repository Pattern**

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

### Regole d'oro
- ✅ **Una Resource per entità** specifica
- ✅ **Usa `with()`** per includere relazioni
- ✅ **Usa `when()`** per dati condizionali
- ✅ **Usa `makeHidden()`** per nascondere campi sensibili
- ❌ **Non fare logica business** nelle Resource → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Repository Pattern**

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

### Regole d'oro
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

### Regole d'oro
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

### Regole d'oro
- ✅ **Una Policy per modello** o risorsa
- ✅ **Metodi specifici** per azioni (view, create, update, delete)
- ✅ **Usa `before()`** per controlli globali
- ✅ **Policy per logica complessa**, Gates per logica semplice
- ❌ **Non fare logica business** nelle Policy → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Repository Pattern**

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

### Regole d'oro
- ✅ **Layout principale** per struttura comune
- ✅ **Componenti** per elementi riutilizzabili
- ✅ **Sections** per contenuto specifico
- ✅ **Directives personalizzate** per logica complessa
- ❌ **Non fare logica business** nei template → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Repository Pattern**

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

### Regole d'oro
- ✅ **Un comando per task** specifico
- ✅ **Usa `handle()`** per la logica principale
- ✅ **Opzioni e argomenti** per configurazione
- ✅ **Output informativo** per l'utente
- ❌ **Non fare logica business** nei comandi → **usa Service Layer**
- ❌ **Non accedere direttamente** al database → **usa Repository Pattern**

### Esempi pratici
```php
// ✅ CORRETTO - Artisan Command
class CreateUserCommand extends Command
{
    protected $signature = 'user:create {name} {email}';
    protected $description = 'Create a new user';
    
    public function handle()
    {
        $user = User::create([
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

### Regole d'oro
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

### Regole d'oro
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

### Regole d'oro
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

### Regole d'oro
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

## 🎯 Il Service Layer: Il Cuore Logico di Laravel

### Perché il Service è il Cuore dell'Applicazione

Il **Service Layer** rappresenta il **cervello** della tua applicazione Laravel. È qui che vive tutta la logica business, le regole del dominio e l'intelligenza dell'applicazione.

### Architettura a Livelli con Service al Centro

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                       │
│  Controller │ Form Request │ Resource │ Blade │ Middleware  │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                   SERVICE LAYER (CUORE)                     │
│  UserService │ PaymentService │ OrderService │ EmailService │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  • Logica Business                                      ││
│  │  • Regole del Dominio                                   ││
│  │  • Orchestrazione                                       ││
│  │  • Coordinamento                                        ││
│  │  • Decisioni Importanti                                 ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                    DATA LAYER                               │
│  Repository │ Model │ Migration │ Seeder │ Factory          │
└─────────────────────────────────────────────────────────────┘
```

### Caratteristiche del Service Layer

#### ✅ **Cosa fa il Service:**
- **Centralizza la logica business** - Tutte le regole del dominio
- **Orchestra le operazioni** - Coordina repository, eventi, altri servizi
- **Mantiene la coerenza** - Applica regole uniformemente
- **Gestisce transazioni complesse** - Operazioni multi-step
- **Comunica tramite eventi** - Decoupling con altri servizi
- **È indipendente dal framework** - Testabile e riutilizzabile

#### ❌ **Cosa NON fa il Service:**
- **Non gestisce HTTP** → usa Controller
- **Non accede direttamente al DB** → usa Repository
- **Non formatta dati per API** → usa Resource
- **Non gestisce validazione input** → usa Form Request
- **Non gestisce autenticazione** → usa Middleware/Policy

### Esempio Pratico: Service come Cuore

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

// ✅ CORRETTO - Service come cuore logico
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

### Flusso Tipico con Service al Centro

```
1. Controller riceve richiesta HTTP
2. Form Request valida i dati
3. Controller chiama Service
4. Service applica logica business
5. Service coordina Repository per i dati
6. Service lancia eventi per comunicare
7. Listener gestiscono le conseguenze
8. Service restituisce risultato
9. Controller restituisce Resource
10. Response HTTP al client
```

### Vantaggi del Service come Cuore

#### 🧠 **Centralizzazione**
- **Una fonte di verità** per la logica business
- **Coerenza** tra Controller, Command, Job
- **Manutenibilità** - modifiche in un posto solo

#### 🔄 **Riusabilità**
- **Stesso Service** da Controller, Command, Job
- **API unificata** per operazioni business
- **Testabilità** - facile da testare in isolamento

#### 🎯 **Separazione delle Responsabilità**
- **Controller**: Gestisce HTTP
- **Service**: Logica business
- **Repository**: Accesso dati
- **Model**: Rappresentazione entità

#### 🚀 **Flessibilità**
- **Cambiamenti** senza impatto su altri layer
- **Evoluzione** del dominio senza rotture
- **Testing** indipendente per ogni layer

### Regole d'Oro per il Service Layer

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

Il **Service Layer** è il **cuore pulsante** della tua applicazione Laravel. È qui che vive l'intelligenza del dominio, dove si prendono le decisioni importanti e dove si coordina tutto il sistema.

**Ricorda**: Se hai logica business, va nel Service. Se non hai logica business, probabilmente non ti serve un Service.

---

## Regole d'Oro Generali

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
1. **Route** → definisce URL e metodo
2. **Middleware** → filtra la richiesta
3. **Controller** → riceve la richiesta
4. **Form Request** → valida i dati
5. **Service** → esegue logica business
6. **Repository** → accede ai dati
7. **Model** → rappresenta l'entità
8. **Resource** → trasforma per API
9. **Response** → restituisce al client

### Componenti per responsabilità
- **HTTP**: Controller, Middleware, Form Request
- **Business**: Service, Event, Listener
- **Data**: Repository, Model, Migration
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

*Questo cheat sheet fornisce una panoramica rapida delle componentistiche principali di Laravel e le regole d'oro per utilizzarle correttamente. Per implementazioni dettagliate e esempi completi, consulta i pattern specifici nella documentazione.*
