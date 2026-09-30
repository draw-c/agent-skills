---
name: php-statamic-developer
description: Elite ninja Statamic developer — Antlers templating, collections, blueprints, Git-first flat-file workflows, and cache-driven performance.
---

# 🥷 Ninja PHP Statamic Developer Skill

> *"Content lives in Git. Structure lives in blueprints. Performance lives in the static cache."*

## Core Principles (The Ninja Code)

- **Everything in Git** — YAML config, blueprints, collections, and content. The repo *is* the source of truth; the database is optional and disposable.
- **Antlers only** — no Blade in templates. Antlers is the templating language; mixing runtimes invites confusion and breaks the flat-file model.
- **Blueprints + collections define structure** — never hand-author content fields in templates; declare them in YAML and let Statamic enforce the shape.
- **Forms are a security surface** — validation via Statamic's form rules, honeypot + reCAPTCHA/Turnstile, and CSRF on every submission.
- **Static cache is the default** — `static_caching` on for production; full or half mode depending on dynamic regions. If a page isn't cached, justify it.
- **Content is code-reviewed** — content changes go through PRs like any other change; no editing on prod.
- **Addons are dependencies** — pin versions, audit, and prefer first-party or well-maintained addons.
- **Multisite is declarative** — site definitions in YAML, translations in content files, no runtime site creation.

## Workflow (The Silent Path)

1. **Scaffold** — `composer create-project statamic/statamic`; configure `.env`, sites, and cache driver.
2. **Structure** — `php please make:collection`, `php please make:blueprint`, `php please make:fieldset`. Commit the YAML.
3. **Content model** — define fields, taxonomies, globals, and navigation in YAML *before* writing templates.
4. **Templates** — `php please make:antler`, build with partials, components, and view composers. No Blade.
5. **Forms** — `php please make:form`; add validation rules, honeypot, and Turnstile/reCAPTCHA.
6. **Cache** — enable static caching; configure invalidation on content save via Git webhook or deploy hook.
7. **Test** — Pest feature tests for routes and forms; Antlers rendering tests for critical templates.
8. **Ship** — Git push triggers deploy; run `php please stache:warm` and `php artisan cache:clear` post-deploy.

## Subagent Dispatch 🌀

| Subagent | Role | Triggered By |
|---|---|---|
| `collection-smith` | Collections, blueprints, fieldsets, taxonomies | New content type |
| `antler-artisan` | Templates, partials, components, view composers | New page / component |
| `form-sentinel` | Forms, validation rules, spam protection, CSRF | New form / submission |
| `git-warden` | Repo structure, PR workflow, content review | Content or config change |
| `cache-strategist` | Static cache, invalidation, warm-up, CDN | Perf / deploy |
| `addon-auditor` | Addon vetting, version pinning, upgrade paths | New addon / upgrade |
| `multisite-scribe` | Sites, locales, translations, domain routing | Multisite work |

## Patterns

**Blueprint (YAML) — declare structure, not templates:**
```yaml
# resources/blueprints/collections/pages/home.yaml
title: Home
sections:
  main:
    display: Main
    fields:
      - handle: hero_heading
        field:
          type: text
          required: true
          validate: 'max:80'
      - handle: hero_image
        field:
          type: assets
          container: assets
          max_files: 1
      - handle: body
        field:
          type: bard
          sets:
            text:
              fields:
                - { handle: text, field: { type: markdown } }
            callout:
              fields:
                - { handle: text, field: { type: text } }
                - { handle: tone, field: { type: select, options: { info: Info, warn: Warning } } }
```

**Antlers template — layout + partial + component:**
```antlers
{{# resources/views/layouts/base.antlers.html #}}
<!doctype html>
<html lang="{{ site:short_locale }}">
<head>
  {{ partial:meta }}
</head>
<body>
  {{ partial:nav }}
  <main>{{ template_content }}</main>
  {{ partial:footer }}
</body>
</html>
```

```antlers
{{# resources/views/pages/home.antlers.html #}}
{{ hero_heading }}
  <h1>{{ value }}</h1>
{{ /hero_heading }}

{{ if hero_image }}
  {{ hero_image }}
    <img src="{{ url }}" alt="{{ alt }}" loading="eager" width="{{ width }}" height="{{ height }}">
  {{ /hero_image }}
{{ /if }}

{{ body }}
  {{ if type == 'text' }}
    <div class="prose">{{ text | markdown }}</div>
  {{ elseif type == 'callout' }}
    <aside class="callout callout--{{ tone }}">{{ text }}</aside>
  {{ /if }}
{{ /body }}
```

**Reusable Antlers component with parameters:**
```antlers
{{# resources/views/components/button.antlers.html #}}
<a href="{{ url }}" class="btn btn--{{ variant ?? 'primary' }}">
  {{ slot }}
</a>
```

```antlers
{{ partial:components/button url="/contact" variant="secondary" }}
  Get in touch
{{ /partial:components/button }}
```

**Form with validation + spam protection:**
```antlers
{{ form:contact }}
  {{ if errors }}
    <ul class="errors">
      {{ errors }}<li>{{ value }}</li>{{ /errors }}
    </ul>
  {{ /if }}

  <input type="text" name="name" value="{{ old:name }}" required>
  <input type="email" name="email" value="{{ old:email }}" required>
  <textarea name="message" required>{{ old:message }}</textarea>

  {{# honeypot — hidden from humans, bots fill it in #}}
  <input type="text" name="{{ honeypot }}" class="hidden" tabindex="-1" autocomplete="off">

  {{# Turnstile (preferred) or reCAPTCHA #}}
  {{ turnstile }}

  <button type="submit">Send</button>
{{ /form:contact }}
```

**Static cache config (`config/statamic/static_caching.php`):**
```php
return [
    'strategy' => 'full',
    'driver' => 'file',
    'exclude' => [
        'contact',        // form-heavy pages
        'account/*',      // authenticated areas
    ],
    'warm' => true,
];
```

**Post-deploy warm + clear (in deploy script):**
```bash
php please stache:warm
php artisan cache:clear
php please static:warm --queue
```

## Review Checklist (The Strike Test)

- [ ] All content, blueprints, and config in Git — nothing hand-edited on prod
- [ ] Antlers only in templates — no Blade, no raw PHP in views
- [ ] Blueprints validate — `php please stache:doctor` clean
- [ ] Fields use appropriate `validate:` rules; required fields marked
- [ ] Forms: validation rules present, honeypot + Turnstile/reCAPTCHA enabled, CSRF active
- [ ] Form submissions tested (success, validation failure, spam rejection)
- [ ] Static cache enabled in production; exclusions documented
- [ ] Cache invalidation tested — content change reflects after deploy
- [ ] `php please stache:warm` runs post-deploy without errors
- [ ] Addons pinned in `composer.lock`; `composer audit` clean
- [ ] Addons vetted — maintained, compatible with current Statamic version
- [ ] Navigation, globals, and taxonomies declared in YAML, not templates
- [ ] Multisite (if used): sites in YAML, translations committed, routing tested
- [ ] Images use `alt` text; `width`/`height` set to prevent CLS
- [ ] SEO: meta partial present, sitemap generated, canonical URLs correct
- [ ] Redirects and 404s handled via Statamic's redirect/error config
- [ ] Pest feature tests cover routes and forms; Antlers rendering tests for critical templates
- [ ] `.env.example` updated; no secrets committed
- [ ] Deploy is Git-driven — no FTP, no manual file copies
- [ ] Lighthouse ≥ 90 on the top five pages (perf, a11y, best practices, SEO)