# Static Site Generator Design

**Status:** Draft

## Purpose

This project is a static-first publishing system for a personal website.
Authors write CommonMark-compatible source files. Goldfish Scheme parses,
transforms, and renders those files into a portable release. A Go server built
with `net/http` previews and delivers the release.

Publishing and HTTP delivery are separate concerns: the generator does not
handle visitor requests, and the server does not parse or evaluate source
markup.

## Architectural decisions

- Goldfish Scheme owns parsing, document transformations, rendering, build
  orchestration, and deployment plugins.
- Go's standard `net/http` package owns the HTTP server.
- Markup does not support executable inline code in the initial release.
- The generator is composed from explicit readers, transforms, builders,
  artifacts, and deployers, following Haunt's core design ideas.
- Each build produces static files plus a versioned manifest for the server.
- Production delivery is static-first. Bounded runtime rules, such as
  redirects and cache policies, are declarative manifest data, not Scheme
  evaluated per request.

## Goals and non-goals

### Goals

- Preserve CommonMark behavior for the supported core syntax.
- Make pages, feeds, tags, assets, themes, syntax highlighting, redirects, and
  deployment targets reusable modules.
- Produce deterministic releases and actionable diagnostics.
- Use the same release format for local preview and production delivery.
- Permit deployment without access to the original source tree.

### Non-goals for the first release

- Executing author-supplied code inside markup.
- A general-purpose application framework or database-backed CMS.
- Full Shiki-compatible grammar coverage.
- Reimplementing TLS, a CDN, or a reverse proxy.
- Runtime loading of untrusted third-party plugins.

## System overview

```text
content + assets + site configuration
                 │
                 ▼
          Goldfish generator
  readers → transforms → builders → artifacts
                 │
                 ▼
        release/
          ├── public/              HTML, CSS, JS, media, feeds
          └── site-manifest.json   routes and serving policy
                 │
        ┌────────┴────────┐
        ▼                 ▼
 Go development server   deployer plugin
 watch + rebuild + reload       │
        │                       ▼
        └──────────────► production Go server
                            optionally behind Caddy or nginx for TLS
```

`public/` is portable static output. `site-manifest.json` holds only behavior
that cannot be represented by a file tree alone.

## Generator design

### Project layout

Project paths are configurable. A conventional layout is:

```text
site/
  site.scm             trusted Goldfish configuration and plugin composition
  content/             markup documents
  static/              files copied unchanged
  themes/              reusable templates and styles
  build/               generated output; never hand-edited
```

`site.scm` is trusted build configuration, not document markup. Removing
inline code means a content file cannot execute Scheme simply by being parsed.

### Build pipeline

1. **Load configuration:** import explicitly listed Goldfish modules and build
   the site registry.
2. **Discover sources:** walk configured directories and select a reader for
   each eligible input.
3. **Read documents:** readers return metadata, a document AST, source path,
   and source locations for diagnostics.
4. **Transform:** transforms validate metadata, resolve links, build indexes,
   or decorate AST nodes such as fenced code blocks.
5. **Build artifacts:** builders create pages, feeds, tag archives, copied
   assets, redirect rules, and manifest contributions.
6. **Validate:** reject duplicate or unsafe output paths, invalid manifest
   rules, and unresolved links when strict checking is enabled.
7. **Emit:** write a staging release, validate it, then make it the active
   release only after the entire build succeeds.
8. **Deploy:** a named deployer can publish the completed release.

The first implementation may rebuild every source file. Artifacts should still
record dependencies and content hashes so incremental builds can be introduced
without changing plugin interfaces.

### Markup and rendering

The parser targets CommonMark core behavior and uses the CommonMark
specification tests as a compatibility suite. It produces an AST rather than
HTML strings, allowing later stages to inspect and transform documents safely.

Custom syntax must be explicit and feature-gated. Documents containing only
CommonMark should retain their normal meaning. Every extension needs documented
syntax, AST nodes, rendering behavior, diagnostics, and tests.

Raw HTML is represented explicitly in the AST. The renderer must expose a
documented policy: trusted personal-site content can render raw HTML, while a
safe profile can escape or omit it.

### Syntax highlighting

Highlighting is a reusable build module, not HTTP-server behavior.

```text
fenced code AST node
  → highlighter(language, source, theme)
  → token tree
  → HTML renderer + generated theme CSS
```

Start with the languages used by the site. Unknown languages render as escaped
plain text. The highlighter returns structured tokens or document fragments,
not untrusted prebuilt HTML. Cache results using source, language, grammar
version, and theme version.

## Plugin model

The project borrows Haunt's small, composable interfaces rather than creating a
large implicit hook system.

| Component | Input | Output | Examples |
| --- | --- | --- | --- |
| Reader | source path and bytes | `Document` | CommonMark reader, future import reader |
| Transform | document or site index | transformed document/index | link resolver, highlighter |
| Builder | build context and documents | artifacts | pages, blog index, RSS, assets |
| Artifact | path and content/writer | public file or manifest contribution | HTML, CSS, redirect |
| Deployer | completed release and target config | deployment result | rsync, object storage |

An artifact carries at least:

```text
path, writer or bytes, content type, cache policy, source dependencies, hash
```

### Composition rules

- Plugins are ordinary, explicitly imported Goldfish modules. Automatic plugin
  discovery is not required initially.
- A plugin declares a name, API version, provided capabilities, and ordering
  requirements.
- The registry resolves ordering before a build and rejects dependency cycles.
- Duplicate artifact paths are errors; plugins cannot silently overwrite each
  other.
- Builders receive an explicit build context rather than unrestricted global
  state.
- Parsing, transformation, and rendering logic should be testable without
  filesystem or network access.
- Prefer these narrow interfaces over unordered "run a hook at any time"
  events.

Themes are builders and renderer components, not a separate global subsystem.
They can supply layouts, document renderers, templates, and theme assets.

```scheme
;; Pseudocode; final Goldfish record syntax is intentionally not fixed.
(site
  #:readers    (list commonmark-reader)
  #:transforms (list validate-metadata resolve-links highlight-code)
  #:builders   (list page-builder blog-builder feed-builder static-builder)
  #:deployers  (list rsync-deployer object-storage-deployer))
```

This makes the site programmable through trusted configuration while keeping
content files non-executable.

## Release contract

Each successful build creates a self-contained release:

```text
build/release/
  public/
    index.html
    posts/example/index.html
    assets/app.css
    feed.xml
  site-manifest.json
```

The manifest has a schema version and is validated by both the generator and
the Go server. It contains declarative redirects, content variants, response
headers, and cache policies. It never contains executable Scheme or arbitrary
shell commands.

```json
{
  "version": 1,
  "redirects": [
    { "from": "/old-post/", "to": "/posts/new-post/", "status": 308 }
  ],
  "headers": [
    { "path": "/assets/", "cache_control": "public, max-age=31536000, immutable" }
  ]
}
```

The server rejects unsupported manifest versions rather than guessing at their
meaning. It loads a new manifest only after the new release validates.

## Go HTTP server

### Responsibilities

The Go server is built on `net/http`. Given an active release and its manifest,
it:

1. exposes development-only health and reload endpoints when enabled;
2. applies validated redirects and header rules;
3. maps a request to a file under `public/`, including configured index-file
   behavior; and
4. returns a correct static response or a 404.

It must not parse Goldfish source, invoke the generator for a visitor request,
or execute markup content.

### Development mode

Development mode watches source and configuration directories, debounces
changes, and invokes the generator build command. On success, it switches to
the new complete release and notifies browsers through Server-Sent Events or a
small live-reload endpoint. On failure, it keeps serving the last successful
release and reports diagnostics in the terminal and a development-only error
endpoint.

### Production mode

Production mode does not watch or build. It serves one selected release, logs
requests and errors, shuts down gracefully, and accepts configuration through
flags and environment variables.

Baseline behavior includes:

- `GET` and `HEAD`;
- safe URI decoding and containment checks to prevent path traversal;
- no directory listings by default;
- MIME types, explicit method handling, and sensible error responses;
- conditional requests and cache headers where artifact metadata permits;
- range support when media serving requires it; and
- request-size and timeout limits for non-static development endpoints.

TLS, certificates, compression, and CDN integration can be handled by Caddy,
nginx, or a managed edge in front of the Go process.

### Runtime extensibility

Build plugins and server plugins are different interfaces. Simple runtime
behavior belongs in the manifest. A future request-time feature should be an
explicit Go module with a documented request/response contract, authentication
policy, and tests.

Do not add a generic mechanism that starts Goldfish for each HTTP request. If
arbitrary Scheme request handlers become a requirement, reconsider the
single-runtime architecture as a separate design decision.

## Deployment

Deployment is a named deployer module. A deployer receives a completed release
and target configuration, then publishes it without rebuilding. Initial targets
can include local copy, rsync over SSH, and object storage.

For an installation using the Go delivery process, deploy the release directory
and the pinned server binary/configuration. Prefer versioned release directories
and an atomic active-release switch so a failed deployment can roll back.
Credentials remain in the deployment environment or secret store, never in the
release manifest.

## Quality gates

- Run CommonMark specification tests for the supported parser profile.
- Unit-test readers, transforms, builders, artifact writers, and deployers.
- Use golden tests for representative source-to-HTML rendering.
- Test plugin ordering, artifact collisions, unsafe paths, and manifest
  validation.
- Integration-test redirects, cache headers, `HEAD`, 404s, URI decoding,
  traversal attempts, and development reloads.
- Build the same source twice and verify deterministic output when variable
  metadata is disabled.

## Delivery plan

1. Define the document AST, CommonMark reader, HTML renderer, and artifact
   writer.
2. Add static assets, basic page builders, metadata, themes, and release
   validation.
3. Implement the minimal production Go `net/http` server.
4. Add watcher-driven rebuilds, transactional release switching, and live
   reload for development.
5. Add syntax highlighting for the site's required languages and themes.
6. Add redirects, cache policies, and deployer modules.
7. Add incremental build caching only after full builds are correct and tested.
