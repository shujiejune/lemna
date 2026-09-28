# Static Site Generator TODO

This is the implementation roadmap for the Goldfish Scheme generator and Go
`net/http` server described in [design.md](design.md). Tasks are ordered by
dependency, not necessarily by estimated effort.

## How to use this list

- `[x]` means the repository already contains the completed decision or
  documentation.
- `[ ]` means work remains.
- Complete a milestone's exit criteria before treating its dependent milestones
  as stable.
- Keep executable behavior in trusted Goldfish configuration and modules; never
  add executable content syntax as an incidental shortcut.
- Update [requirements.md](requirements.md) and [design.md](design.md) whenever
  an unchecked design decision is resolved differently from the current plan.

## Completed groundwork

- [x] Compare Caisse, Haunt, Pollen, and Spindle and record the findings in
  [`analysis/comparison.md`](../analysis/comparison.md).
- [x] Choose Goldfish Scheme for parsing, rendering, build orchestration, and
  deployment modules.
- [x] Choose Go with `net/http` for the built-in HTTP server.
- [x] Separate the generator's release output from HTTP request handling.
- [x] Exclude executable inline markup from the initial scope.
- [x] Write the high-level architecture in [design.md](design.md).
- [x] Update [requirements.md](requirements.md) with the chosen constraints.

---

## Milestone 0 — Resolve implementation contracts

### Scope and version decisions

- [ ] Choose and pin the supported Goldfish Scheme release.
- [ ] Choose and pin the supported Go release.
- [ ] Define supported host operating systems for development and deployment.
- [ ] Choose the initial license for original generator and server code.
- [ ] Name the generator and server commands.
- [ ] Define a semantic-versioning policy for the generator, server, plugin API,
  and release-manifest schema.
- [ ] Record compatibility guarantees for site configurations and markup files.
- [ ] Define the initial v1 boundary: what is required before the project can
  claim CommonMark-compatible static-site generation.
- [ ] Decide whether the first public release is intended only for the personal
  site or also as a reusable tool for other sites.

### Source, metadata, and template decisions

- [ ] Choose the canonical extension for the custom/CommonMark-compatible
  content format.
- [ ] Choose one metadata/front-matter syntax and document exactly where it is
  valid.
- [ ] Define the initial metadata schema: at minimum title, route/slug, date,
  draft state, tags, template, description, and canonical URL policy.
- [ ] Decide which metadata fields are required for pages versus posts.
- [ ] Specify date and time-zone formats accepted by metadata validation.
- [ ] Decide the default raw-HTML policy and the opt-in safe policy.
- [ ] Define the initial theme and layout interface without allowing templates to
  execute content-provided code.
- [ ] Choose whether templates are trusted Scheme modules, a restricted template
  language, or renderer functions supplied by a theme module.
- [ ] Define the first custom markup extensions, if any; defer extensions until
  their CommonMark interaction is specified and tested.
- [ ] Explicitly reserve inline-code syntax so ordinary documents cannot
  accidentally gain executable semantics later.

### Release and serving decisions

- [ ] Freeze the initial release layout: staging location, release directory,
  `public/` root, manifest name, and active-release selection mechanism.
- [ ] Write the normative v1 `site-manifest.json` schema, including path-matching
  and precedence rules.
- [ ] Decide which redirect statuses v1 accepts and whether redirect sources are
  exact paths, prefixes, or both.
- [ ] Define header-rule matching, precedence, and the allowed response headers.
- [ ] Define URL-to-file rules for `/`, trailing slashes, `index.html`, and file
  extensions.
- [ ] Decide the first cache-policy vocabulary and its defaults for HTML,
  fingerprinted assets, and unversioned assets.
- [ ] Decide whether the first release needs custom 404/other error pages.
- [ ] Choose the development live-reload mechanism: Server-Sent Events plus a
  reload client, or a documented manual-refresh equivalent.
- [ ] Choose the first deployment target to implement after local deployment.

### Exit criteria

- [ ] Record every resolved decision above in `design.md` or a versioned schema
  document.
- [ ] Create a short v1 acceptance checklist that can be used before release.

---

## Milestone 1 — Repository, tooling, and test foundation

### Repository layout

- [ ] Choose and create the source directories for Goldfish modules, Go server
  code, shared fixtures, example sites, integration tests, and release tooling.
- [ ] Add a top-level README that states the project purpose, supported runtimes,
  and current development status.
- [ ] Add a `.gitignore` for generated releases, Goldfish/Go build products,
  editor files, local deployment state, and secrets.
- [ ] Keep the researched upstream repositories separate from new project source
  directories.
- [ ] Create a minimal example site that will serve as the end-to-end fixture.
- [ ] Add a fixture site that intentionally fails validation for diagnostic tests.
- [ ] Document the conventional site layout: configuration, content, static
  assets, themes, generated output, and local state.

### Goldfish development setup

- [ ] Verify Goldfish module imports, record/structure facilities, filesystem
  APIs, process APIs, JSON support, hashing support, and test-runner options.
- [ ] Add a reproducible way to invoke the generator in development.
- [ ] Create a minimal generator entry point that reports its version and exits
  successfully.
- [ ] Establish Goldfish formatting and linting conventions, if supported.
- [ ] Establish a test command that returns a nonzero status on failures.
- [ ] Add a small assertion, fixture-loading, temporary-directory, and
  golden-file test helper module.
- [ ] Verify Unicode, UTF-8, newline, filesystem-path, and locale behavior on
  supported platforms.

### Go development setup

- [ ] Create the Go module and pin its Go language version.
- [ ] Create a minimal server command that reports its version and exits
  successfully.
- [ ] Establish package boundaries for manifest loading, route resolution,
  static serving, development mode, and command-line configuration.
- [ ] Add `go test`, `go vet`, and race-detector commands to the development
  workflow.
- [ ] Decide whether a file-watching dependency is acceptable or whether the
  first watcher will use platform-independent polling.
- [ ] Add a Go test helper for temporary release directories and HTTP requests.

### Automation

- [ ] Add one command that runs Goldfish unit tests, Go unit tests, and
  integration tests.
- [ ] Add a formatter-check command for all supported source languages.
- [ ] Add a documentation-link check for files under `docs/`.
- [ ] Add continuous integration for supported Go and Goldfish versions.
- [ ] Make CI fail when generated fixtures unexpectedly differ from committed
  golden output.

### Exit criteria

- [ ] A clean checkout can run the empty generator, empty server, and all test
  commands using documented setup steps.

---

## Milestone 2 — Generator core data model and command interface

### Command interface

- [ ] Define the generator command's subcommands, including at least build,
  validation/check, and deployment.
- [ ] Define common flags for site root, configuration path, output/release
  path, verbosity, strict mode, and reproducible-build mode.
- [ ] Implement `--help` and clear invalid-argument diagnostics.
- [ ] Implement a structured diagnostic format with severity, source path, line,
  column, error code, and human-readable message where applicable.
- [ ] Make the command return nonzero on configuration, parsing, validation,
  emission, or deployment errors.
- [ ] Add a machine-readable diagnostics mode only if the editor/CI use case is
  defined.

### Core records

- [ ] Define the `Site` configuration record.
- [ ] Define the `Document` record: source identifier, normalized source text,
  metadata, AST, dependencies, and source locations.
- [ ] Define the document-AST node representation and source-span representation.
- [ ] Define the `Artifact` record: relative output path, content/writer,
  content type, cache policy, dependencies, and content hash.
- [ ] Define the `ManifestContribution` record.
- [ ] Define the immutable-or-conventionally-read-only `BuildContext` record.
- [ ] Define the `Release` record returned after successful emission.
- [ ] Define a normalized error/result representation for plugin boundaries.
- [ ] Document ownership and mutation rules for documents, ASTs, contexts, and
  artifacts.

### Paths, files, and determinism

- [ ] Implement normalized relative-path validation for source and output paths.
- [ ] Reject absolute paths, empty paths, `.`/`..` traversal, platform-specific
  unsafe names, and output paths outside the release root.
- [ ] Define symlink behavior for source discovery and output emission.
- [ ] Define text-file decoding and reject or diagnose invalid UTF-8 according to
  the markup specification.
- [ ] Normalize input line endings while retaining enough location information
  for useful diagnostics.
- [ ] Implement stable byte hashing for source files and artifacts.
- [ ] Support `SOURCE_DATE_EPOCH` or an equivalent explicit build timestamp.
- [ ] Ensure unordered collections are sorted before they affect output.
- [ ] Add unit tests for every path, hashing, timestamp, and ordering rule.

### Configuration loading

- [ ] Define the trusted `site.scm` loading contract and its expected exported
  site value.
- [ ] Implement configuration loading with errors that name the configuration
  file and failing location.
- [ ] Allow explicitly imported Goldfish modules only; do not implement implicit
  plugin discovery in v1.
- [ ] Validate configuration before source discovery starts.
- [ ] Add a minimal valid `site.scm` to the example site.
- [ ] Add tests for missing, malformed, and incompatible configurations.

### Exit criteria

- [ ] The generator can load a minimal site configuration, construct an empty
  build context, and report deterministic diagnostics for invalid input.

---

## Milestone 3 — CommonMark-compatible document model

### Parser profile and fixtures

- [ ] Pin the target CommonMark specification version and store its license and
  test-fixture provenance.
- [ ] Identify the exact CommonMark examples included in the supported profile.
- [ ] Create a test runner that maps specification examples to parser and HTML
  renderer assertions.
- [ ] Classify each specification failure as unsupported, parser defect,
  renderer defect, or intentionally disabled extension.
- [ ] Add regression fixtures for every fixed parser defect.
- [ ] Add tests proving that markup text is data, not executable Goldfish code.

### Source and AST infrastructure

- [ ] Represent a document as a sequence of block nodes.
- [ ] Represent inline children as ordered AST nodes rather than pre-rendered
  HTML strings.
- [ ] Preserve source spans for blocks, inlines, metadata, and parse errors.
- [ ] Define AST nodes for text, soft break, hard break, emphasis, strong,
  code span, link, image, raw HTML, and character/entity references.
- [ ] Define AST nodes for paragraphs, headings, thematic breaks, block quotes,
  lists, list items, fenced code, indented code, HTML blocks, and link
  reference definitions.
- [ ] Decide whether link reference definitions remain in the public AST or are
  retained in a document-level lookup table.
- [ ] Add a debug AST serializer for parser tests and diagnostics.

### Block parsing

- [ ] Implement blank-line recognition and paragraph continuation.
- [ ] Implement ATX headings, including closing-sequence rules.
- [ ] Implement setext headings.
- [ ] Implement thematic breaks without confusing them with lists or headings.
- [ ] Implement indented code blocks.
- [ ] Implement fenced code blocks, including info strings and closing-fence
  validation.
- [ ] Implement block quotes, including nested and lazy continuation behavior.
- [ ] Implement unordered lists.
- [ ] Implement ordered lists, including start numbers and interruption rules.
- [ ] Implement nested list indentation and continuation blocks.
- [ ] Determine and encode tight-versus-loose list rendering behavior.
- [ ] Implement link-reference-definition parsing and normalization.
- [ ] Implement CommonMark HTML block recognition.
- [ ] Implement precedence rules among headings, lists, block quotes, code,
  thematic breaks, and paragraphs.
- [ ] Implement tab expansion rules required by the selected CommonMark profile.
- [ ] Add focused tests for each block construct and ambiguous boundary case.

### Inline parsing

- [ ] Implement literal text and backslash escapes.
- [ ] Implement named and numeric character/entity references.
- [ ] Implement soft and hard line breaks.
- [ ] Implement code spans, including delimiter normalization rules.
- [ ] Implement emphasis and strong emphasis using a delimiter-stack algorithm
  compatible with the selected CommonMark profile.
- [ ] Implement inline links and images.
- [ ] Implement full and collapsed reference links and images.
- [ ] Implement shortcut reference links and images.
- [ ] Implement autolinks and URI/email validation behavior.
- [ ] Implement inline raw-HTML recognition.
- [ ] Implement nesting and precedence rules between links, emphasis, code spans,
  and raw HTML.
- [ ] Add focused tests for each inline construct and delimiter edge case.

### Metadata and extensions

- [ ] Parse metadata before CommonMark blocks only when the chosen front-matter
  opener is present in the permitted position.
- [ ] Produce a clear error for malformed metadata rather than silently treating
  it as page content.
- [ ] Validate metadata types, unknown fields, duplicate fields, and required
  fields.
- [ ] Ensure metadata syntax does not alter the meaning of ordinary CommonMark
  documents that do not opt into it.
- [ ] Define an extension-registration mechanism that is disabled by default.
- [ ] Write a compatibility test for every enabled extension against nearby core
  CommonMark syntax.

### Exit criteria

- [ ] The supported CommonMark specification suite passes.
- [ ] Parsing a representative document produces a location-aware AST without
  rendering HTML during parsing.
- [ ] Metadata and all enabled extensions have documented syntax and tests.

---

## Milestone 4 — HTML rendering and theme boundary

### Core renderer

- [ ] Implement HTML escaping for text and attribute values.
- [ ] Render paragraphs, headings, thematic breaks, block quotes, lists, list
  items, and code blocks from AST nodes.
- [ ] Render inline text, emphasis, strong, code spans, links, images, breaks,
  entities, and raw HTML from AST nodes.
- [ ] Render fenced-code language information as a safely escaped CSS class or
  renderer input.
- [ ] Implement the configured raw-HTML policy: render, escape, or omit.
- [ ] Ensure untrusted document text cannot inject markup outside the explicitly
  allowed raw-HTML policy.
- [ ] Define safe URL handling for rendered links and images, including
  disallowed or transformed schemes if required by the site policy.
- [ ] Add golden HTML tests for every AST node type.
- [ ] Compare renderer output against the selected CommonMark fixture output.

### Page shell and theme interface

- [ ] Define the input supplied to a theme: site metadata, page metadata,
  rendered body or body AST, navigation data, and generated asset URLs.
- [ ] Define how a theme contributes layouts, page renderers, and theme assets.
- [ ] Implement a minimal default theme that produces valid complete HTML
  documents.
- [ ] Render `<title>`, language, canonical URL, description, and other
  metadata only after values are escaped and validated.
- [ ] Define the template-selection rule and an error for a missing template.
- [ ] Ensure templates are trusted project code/configuration, not sourced from
  document content.
- [ ] Add a base URL/site URL configuration field and test internal URL joining.
- [ ] Add a theme fixture with more than one layout to test selection.
- [ ] Validate generated HTML in integration tests using an HTML parser or
  validator suitable for automated checks.

### Exit criteria

- [ ] A Markdown/CommonMark page plus metadata renders to a complete, escaped,
  valid HTML page through the default theme.

---

## Milestone 5 — Plugin registry and build pipeline

### Plugin contracts

- [ ] Define reader plugin fields: name, API version, source matcher, priority,
  and read procedure.
- [ ] Define transform plugin fields: name, API version, phase, capabilities,
  ordering constraints, and transform procedure.
- [ ] Define builder plugin fields: name, API version, required capabilities,
  ordering constraints, and build procedure.
- [ ] Define deployer plugin fields: name, API version, target configuration
  schema, and deploy procedure.
- [ ] Define a compatible plugin-API version check and diagnostic.
- [ ] Decide whether plugin names are globally unique or namespaced.
- [ ] Document what plugins may read, write, and mutate.

### Registry and ordering

- [ ] Implement explicit registration from `site.scm`.
- [ ] Reject duplicate plugin names and duplicate capability providers where they
  are ambiguous.
- [ ] Build a dependency graph from before/after and capability requirements.
- [ ] Topologically sort plugins into deterministic execution order.
- [ ] Detect and report ordering cycles with the involved plugin names.
- [ ] Define deterministic tie-breaking for independent plugins.
- [ ] Add tests for valid ordering, missing capability, duplicate plugin, and
  cyclic dependency cases.

### Pipeline execution

- [ ] Implement source discovery for configured content roots.
- [ ] Exclude output, hidden/local-state, and configured ignored directories
  from source discovery.
- [ ] Sort discovered source paths deterministically.
- [ ] Select exactly one reader per eligible source and diagnose zero or multiple
  matches.
- [ ] Read all documents before site-wide transforms/builders require them.
- [ ] Run document transforms in resolved order.
- [ ] Build site-wide indexes after the document set is stable.
- [ ] Run builders in resolved order and collect artifacts plus manifest
  contributions.
- [ ] Preserve source and plugin provenance on diagnostics and artifacts.
- [ ] Stop before emission when any error-severity diagnostic exists.
- [ ] Add an optional strict mode for warnings such as unresolved internal links.

### Standard early plugins

- [ ] Implement the CommonMark reader plugin.
- [ ] Implement a metadata-validation transform plugin.
- [ ] Implement an internal-link-resolution transform plugin.
- [ ] Implement a static-file builder plugin.
- [ ] Implement a basic page builder plugin.
- [ ] Add a small example `site.scm` that composes only these standard plugins.

### Exit criteria

- [ ] A site configuration can explicitly compose readers, transforms, and
  builders to produce a deterministic artifact plan.
- [ ] Plugin errors identify the plugin, source file when relevant, and failed
  contract.

---

## Milestone 6 — Content features and site indexes

### Routes and page artifacts

- [ ] Define route derivation from source paths, explicit metadata routes, and
  slugs.
- [ ] Validate uniqueness of routes before rendering output.
- [ ] Decide whether routes always emit `index.html` directories, explicit file
  names, or both under a documented policy.
- [ ] Implement route-to-output-path conversion.
- [ ] Generate a standard page artifact for each non-draft document.
- [ ] Exclude drafts from production output and define their development-mode
  behavior.
- [ ] Build an in-memory route index used by link validation and builders.
- [ ] Validate links to known internal routes and fragments when strict mode is
  enabled.
- [ ] Add test fixtures for route collisions, invalid routes, drafts, links, and
  fragment references.

### Posts, collections, and navigation

- [ ] Define the initial collection model: posts, pages, and optional custom
  collections.
- [ ] Build a deterministic post index sorted by normalized date and tie-breaker.
- [ ] Implement a blog/post-list page builder.
- [ ] Implement configurable pagination if the initial site needs it.
- [ ] Define tags/categories metadata validation and normalization.
- [ ] Implement tag/category index and archive builders if the site needs them.
- [ ] Define previous/next post navigation semantics.
- [ ] Define a simple explicit navigation source if automatic navigation is not
  sufficient.
- [ ] Add tests for ordering, pagination boundaries, duplicate tags, and empty
  collections.

### Site-wide output

- [ ] Choose the first feed format (Atom or RSS) and define required metadata.
- [ ] Implement the chosen feed builder with escaped XML and stable ordering.
- [ ] Add the alternate feed format only when there is a concrete need.
- [ ] Implement a sitemap builder if enabled by configuration.
- [ ] Generate robots-related files only if the deployment policy requires them.
- [ ] Add canonical URLs to pages, feeds, and sitemap entries consistently.
- [ ] Validate generated XML in tests.

### Exit criteria

- [ ] The example site can build pages, posts, static assets, a site index, and
  the selected site-wide artifacts without manual output editing.

---

## Milestone 7 — Artifact planning, release emission, and manifest

### Artifact validation

- [ ] Validate every artifact path before a writer is opened.
- [ ] Reject duplicate public output paths, including collisions caused by path
  normalization or case-insensitive target filesystems.
- [ ] Define how builders contribute manifest entries without writing manifest
  JSON themselves.
- [ ] Validate artifact content type and cache-policy values.
- [ ] Compute and retain artifact content hashes.
- [ ] Sort artifacts and manifest contributions deterministically.
- [ ] Add tests for collision, traversal, invalid cache policy, and duplicate
  manifest-rule failures.

### Manifest v1

- [ ] Implement a single Goldfish representation of manifest v1.
- [ ] Implement manifest schema validation in the generator.
- [ ] Define and validate redirect source paths, destinations, statuses, and
  loop/conflict behavior.
- [ ] Define and validate header-rule paths, values, and precedence.
- [ ] Reject manifest data that could enable arbitrary code execution or invalid
  HTTP response construction.
- [ ] Serialize manifest JSON deterministically.
- [ ] Add valid and invalid manifest fixtures shared with Go server tests.
- [ ] Document backward- and forward-compatibility behavior for manifest
  versions.

### Transactional release writing

- [ ] Create a unique staging directory outside the active release root.
- [ ] Emit public artifacts into staging with correct directories and file modes.
- [ ] Write `site-manifest.json` only after all manifest contributions validate.
- [ ] Re-read or otherwise verify staged output before activation.
- [ ] Atomically rename or switch an active-release pointer only after a fully
  successful build.
- [ ] Preserve the previous successful release when build or activation fails.
- [ ] Clean stale staging directories safely without deleting an active release.
- [ ] Define release naming/version metadata and include it without compromising
  reproducible-build mode.
- [ ] Add tests that inject write, validation, and activation failures.
- [ ] Add a test proving a failed rebuild leaves the old release byte-for-byte
  available.

### Reproducibility and inspection

- [ ] Add a `check` command that validates a site without activating a release.
- [ ] Add an artifact-plan inspection/debug output for developers.
- [ ] Build the same fixture twice with fixed inputs and verify identical output
  hashes in reproducible mode.
- [ ] Document which fields intentionally make a non-reproducible build differ.

### Exit criteria

- [ ] A successful build creates a validated, self-contained release with
  `public/` and `site-manifest.json`.
- [ ] A failed build never replaces the active release.

---

## Milestone 8 — Go production HTTP server

### Command and configuration

- [ ] Define server flags/environment variables for listen address, release
  directory, production/development mode, logging, and graceful-shutdown
  timeout.
- [ ] Implement configuration precedence and validation.
- [ ] Require an explicit release directory in production mode.
- [ ] Print a clear startup summary without leaking secrets or local paths beyond
  the configured logging policy.
- [ ] Implement signal-driven graceful shutdown with a bounded timeout.

### Manifest loading

- [ ] Define Go types corresponding exactly to manifest v1.
- [ ] Decode manifest JSON with unknown-field and version handling chosen by the
  v1 contract.
- [ ] Validate paths, redirect statuses, headers, matching rules, and conflicts
  independently in Go.
- [ ] Reject malformed or unsupported releases before binding or switching them
  into service.
- [ ] Compile validated redirect and header rules into immutable request-time
  structures.
- [ ] Share manifest test fixtures with the Goldfish generator.
- [ ] Add compatibility tests that ensure generator-produced manifests load in
  the server.

### Request routing and safety

- [ ] Accept `GET` and `HEAD`; return a correct `405 Method Not Allowed` for
  unsupported methods.
- [ ] Define handling for `OPTIONS` only if required by a concrete deployment
  use case.
- [ ] Safely parse and normalize request paths without treating query strings as
  filesystem paths.
- [ ] Reject or safely handle malformed percent-encoding and encoded traversal
  attempts.
- [ ] Ensure filesystem resolution cannot escape the active release's `public/`
  directory, including through symlinks.
- [ ] Apply validated redirect rules in their documented precedence order.
- [ ] Apply validated header rules in their documented precedence order.
- [ ] Resolve `/`, directory routes, trailing slashes, and `index.html` exactly
  as specified by the release contract.
- [ ] Disable directory listings.
- [ ] Serve a configured static 404 page or a minimal correct 404 response.
- [ ] Decide and test whether missing directory slashes redirect or return 404.
- [ ] Add table-driven routing tests for all URL mapping rules.

### Static response behavior

- [ ] Serve files with correct MIME types and explicit charset behavior for text
  formats.
- [ ] Implement `HEAD` without writing a response body.
- [ ] Support conditional requests and range requests using safe standard-library
  primitives where appropriate.
- [ ] Apply artifact/manifest cache policy without allowing invalid values to
  reach responses.
- [ ] Decide whether to generate ETags, rely on standard file metadata, or use
  artifact hashes, then test the choice.
- [ ] Set defensive baseline response headers appropriate for static content,
  subject to documented opt-out rules.
- [ ] Avoid buffering large static files in memory.
- [ ] Set server, read-header, read, write, and idle timeouts appropriate for
  static delivery.
- [ ] Add tests for MIME types, `HEAD`, 404, 405, range, conditional requests,
  cache headers, redirects, and header rules.

### Observability and reliability

- [ ] Implement structured request logs with method, path, status, byte count,
  duration, and request ID policy.
- [ ] Implement startup, manifest-load, and request-error logs.
- [ ] Add a production-safe health endpoint only if deployment orchestration
  needs one; otherwise document its absence.
- [ ] Ensure panic recovery returns a safe 500 response and logs the failure.
- [ ] Add tests for shutdown while a request is in progress.
- [ ] Run Go tests with the race detector for server state and release switching.

### Exit criteria

- [ ] The Go server can serve a generated release correctly in production mode
  with no Goldfish process or source content available.

---

## Milestone 9 — Development server, rebuilds, and live reload

### Watch and build orchestration

- [ ] Define the development command interface: site root, generator command,
  server address, initial-build behavior, and open-browser behavior if any.
- [ ] Watch content, static assets, themes, and trusted configuration/module
  directories.
- [ ] Exclude output, staging, VCS, editor-state, and configured ignored paths
  from watches.
- [ ] Debounce bursts of filesystem events.
- [ ] Serialize rebuilds and coalesce changes that occur during an active build.
- [ ] Capture generator stdout/stderr and present source diagnostics clearly.
- [ ] Keep serving the last successful release if the first build fails; provide
  a clear development-only failure response if no successful release exists.
- [ ] Keep serving the last successful release after subsequent failed rebuilds.
- [ ] Switch server state to a newly validated release atomically.
- [ ] Add unit/integration tests for changed content, changed asset, changed
  configuration, build failure, and recovery after failure.

### Reload protocol

- [ ] Select and document a development-only reload endpoint namespace.
- [ ] Implement the live-reload event stream if Server-Sent Events was selected.
- [ ] Implement reconnect, keepalive, and cleanup behavior for reload clients.
- [ ] Implement a minimal reload client without exposing it in production mode.
- [ ] Decide how reload-client code reaches HTML pages without permanently
  changing production output.
- [ ] Trigger reload only after the new release becomes active.
- [ ] Display build failures in the browser only through a development-only
  endpoint or overlay.
- [ ] Test that production mode has no reload endpoint or reload-script
  injection.

### Developer experience

- [ ] Add a development health/status endpoint that reports the active release
  and last build result without exposing it in production.
- [ ] Report watched paths and ignored paths in verbose mode.
- [ ] Document the local preview workflow and failure-recovery behavior.
- [ ] Test development mode in a real browser before calling live reload done.

### Exit criteria

- [ ] Editing a page, asset, theme, or configuration rebuilds safely and refreshes
  the preview without interrupting service of the last successful release.

---

## Milestone 10 — Custom syntax highlighter

### Design and security

- [ ] Define the highlighter token-tree data model and renderer interface.
- [ ] Choose an implementation strategy for custom grammars/lexers and document
  its supported grammar complexity.
- [ ] Choose the initial languages based on actual site content.
- [ ] Define the language-alias and fenced-code-info-string normalization rules.
- [ ] Define a safe plain-text fallback for unknown or malformed languages.
- [ ] Define a maximum input size and time/resource policy for highlighted code.
- [ ] Ensure highlighted source is always escaped before HTML emission.

### Implementation

- [ ] Implement the highlighter plugin interface from fenced-code AST nodes to
  token trees.
- [ ] Implement plain-text tokenization and rendering first.
- [ ] Implement the first required language lexer and its test corpus.
- [ ] Implement each additional required language lexer and its test corpus.
- [ ] Implement token nesting/overlap rules and invalid-input recovery.
- [ ] Add stable CSS class names for token kinds.
- [ ] Implement at least one theme stylesheet generated or supplied by the
  highlighter/theme module.
- [ ] Add a second theme only after the base token vocabulary is stable.
- [ ] Cache highlighting using source, language, grammar version, and theme
  version.
- [ ] Add tests for injection attempts, unknown languages, Unicode source,
  malformed code, and large inputs.

### Integration

- [ ] Connect fenced-code rendering to the highlighter transform or renderer.
- [ ] Ensure non-highlighted code blocks remain valid and readable without CSS.
- [ ] Add golden page-output fixtures for highlighted code.
- [ ] Document supported languages, aliases, limits, and fallback behavior.

### Exit criteria

- [ ] Required code fences produce escaped, deterministic highlighted HTML, while
  unknown languages produce escaped plain text.

---

## Milestone 11 — Deployment and release operations

### Deployer framework

- [ ] Add a generator deployment command that operates only on a completed,
  validated release.
- [ ] Define target configuration loading and validation for trusted deployer
  modules.
- [ ] Ensure deployers receive a `Release` record rather than rebuilding the
  site themselves.
- [ ] Implement dry-run support where a target can safely provide it.
- [ ] Implement clear deployment logs and nonzero failures.
- [ ] Ensure credentials are read from environment variables, files outside the
  release, or a secret store—not from source content or the manifest.
- [ ] Add tests using local fake targets before contacting any real service.

### Local and rsync deployment

- [ ] Implement a local-copy deployer for integration tests and simple hosting.
- [ ] Validate target paths before local-copy deployment.
- [ ] Implement an rsync-over-SSH deployer if it is the selected first remote
  target.
- [ ] Support versioned remote release directories.
- [ ] Implement an atomic remote active-release switch, such as a symlink or
  server-specific pointer update.
- [ ] Preserve the previously active remote release until the new release has
  been verified.
- [ ] Implement a documented rollback command or procedure.
- [ ] Optionally run a remote health check only after its authentication and
  failure semantics are defined.
- [ ] Add deployment integration tests against a local temporary target.

### Go server packaging

- [ ] Build the Go server reproducibly for each supported deployment platform.
- [ ] Record server version and supported manifest schema version in the binary
  or startup output.
- [ ] Define how the server binary, server configuration, and release directory
  are delivered together.
- [ ] Add a pre-deployment compatibility check between server and manifest
  schema versions.
- [ ] Document service-manager configuration for the selected production host.
- [ ] Document reverse-proxy/TLS configuration boundaries without attempting to
  reimplement them in the server.

### Object storage and CDN (only if needed)

- [ ] Decide whether object storage is actually required after rsync/local
  deployment works.
- [ ] Define content-type, cache-control, immutable-asset, and invalidation
  behavior for the selected provider.
- [ ] Implement an object-storage deployer only after the provider and rollback
  model are selected.
- [ ] Test a deploy/rollback cycle against a non-production bucket.

### Exit criteria

- [ ] A completed release can be deployed, activated, verified, and rolled back
  without rebuilding or accessing content source files on the target host.

---

## Milestone 12 — Quality, security, and performance gates

### Generator quality

- [ ] Run the complete supported CommonMark fixture suite in CI.
- [ ] Run unit tests for every standard reader, transform, builder, artifact
  writer, and deployer.
- [ ] Add regression tests for all reported generator bugs.
- [ ] Test malformed UTF-8, huge files, deeply nested markup, long delimiters,
  and pathological link/reference input.
- [ ] Set and test resource limits or diagnostics for inputs that exceed chosen
  limits.
- [ ] Test duplicate routes, output collisions, symlink handling, and path
  traversal attempts.
- [ ] Test deterministic output on more than one platform if cross-platform
  support is claimed.
- [ ] Review Goldfish configuration trust boundaries and document that a site
  configuration is code with full build-machine authority.

### Server security and correctness

- [ ] Add table-driven tests for percent encoding, Unicode paths, dot segments,
  encoded separators, symlink escapes, and malformed request targets.
- [ ] Fuzz the pure URL/path-resolution functions with Go fuzz tests.
- [ ] Fuzz manifest decoding and validation.
- [ ] Verify redirects cannot produce invalid header values or response splitting.
- [ ] Verify configured headers cannot violate the documented safety policy.
- [ ] Verify development-only endpoints are inaccessible in production mode.
- [ ] Verify logs do not record credentials, authorization headers, or other
  sensitive values.
- [ ] Review response headers and raw-HTML policy for the personal site's threat
  model.
- [ ] Run dependency and static-analysis checks appropriate for Goldfish and Go.

### Performance and operations

- [ ] Benchmark representative full builds and record a baseline.
- [ ] Benchmark parser and syntax-highlighter behavior on large representative
  documents.
- [ ] Benchmark static-file throughput, concurrent requests, range requests,
  and manifest reload behavior.
- [ ] Profile only after a benchmark identifies a real bottleneck.
- [ ] Verify the server does not retain old releases or file handles after an
  active-release switch.
- [ ] Set log rotation/collection expectations for the production environment.
- [ ] Write backup and restore procedures for source, deployment configuration,
  and active releases.

### Exit criteria

- [ ] CI validates the generator/server contract, parser profile, security
  boundaries, and representative end-to-end site builds.

---

## Milestone 13 — Documentation, examples, and first release

### User documentation

- [ ] Document installation of Goldfish, the generator, and the Go server.
- [ ] Document a quick-start site from initialization through local preview.
- [ ] Document the site layout and trusted `site.scm` configuration model.
- [ ] Document the supported CommonMark profile and each custom extension.
- [ ] Document metadata fields, validation rules, route rules, and draft
  behavior.
- [ ] Document theme, reader, transform, builder, artifact, and deployer APIs.
- [ ] Document release layout and manifest v1 in one normative reference.
- [ ] Document production serving, reverse-proxy/TLS boundaries, and deployment.
- [ ] Document troubleshooting for parse errors, plugin order errors, failed
  builds, manifest incompatibility, and development reload failures.
- [ ] Document security boundaries: trusted configuration, non-executable
  content, raw HTML, deployment secrets, and production endpoint behavior.

### Examples

- [ ] Make the minimal example site build and serve in CI.
- [ ] Add a documented blog example with posts, tags or archives, feed, assets,
  and highlighted code if those features are in v1.
- [ ] Add a small custom reader/transform/builder example.
- [ ] Add a theme example with multiple layouts.
- [ ] Add a deployment example using only placeholder hostnames and no secrets.

### Release preparation

- [ ] Review every unchecked v1 task and explicitly defer nonessential work.
- [ ] Run the v1 acceptance checklist against a clean checkout.
- [ ] Test a source-free production deployment using only the generated release
  and server binary/configuration.
- [ ] Test upgrade compatibility from the previous manifest schema, if one
  exists.
- [ ] Tag the first release and publish release notes with known limitations.
- [ ] Establish an issue template for parser, generator, server, security, and
  deployment reports.

---

## Post-v1 work (do not block the first correct release)

- [ ] Add content-hash/dependency-based incremental builds after full builds are
  correct, deterministic, and benchmarked.
- [ ] Add watch-mode selective rebuilding after dependency tracking is proven.
- [ ] Add additional CommonMark extensions only with syntax, AST, rendering, and
  compatibility tests.
- [ ] Add more highlighter languages only when actual site content requires
  them.
- [ ] Add object-storage/CDN deployment only when a target provider is chosen.
- [ ] Add image/media processing only when a documented content workflow needs
  it.
- [ ] Add search generation only after selecting a static search-index contract.
- [ ] Add internationalization only after defining language, route, fallback,
  feed, and sitemap behavior.
- [ ] Add a plugin migration/deprecation mechanism once there is more than one
  published plugin API version.
- [ ] Add external/third-party plugin distribution only after defining trust,
  versioning, and supply-chain policy.

## Explicitly out of scope unless the design is revisited

- Executing Goldfish, Lua, JavaScript, or another language embedded in content
  markup.
- Starting Goldfish or invoking the generator for each visitor HTTP request.
- A generic arbitrary-code server-plugin system.
- A database-backed CMS or general-purpose dynamic web framework.
- Reimplementing TLS termination, HTTP/2/3, CDN behavior, or a full reverse
  proxy in the Go server.
