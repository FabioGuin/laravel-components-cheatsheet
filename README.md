# Laravel Components Cheat Sheet

> **Riferimento rapido** per le componentistiche principali di Laravel e le best practices per utilizzarle correttamente

<div align="center">
  <img src="assets/hero-image.png" alt="Laravel Components Cheat Sheet" width="800" />
</div>

[![Laravel](https://img.shields.io/badge/Laravel-9+-FF2D20?style=for-the-badge&logo=laravel&logoColor=white)](https://laravel.com)
[![PHP](https://img.shields.io/badge/PHP-8.1+-777BB4?style=for-the-badge&logo=php&logoColor=white)](https://php.net)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue?style=for-the-badge)](LICENSE)

## Panoramica

Questo cheat sheet fornisce una guida completa e pratica per le componentistiche principali di Laravel, con focus sulle **best practices** per utilizzarle correttamente.

## Obiettivo

- **Riferimento rapido** per sviluppatori Laravel
- **Best Practices** per ogni componentistica
- **Esempi pratici** di codice corretto e sbagliato
- **Pattern alternativi** per ogni anti-pattern
- **Service Layer** come core logico dell'applicazione

## Contenuto

### 📚 **Cheat Sheet Principali**

#### 1. [**Laravel Components Cheat Sheet**](LARAVEL-COMPONENTS-CHEAT-SHEET.md)
**Panoramica generale** delle 18 componentistiche principali di Laravel:
- Service Container, Service Provider, Service, Repository
- Model, Controller, Middleware, Form Request
- Resource, Event & Listener, Job, Policy
- Blade Templates, Artisan Commands, Migration, Seeder, Factory, Test

#### 2. [**Laravel $this->app Methods Cheat Sheet**](LARAVEL-APP-METHODS-CHEAT-SHEET.md)
**Approfondimento specifico** sui metodi di `$this->app`:
- 17 sezioni dettagliate sui metodi del Service Container
- Guida completa a binding, risoluzione, configurazione
- Metodi helper sostitutivi quando `$this->app` non è accessibile
- Best practices e esempi pratici

### 🔗 **Relazione tra i Cheat Sheet**

- **Components Cheat Sheet**: Panoramica generale e best practices
- **$this->app Methods Cheat Sheet**: Approfondimento tecnico specifico
- **Cross-referencing**: Link bidirezionali tra i documenti
- **Coerenza**: Principi allineati e complementari

## Come Usare

### 🚀 **Per Iniziare**
1. **Inizia** con il [**Laravel Components Cheat Sheet**](LARAVEL-COMPONENTS-CHEAT-SHEET.md) per la panoramica generale
2. **Approfondisci** con il [**Laravel $this->app Methods Cheat Sheet**](LARAVEL-APP-METHODS-CHEAT-SHEET.md) per i dettagli tecnici

### 📖 **Per Consultazione**
- **Service Container**: Components → $this->app Methods
- **Service Provider**: Components → $this->app Methods (sezione Service Provider)
- **Best Practices**: Entrambi i cheat sheet sono allineati
- **Helper Sostitutivi**: $this->app Methods → Metodi Helper Sostitutivi

## Struttura

```
Laravel-Components-CheatSheet/
├── README.md                           # Questo file
├── LARAVEL-COMPONENTS-CHEAT-SHEET.md  # Cheat sheet principale (panoramica)
├── LARAVEL-APP-METHODS-CHEAT-SHEET.md # Cheat sheet $this->app (approfondimento)
├── LICENSE                             # Licenza Apache 2.0
├── assets/                             # Risorse grafiche
│   └── hero-image.png                  # Immagine HERO
└── .gitignore                          # File da ignorare
```

## Caratteristiche

- ✅ **Best Practices** per ogni componentistica
- ✅ **Esempi pratici** di codice corretto e sbagliato
- ✅ **Pattern alternativi** per ogni anti-pattern
- ✅ **Approfondimento** sul Service Layer
- ✅ **Quick reference** per consultazione rapida
- ✅ **Formattazione** chiara e leggibile

## Link Utili

- [Laravel Documentation](https://laravel.com/docs/12.x)
- [Laravel Service Container](https://laravel.com/docs/12.x/container)
- [Laravel Service Providers](https://laravel.com/docs/12.x/providers)
- [Laravel Contributing Guide](https://laravel.com/docs/12.x/contributions)

## Contribuire

Le contribuzioni sono benvenute! Se vuoi migliorare questo cheat sheet:

1. **Fork** del repository
2. **Crea** un branch per la tua feature
3. **Modifica** il contenuto
4. **Crea** una Pull Request

## Licenza

Questo progetto è rilasciato sotto licenza Apache 2.0. Vedi il file [LICENSE](LICENSE) per i dettagli.

## Autore

Creato per la community Laravel italiana.

---

**Nota**: Questo cheat sheet è complementare al progetto [Common Design Patterns](https://github.com/FabioGuin/common-design-patterns) che si concentra sui pattern di design applicati a Laravel.
