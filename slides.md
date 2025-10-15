---
theme: ../slidev-invo-theme
layout: cover
transition: slide-left | slide-right
drawings:
  persist: false
---

# Environment Cloning

---

# Agenda

## Review commands
- db:clone - The main command
- backup:run - [spatie/laravel-backup](https://spatie.be/docs/laravel-backup/)
- db:sanitize - Remove sensitive PII
- config:clone - Clone config files from remote server
- storage:clear - Remove cloned files

## Demo db:clone command

---

# db:clone

```zsh [php artisan db:clone]
Description:
  Clone a remote database via SSH using Spatie Laravel Backup, optionally sanitize PII data, and replace local database

Usage:
  db:clone [options] [--] <remote-user> <remote-host>

Arguments:
  remote-user                    SSH username for remote server
  remote-host                    Remote server hostname or IP

Options:
      --use-ssh-key              Use SSH key authentication instead of password
      --include-files            Include files in clone
      --include-config           Include config files and environment variables in clone
      --backup                   Create backup of local database before replacement
      --remote-dir[=REMOTE-DIR]   [default: "~/public_html/invo"]
      --no-sanitize              Skip database sanitization
      --dry-run                  Show what would be done without executing
```

<!--
    For the demo, run `sail artisan db:clone username 127.0.0.1 --include-files --include-config --backup --no-sanitize --dry-run --remote-dir="~/public_html/videobanking"`
-->
---

# db:clone

```php [config/database.php]{*}{maxHeight:'400px'}
'connections' => [
    'mysql' => [
        'dump' => [
            'useSingleTransaction' => true,
            'excludeTables' => [
                'call_event',
                'call_queue_answer',
                'call_users',
                'calls',
                'calls_events',
                'calls_statuses',
                'interactions',
                'interaction_comments',
                'interaction_events',
                'interaction_users',
                'invo_campaign_actions',
                'invo_campaign_interactions',
                'invo_campaign_recipients',
                'invo_campaign_tasks',
                'invo_campaign_triggers',
                'invo_campaigns',
                'password_resets',
                'password_history',
                'oauth_access_tokens',
                'oauth_refresh_tokens',
                'oauth_clients',
                'oauth_personal_access_clients',
                'oauth_auth_codes',
                'notifications',
                'push_subscriptions',
                'schedule_assignments',
                'schedule_event_exceptions',
                'schedule_events',
                'schedules',
                'websockets_statistics_entries',
            ]
        ]
    ]
]
```

<!--
`useSingleTransaction` will apply the `--single-transaction` flag to the `mysqldump` command, which will ensure that the dump is consistent and will not be affected by other transactions and the tables are not locked when creating backups on the remote server.
-->
---

# backup:run

Generates a .zip file of the application files and database in `storage/app/backups`.

```zsh [php artisan backup:run]
Description:
  Run the backup.

Usage:
  backup:run [options]

Options:
      --filename[=FILENAME]          
      --only-db                      
      --db-name[=DB-NAME]             (multiple values allowed)
      --only-files                   
      --only-to-disk[=ONLY-TO-DISK]  
      --disable-notifications        
      --timeout[=TIMEOUT]            
      --tries[=TRIES]                
      --isolated[=ISOLATED]          Do not run the command if another instance of the command is already running [default: false]
```

---

# backup:run

```php [config/backup.php] {3-5|6-9}
'source' => [
    'files' => [
        'include' => [
            storage_path('app')
        ],
        'exclude' => [
            storage_path('app/public/text'),
            storage_path('app/files'),
        ],
    ],
]
```

---

# backup:run

```php [config/backup.php]{*}{maxHeight:'400px'}
'source' => [
    'config_clone' => [
        'blacklisted_keys' => [],
        'whitelisted_env' => [
            'APP_KEY',
            'BROWSER',
            'DOCUMENT_SIGNING',
            'LOBBY_ENABLED',
            'LOBBY_REFRESH',
            'SIGNATURE_SOCKET',
            'WEB_CHANNEL',
            'MOBILE_CHANNEL',
            'IN_BRANCH_CHANNEL',
            'WEBSOCKET_ENABLE',
            'ENCRYPT_PERM_KEY',
            'ENCRYPT_A',
            'ENCRYPT_B',
            'KIOSK_LIMIT',
            'VIDYO_LINES',
            'VIDYO_CHANNEL'
        ]
    ],
]
```
---

# backup:run

```php [app/console/Kernal.php]{*}{lines:true,startLine:87}
// Clean up old backup files - run daily, spatie/laravel-backup package handles retention logic
$schedule->command('backup:clean')
         ->timezone(config('settings.locale_timezone'))
         ->daily()
         ->evenInMaintenanceMode();
```

### Other Considerations

- Storage space
- Memory usage
- Pre deploy backups

<!--
The db:clone script attempts to remove backups even if the user exits the command or if the command errors out. If 
anything is missed, this scheduled command will remove it.
-->
---

# db:sanitize

```zsh [php artisan db:sanitize]
Description:
  Sanitize database by anonymizing PII data in specified tables

Usage:
  db:sanitize [options]

Options:
      --table[=TABLE]            Specific tables to sanitize (default: all) (multiple values allowed)
      --parallel                 Run table sanitization in parallel
      --chunk-size[=CHUNK-SIZE]  Number of records to process per chunk [default: "1000"]
      --dry-run                  Show what would be sanitized without making changes
      --force                    Skip confirmation prompt (for automated usage)
```

---

# config:clone

```zsh [php artisan config:clone]
Description:
  Clone configuration files and environment variables from a remote server

Usage:
  config:clone [options] [--] <remote-user> <remote-host>

Arguments:
  remote-user                    SSH username for remote server
  remote-host                    Remote server hostname or IP

Options:
      --use-ssh-key              Use SSH key authentication instead of password
      --password[=PASSWORD]      SSH password (for non-interactive use)
      --remote-dir[=REMOTE-DIR]  Remote application directory [default: "~/public_html/invo"]
      --backup                   Create backups of existing config files before overwriting
      --env-only                 Clone only environment variables
      --config-only              Clone only config files
      --dry-run                  Show what would be done without executing
```

---

# storage:clear

```zsh [php artisan storage:clear]
Description:
  Clear all git-ignored files from storage directory

Usage:
  storage:clear [options]

Options:
      --dry-run         Show what would be deleted without actually deleting
      --force           Skip confirmation prompts
```

---
layout: center
---

# Demo

---
layout: center
---

# Questions?

---

# Discussions

- Refactoring
- AI code reviews