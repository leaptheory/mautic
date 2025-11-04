# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mautic is the world's largest open-source marketing automation platform built on Symfony 6.4, providing a complete alternative to proprietary tools like HubSpot and Marketo. It's privacy-focused, self-hosted, and fully customizable with a modular plugin/theme architecture.

**Technology Stack:**
- **Backend:** PHP 8.3, Symfony 6.4, Doctrine ORM 2.x
- **Frontend:** jQuery 3.7, Bootstrap 3.4.1, CKEditor 5, Chart.js 2.9.4
- **Database:** MySQL 8.4 (required for SKIP LOCKED support)
- **Message Queue:** RabbitMQ via symfony/amqp-messenger
- **Development Environment:** DDEV (Docker-based)

## Development Environment

### DDEV Setup (Recommended)

```bash
# Start environment (auto-installs on first run)
ddev start

# Default credentials: admin / Maut1cR0cks!
# Main URL: https://mautic.ddev.site
# PHPMyAdmin: https://mautic.ddev.site:8037
# Mailpit: https://mautic.ddev.site:8026

# Execute commands inside container
ddev exec <command>

# SSH into container
ddev ssh

# View logs
ddev logs

# Stop environment
ddev stop
```

**DDEV Configuration:**
- PHP 8.3 with apache-fpm webserver
- MySQL 8.4 database
- Timezone set to UTC (matches staging/production)
- Additional services: RabbitMQ, Redis, Selenium, Mailpit, PHPMyAdmin
- Auto-setup script runs on `ddev start`: `.ddev/mautic-setup.sh`

### Manual Setup (Without DDEV)

```bash
composer install
npm ci
npm run build
php bin/console mautic:install https://localhost
php bin/console mautic:plugins:reload
```

## Common Development Commands

### Testing

```bash
# Run all PHPUnit tests
composer test

# Run end-to-end tests (Codeception)
composer e2e-test

# Run specific test file
bin/phpunit --filter TestClassName app/bundles/BundleName/Tests/

# Static analysis
composer phpstan
```

### Code Quality

```bash
# Check code style (dry run)
composer cs

# Fix code style automatically
composer fixcs

# Code modernization with Rector
composer rector
```

### Asset Building

```bash
# Install Node dependencies
npm ci

# Build CKEditor5 webpack bundle
npm run build

# Apply npm patches
npx patch-package

# Generate all assets (runs after composer install/update)
composer generate-assets

# Compile LESS to CSS (uses Grunt)
grunt

# Watch LESS files for changes
grunt watch
```

### Symfony Console Commands

```bash
# Clear cache
php bin/console cache:clear
php bin/console cache:warmup

# Database migrations
php bin/console doctrine:migrations:migrate
php bin/console doctrine:migrations:generate

# Plugin management
php bin/console mautic:plugins:reload
php bin/console mautic:plugins:install

# Lead/Contact management
php bin/console mautic:segments:update
php bin/console mautic:contacts:deduplicate

# Campaign automation
php bin/console mautic:campaigns:trigger
php bin/console mautic:campaigns:rebuild

# Email processing
php bin/console mautic:emails:send
php bin/console mautic:email:process

# List all available commands
php bin/console list mautic
```

### Git Hooks

```bash
# Install pre-commit and post-checkout hooks
composer githooks
```

## Codebase Architecture

### Bundle-Based Modular Structure

Mautic uses a **30+ bundle architecture** where each bundle is a self-contained Symfony feature module:

```
app/bundles/
├── ApiBundle/              # REST API with OAuth2
├── CampaignBundle/         # Marketing automation & workflows
├── CategoryBundle/         # Content categorization
├── ChannelBundle/          # Multi-channel message delivery
├── ConfigBundle/           # System configuration UI
├── CoreBundle/             # Core utilities, helpers, base classes
├── DashboardBundle/        # Dashboard widgets & analytics
├── DynamicContentBundle/   # Content personalization
├── EmailBundle/            # Email marketing & templates
├── FormBundle/             # Form builder & submissions
├── LeadBundle/             # Contact/Lead CRM (main data model)
├── IntegrationsBundle/     # Third-party integrations framework
├── NotificationBundle/     # In-app notifications
├── PageBundle/             # Landing page builder
├── PluginBundle/           # Plugin system framework
├── PointBundle/            # Lead scoring system
├── ReportBundle/           # Custom reports & analytics
├── SmsBundle/              # SMS marketing
├── StageBundle/            # Lead pipeline stages
├── UserBundle/             # User & role management
└── WebhookBundle/          # Webhook delivery system
```

### Standard Bundle Structure

Each bundle follows Symfony conventions:

```
BundleName/
├── Controller/           # Web request handlers (extends CommonController)
├── Command/              # CLI commands (extend ContainerAwareCommand)
├── Entity/               # Doctrine ORM entities
├── Repository/           # Data access layer (query builders)
├── Form/                 # Form types & validators
├── Model/                # Business logic (services)
├── Helper/               # Utility classes
├── EventListener/        # Symfony event subscribers
├── Views/                # Twig templates
├── Assets/               # Bundle-specific CSS/JS
│   ├── css/
│   ├── js/
│   └── builder/         # Page/email builder assets
├── Translations/         # i18n translation files
├── Config/               # Bundle configuration
├── Tests/                # Unit & functional tests
│   ├── Unit/
│   └── Functional/
└── BundleNameBundle.php  # Bundle registration class
```

### Key Architectural Patterns

**1. Model Classes (Business Logic)**
- Located in `Model/` directory within each bundle
- Handle business logic, not controllers
- Example: `EmailBundle/Model/EmailModel.php` handles email operations

**2. Repositories (Data Access)**
- All database queries go through repositories
- Located in `Entity/Repository/`
- Extend Doctrine's `EntityRepository` or `CommonRepository`
- Use QueryBuilder for complex queries

**3. Form Types**
- Forms defined as classes extending `AbstractType`
- Located in `Form/Type/` directory
- Form validation via Symfony validators

**4. Event System**
- Heavy use of Symfony EventDispatcher
- Custom events defined in bundle `Events/` directory
- Event subscribers in `EventListener/` directory
- Key events: contact updates, campaign triggers, email sends, form submissions

**5. Factory Pattern**
- Model factories in `Model/Factory/` directories
- Example: `LeadBundle/Model/Factory/LeadFactory.php`

### Core Bundle Details

**LeadBundle** (The CRM Core)
- Central entity: `Lead` (aka Contact)
- Tracks contact fields, segments, activity, points
- Most other bundles interact with LeadBundle
- Timeline tracking for all contact activities

**CampaignBundle** (Marketing Automation)
- Workflow builder using jsPlumb for visual drag-and-drop
- Event-driven automation triggers
- Decision logic and conditional branching
- Scheduled actions and delays

**EmailBundle** (Email Marketing)
- Email template builder (CKEditor 5, GrapeJS)
- A/B testing support
- Email tracking (opens, clicks)
- Transactional and marketing emails
- Multiple transport options (SMTP, API integrations)

**FormBundle** (Form Builder)
- Drag-and-drop form builder
- Progressive profiling
- Form actions (contact updates, email sends)
- Conditional field display

**CoreBundle** (Foundation)
- Base controllers: `CommonController`, `FormController`, `AjaxController`
- Helper services: `PathsHelper`, `TemplatingHelper`, `UserHelper`
- Core entities: `IpAddress`, `AuditLog`
- Translation management

## Plugin System

### Plugin Architecture

Plugins extend Mautic's functionality and are located in `plugins/` directory:

```
plugins/
├── GrapesJsBuilderBundle/     # Visual page/email builder
├── MauticCrmBundle/           # CRM integrations (Salesforce, etc.)
├── MauticEmailMarketingBundle/ # Email service integrations
├── MauticSocialBundle/        # Social media integrations
├── MauticFocusBundle/         # Pop-ups, bars, form embeds
├── MauticClearbitBundle/      # Data enrichment
├── MauticCloudStorageBundle/  # S3, Dropbox integration
└── LeapTheoryBundle/          # Custom plugin (symlinked)
```

### Plugin Structure

Plugins are Symfony bundles with additional Mautic integration:

```
PluginName/
├── Config/
│   ├── config.php           # Plugin metadata & configuration
│   └── services.php         # Service definitions
├── Integration/             # Integration classes
│   └── NameIntegration.php  # Main integration logic
├── Form/                    # Configuration forms
├── Assets/                  # Plugin assets
└── PluginNameBundle.php     # Bundle class
```

### Working with Plugins

```bash
# Reload plugin list (after adding new plugin)
php bin/console mautic:plugins:reload

# Install all discovered plugins
php bin/console mautic:plugins:install

# Clear cache after plugin changes
php bin/console cache:clear
```

## Theme System

Themes control the appearance of emails and landing pages:

```
themes/
├── aurora/
├── oxygen/
├── fresh-left/
├── blank/          # Base theme for customization
└── ...             # 42+ built-in themes
```

Each theme has:
- `config.json` - Theme metadata
- `html/` - Template files
- `css/` - Theme styles
- `thumbnail.png` - Preview image

## Database & Doctrine

### Migrations

```bash
# Create new migration
php bin/console doctrine:migrations:generate

# Run pending migrations
php bin/console doctrine:migrations:migrate

# Check migration status
php bin/console doctrine:migrations:status
```

### Important Notes

- All tables use configurable prefix (default: empty, test environment uses `test_`)
- MySQL 8.4 required for `SKIP LOCKED` in queue processing
- Test database: `mautictest` (separate from `mautic` dev database)
- Doctrine lifecycle callbacks heavily used (PrePersist, PostLoad, etc.)

### Entity Conventions

```php
// Entities in Entity/ directory
namespace Mautic\BundleName\Entity;

use Doctrine\ORM\Mapping as ORM;
use Mautic\CoreBundle\Entity\FormEntity;

#[ORM\Entity(repositoryClass: "Mautic\BundleName\Entity\Repository\EntityNameRepository")]
#[ORM\Table(name: "table_name")]
class EntityName extends FormEntity
{
    // Properties with ORM attributes
    // Getters and setters
}
```

## Frontend Architecture

### JavaScript Organization

```
media/
├── js/                   # Core JavaScript files
├── libraries/            # Third-party JS libraries
│   ├── jquery/
│   ├── bootstrap/
│   └── ckeditor/
└── bundles/              # Bundle-specific compiled assets
    └── bundlename/
        ├── css/
        └── js/
```

### Key Frontend Libraries

- **jQuery 3.7.0** - DOM manipulation (legacy, being reduced)
- **Bootstrap 3.4.1** - UI framework
- **CKEditor 5** - Rich text editor (webpack compiled)
- **Chart.js 2.9.4** - Analytics charts
- **jsPlumb 2.15.6** - Campaign builder drag-and-drop
- **chosen.js** - Select dropdowns
- **Dropzone 4.3.0** - File uploads
- **Moment.js 2.29.4** - Date/time utilities

### Asset Pipeline

1. **LESS Compilation** (via Grunt)
   - Source: `app/bundles/*/Assets/css/*.less`
   - Compiled to: `media/css/`
   - Run: `grunt` or `grunt watch`

2. **CKEditor 5 Bundling** (via Webpack)
   - Config: `webpack.config.js`
   - Build: `npm run build`
   - Output: Custom CKEditor build with Mautic plugins

3. **Asset Generation**
   - Command: `php bin/console mautic:assets:generate`
   - Copies bundle assets to `media/` for web access

## Testing Strategy

### Test Structure

```
tests/
├── acceptance/           # Codeception E2E tests
│   └── *Cest.php
├── functional/          # Functional tests (not used much)
└── unit/                # Unit tests (not used much)

app/bundles/*/Tests/     # Bundle-specific tests (primary location)
├── Unit/
├── Functional/
└── Fixtures/            # Test data fixtures
```

### Running Tests

```bash
# All PHPUnit tests (2GB memory)
composer test

# Specific test class
bin/phpunit --filter EmailModelTest

# With coverage
bin/phpunit --coverage-html var/coverage

# Codeception E2E tests (requires Selenium)
composer e2e-test

# Specific acceptance test
bin/codecept run acceptance LoginCest
```

### Test Database

- Test environment uses separate `mautictest` database
- Tables prefixed with `test_` in test config
- Fixtures loaded via `liip/test-fixtures-bundle`

### Writing Tests

```php
// Functional test example
namespace Mautic\BundleName\Tests\Functional;

use Mautic\CoreBundle\Test\MauticMysqlTestCase;

class SomeTest extends MauticMysqlTestCase
{
    public function testSomething(): void
    {
        $entity = new EntityName();
        // Test logic
        $this->assertEquals($expected, $actual);
    }
}
```

## RabbitMQ Message Queue

### Configuration

- Service: RabbitMQ (AMQP 0-9-1)
- Symfony integration: `symfony/amqp-messenger`
- DSN configured in `config/parameters.yml`
- Used for: async email processing, campaign triggers, webhook delivery

### Working with Queues

```bash
# Consume messages from queue
php bin/console messenger:consume async -vv

# Check queue status
php bin/console messenger:stats

# Failed message handling
php bin/console messenger:failed:show
php bin/console messenger:failed:retry
```

### Key Notes

- Local DDEV uses non-SSL RabbitMQ (SSL disabled in DDEV config)
- Production uses SSL certificates for AMQP connections
- Event logging disabled to reduce noise (see recent commits)

## Important Configuration Files

```
config/
└── parameters.yml         # Symfony parameters (database, queue DSN, etc.)

app/config/
├── config.yml             # Main Symfony configuration
├── security.yml           # Security & authentication
├── routing.yml            # Route definitions
└── parameters.yml         # Application parameters (symlinked)

.ddev/
├── config.yaml            # DDEV environment config
├── mautic-setup.sh        # Auto-setup script
└── supervisor/            # Supervisor task configs (custom)

phpunit.xml.dist           # PHPUnit configuration (in app/)
codeception.yml            # Codeception configuration
webpack.config.js          # CKEditor 5 webpack config
Gruntfile.js              # LESS compilation config
```

## Development Workflow Best Practices

### Branch Strategy

- **Main branch:** `7.x` (not `main` or `master`)
- Feature branches: `feat/feature-name`
- Bugfix branches: `fix/bug-description`
- Create PRs against `7.x`

### Code Quality Checks

Before committing:

```bash
# Fix code style
composer fixcs

# Run static analysis
composer phpstan

# Run tests
composer test
```

### Making Changes to Bundles

1. **Identify the correct bundle** - Features are organized by business domain
2. **Start with the Model** - Business logic goes in `Model/` classes
3. **Add Repository methods** - Data access in `Entity/Repository/`
4. **Create/update Controller** - Keep controllers thin, delegate to Models
5. **Update Forms** - Form types in `Form/Type/`
6. **Add tests** - In bundle's `Tests/` directory
7. **Update translations** - In bundle's `Translations/` if adding UI strings

### Common Pitfalls to Avoid

1. **Don't put business logic in Controllers** - Use Model classes
2. **Don't use direct queries** - Use repositories with QueryBuilder
3. **Don't forget cache clearing** - After config changes: `php bin/console cache:clear`
4. **Don't skip migrations** - Database schema changes require migrations
5. **Don't commit vendor/** - Managed by Composer
6. **Don't commit media/css/** - Generated files, not source
7. **Don't bypass the event system** - Use EventDispatcher for extensibility

## Performance Considerations

### Caching

- Symfony cache in `var/cache/`
- Doctrine query cache and result cache available
- Redis available in DDEV for caching
- Clear cache after: config changes, plugin installs, schema updates

### Segments and Campaigns

```bash
# Rebuild contact segments (can be slow)
php bin/console mautic:segments:update

# Rebuild campaign membership
php bin/console mautic:campaigns:rebuild

# These should be run via cron in production
```

### Large Installations

- Use RabbitMQ for async processing
- Enable Redis caching
- MySQL 8.4 SKIP LOCKED prevents lock contention
- Consider segment filters carefully (can impact performance)

## Debugging

### Enable Debug Mode

```bash
# In .env or config/parameters.yml
APP_ENV=dev
APP_DEBUG=1
```

### Useful Debug Commands

```bash
# Symfony profiler (when debug enabled)
# Visit: https://mautic.ddev.site/_profiler

# Dump container services
php bin/console debug:container

# Debug routes
php bin/console debug:router

# Debug events
php bin/console debug:event-dispatcher

# Clear all caches
php bin/console cache:clear --no-warmup
rm -rf var/cache/*
```

### Logging

- Logs location: `var/logs/`
- Main log: `var/logs/mautic_dev.log` (or `mautic_prod.log`)
- Separate logs per bundle in some cases
- Error logs: `var/logs/error.log`

## Security Notes

- OAuth2 for API authentication (via `ApiBundle`)
- Symfony Security component for authentication/authorization
- Role-based access control (RBAC)
- GDPR compliance features built-in
- Security issues: Report via SECURITY.md process
- Never commit credentials (check `.gitignore`)

## Contributing

- Guidelines: `.github/CONTRIBUTING.md`
- Community handbook: https://contribute.mautic.org
- PR template: `.github/PULL_REQUEST_TEMPLATE.md`
- Code of Conduct: `.github/CODE_OF_CONDUCT.md`

## Documentation Resources

- **Developer Docs:** https://devdocs.mautic.org
- **User Docs:** https://docs.mautic.org
- **API Reference:** In DevDocs
- **Forum:** https://forum.mautic.org
- **Slack:** https://mautic.org/slack
- **GitHub Issues:** https://github.com/mautic/mautic/issues

## Upgrade Guides

Major version migrations documented in:
- `UPGRADE-3.0.md`
- `UPGRADE-4.0.md`
- `UPGRADE-5.0.md` (extensive changes)
- `UPGRADE-6.0.md` (current version)
- `UPGRADE-PHP-TO-TWIG-TEMPLATES.md`

## Quick Reference: File Locations

| What | Where |
|------|-------|
| Bundle code | `app/bundles/BundleName/` |
| Plugins | `plugins/PluginName/` |
| Themes | `themes/theme-name/` |
| Tests | `app/bundles/*/Tests/` and `tests/` |
| Assets (source) | `app/bundles/*/Assets/` |
| Assets (compiled) | `media/` |
| Translations | `translations/` and `app/bundles/*/Translations/` |
| CLI commands | `bin/console` |
| Configuration | `app/config/` and `config/` |
| Logs | `var/logs/` |
| Cache | `var/cache/` |
| Migrations | `app/migrations/` |
| DDEV config | `.ddev/` |
