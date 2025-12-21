This is a guide detailing the architecture and infrastructure used for developing {du}punkto software.

## Languages

{du}punkto software is primarily developed in three languages: Elixir, TypeScript (using Bun), and PHP8.

The language chosen depends on the requirements of the project. Generally, the following guidelines apply:

- Is this a large, complicated web-based application? Use Elixir with PostgreSQL for maintainability (and scalability).
- Is this self-hosted consumer software? Use PHP8 with SQLite to maximise compatibility and ease-of-use.
- Is this highly interactive? Use TypeScript with the Bun runtime.

## Development

{du}punkto makes heavy use of [`Bakefile`](//git.dupunkto.org/meta/dotfiles/blob/master/bin/bake) (a Bash-based alternative to `Makefile`) to streamline development. The following targets are recommended for all projects:

- `setup` installs dependencies.
- `serve` starts a local development server.
- `build` builds or bundles production software.
- `publish` generates and uploads documentation to [docs.dupunkto.org](//docs.dupunkto.org).
- `deploy` builds and deploys the current source tree to production.

## Deployment

{du}punkto software is deployed to the November server using [Lift](//git.dupunkto.org/meta/dotfiles/blob/master/bin/lift), our in-house Docker-based deployment infrastructure.

All containers are exposed behind a Caddy reverse-proxy which manages SSL, sets appropriate security headers and handles proper caching.

To deploy any project, simply run `lift deploy` (although `bake deploy` is preferred, since some projects do additional post-deploy maintenance via their `Bakefile`)
