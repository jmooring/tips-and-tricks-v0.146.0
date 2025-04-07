+++
title = 'Home'
date = 2023-01-01T08:00:00-07:00
draft = false
+++

# Introducing the new template system in v0.146.0

Table of contents:

- [Introduction](#introduction) 
- [Breaking changes](#breaking-changes)
  - [Interpretation of dots in file names](#interpretation-of-dots-in-file-names)
  - [Redundant “partials” prefix in partial calls](#redundant-partials-prefix-in-partial-calls)
  - [Inline partial overrides removed](#inline-partial-overrides-removed)
- [Inline partials](#inline-partials)
- [Calling embedded partial templates](#calling-embedded-partial-templates)

## Introduction

Hugo v0.146.0 introduces a redesigned template system focused on making template lookups more consistent and easier to understand.

Key changes include:

- Template resolution now considers the full path of content files, not just the top-level section or content type.
- Base templates, page templates, content views, shortcodes, render hooks now follow the same template lookup logic. This allows for more consistent naming conventions across template types.
- The `shortcodes` and `partial` directories have been renamed to `_shortcodes` and `_partials`.
- The `_shortcodes` and `_markup` directories can now be placed at any level within the project's `layouts` directory.
- The concept of `_internal` templates has been removed. See [details](#calling-embedded-partial-templates).
- Templates named `index` are ignored. Use `home` instead.
- A new `all` template acts as a fallback when a specific template for a page kind is not found.

The new template system is generally backwards compatible, except for the documented breaking changes listed below. Please be aware that due to the vast range of existing template implementations, some unforeseen edge cases might not be fully compatible.

## Breaking changes

This release introduces changes that might require updates to your project.

### Interpretation of dots in file names

#### Change

To create more consistent path handling between content and templates, Hugo now treats everything after the first dot (`.`) in a filename as an identifier (like a language code or file extension), rather than part of the base name used for generating URLs.

#### Example

A file previously named `content/about-dot.net.md` will now render its output to `public/about-dot/index.html`. The `.net` part is interpreted as an identifier, not part of the page's slug.

#### Action required

If you have files using dots in their base names (like the example above), you should rename them using hyphens instead (e.g., `about-dot-net.md`). If you need to maintain the previous URL structure, you can explicitly set the desired path using the `slug` field in the file's front matter.

### Redundant "partials" prefix in partial calls

#### Change

Hugo is no longer lenient towards partial calls that incorrectly include the `partials/` directory prefix. While previous versions might have tolerated calls like `{{ partial "partials/foo.html" }}`, this was technically incorrect as Hugo looks within the partials directory automatically.

#### New behavior

Starting with v0.146.0, using the `partials/` prefix in a call like:

```text
{{ partial "partials/foo.html" }}
```

will generate a specific warning:

```text
WARN  Partial name "partials/foo.html" starting with 'partials/' (as in {{ partial "partials/foo.html"}}) is most likely not what you want. Before 0.146.0 we did a double lookup in this situation.
You can suppress this warning by adding the following to your site configuration:
ignoreLogs = ['warning-partial-superfluous-prefix']
```

Furthermore, the call will likely result in an error if the partial cannot be found without the prefix (as the redundant lookup logic has been removed):

```text
Error: ... error calling partial: partial "partials/foo" not found
```

Suppressing the warning above is only appropriate if you have intentionally placed your partial templates inside an unusual nested directory like `layouts/partials/partials/`. For standard Hugo projects, this warning indicates an error in your template code.

#### Action required

Review your templates and ensure all partial calls omit the `partials/` prefix. The correct way to call the partial `layouts/partials/foo.html` is simply:

```text
{{ partial "foo.html" }}
```

### Inline partial overrides removed

#### Change

The ability to override partial templates located in the `layouts/_partials` directory using inline partials has been removed. Previously, this undocumented behavior allowed inline partials (called without a file extension) to take precedence.

This change is expected to impact a small number of sites.

#### New behavior

Partial templates within the `layouts/_partials` directory will now consistently be used instead of any inline partials with the same name.

#### Required action

If you have inline partials that share names with files in your `layouts/_partials` directory, you will need to rename your inline partials to prevent naming conflicts.

## Directory structure

TODO

## Lookup order / weights / something

TODO

## Inline partials

As noted above, partial templates now reside in the `layouts/_partials` directory instead of `layouts/partials`. Use the same path when defining inline partials.

Change this:

```text
{{ partial "foo" }}

{{ define "partials/foo" }}
   FOO
{{ end }}
```

To this:

```text
{{ partial "foo.html" }}

{{ define "_partials/foo.html" }}
   FOO
{{ end }}
```

Remember that inline partials possess global scope; they can be called from any template, regardless of where they are defined. Exercise caution with naming to prevent namespace collisions.

## Calling embedded partial templates

Starting with v0.146.0, the way you call Hugo's embedded partial templates has been standardized. Instead of using the `template` function and the `_internal/` prefix, you now use the familiar `partial` function.

Old syntax:

```text
{{ template "_internal/disqus.html" . }}
{{ template "_internal/google_analytics.html" . }}
{{ template "_internal/opengraph.html" . }}
{{ template "_internal/pagination.html" . }}
{{ template "_internal/schema.html" . }}
{{ template "_internal/twitter_cards.html" . }}
```

New syntax:

```text
{{ partial "disqus.html" . }}
{{ partial "google_analytics.html" . }}
{{ partial "opengraph.html" . }}
{{ partial "pagination.html" . }}
{{ partial "schema.html" . }}
{{ partial "twitter_cards.html" . }}
```

This change applies to all embedded partials (Disqus, Google Analytics, OpenGraph, Pagination, Schema, Twitter Cards) and makes their usage consistent with your custom partials.

Note that the old syntax still works, but it will be deprecated in a future release.

## Other (these may need their own top level section)

Standard layout "all". See <https://github.com/gohugoio/hugo/issues/13545>.
