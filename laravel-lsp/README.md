# laravel-lsp

Laravel language server for Claude Code, providing framework-aware code intelligence for Laravel applications: completions, hover information, diagnostics, document links, and Go to Definition for Laravel and Blade code.

## Supported Extensions

`.php`

## Installation

Install Laravel LSP globally with Composer:

```bash
composer global require laravel/lsp
```

Ensure Composer's global bin directory is on your `PATH` (typically `~/.composer/vendor/bin` or `~/.config/composer/vendor/bin`). If the `laravel-lsp` command is not found after installation, add that directory to your `PATH`.

## Usage

The server communicates over stdio and is launched automatically as `laravel-lsp` for PHP files. It works best when started from the root of a Laravel application (where the `artisan` file lives).

## More Information

- [Laravel LSP on GitHub](https://github.com/laravel/lsp)
- [Laravel LSP on Packagist](https://packagist.org/packages/laravel/lsp)
