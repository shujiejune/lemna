# Requirements

## Product

- Build a static-first site generator for a personal website.
- Author content in a custom markup language that is compatible with CommonMark.
- Implement the markup parser and HTML renderer in Goldfish Scheme.
- Provide a custom syntax-highlighting tool as an alternative to Shiki.
- Make the generator programmable through trusted Goldfish Scheme
  configuration and modules.
- Make the generator modular and reusable, following Haunt's reader, builder,
  artifact, and publisher/deployer style of composition.
- Do **not** support executable inline code in markup in the initial release.

## Serving

- Provide a built-in HTTP server written in Go using `net/http`.
- Use the same server for local development preview and visitor-facing delivery.
- In development, support rebuild-on-change and live reload or an equivalent
  refresh mechanism.
- Keep runtime behavior separate from the Goldfish Scheme build pipeline; the
  generator produces a static output directory and a versioned server manifest.

## Deployment

- Support custom deployment through pluggable deployer/publisher modules.
- Deploy a complete, independently runnable release: generated output, server
  manifest, and any server configuration or binary required by the target.

See [design.md](design.md) for the proposed architecture and rollout plan.
