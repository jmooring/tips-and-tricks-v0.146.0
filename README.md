# Introducing the new template system in v0.146.0

Table of contents:

- [Introduction](#introduction) 
- [Breaking changes](#breaking-changes)
  - [Interpretation of dots in file names](#interpretation-of-dots-in-file-names)
  - [Redundant “partials” prefix in partial calls](#redundant-partials-prefix-in-partial-calls)
- [Inline partials](#inline-partials)

## Introduction



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


## Directory structure



## Lookup order / weights / something



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

## Other (these may need their own top level section)

Standard layout "all". See <https://github.com/gohugoio/hugo/issues/13545>.

Deprecate "_internal". See <https://github.com/gohugoio/hugo/issues/13553>.

Templates in content. See <https://github.com/gohugoio/hugo/issues/13543>.
