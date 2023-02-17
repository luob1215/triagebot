# Triagebot

This is the triage and team assistance bot for the rust-lang organization.

Please see the [wiki] for our documentation, and feel free to contribute edits
if you find something helpful!

[wiki]: https://github.com/rust-lang/triagebot/wiki

## How triagebot works

Triagebot consists of a webserver with several endpoints.
The `/github-hook` and `/zulip-hook` endpoints receive webhook notifications from the respective services.
Triagebot can then respond to those notifications to perform various actions such as adjusting labels.

The Triagebot webserver also includes several other endpoints intended for users to access directly, such as https://triage.rust-lang.org/agenda.

Triagebot uses a Postgres database to retain some state.
In production, it uses [RDS](https://aws.amazon.com/rds/).

The server at https://triage.rust-lang.org/ runs on ECS and is configured via [Terraform](https://github.com/rust-lang/simpleinfra/blob/master/terraform/shared/services/triagebot/main.tf#L8).
Updates are automatically deployed when merged to master.

## Installation

To compile the Triagebot you need OpenSSL development library to be installed (e.g. for Ubuntu-like Linux distributions `sudo apt install libssl-dev`).

Run `cargo build` to compile the triagebot.

## Running triagebot

It is possible to run triagebot yourself, and test changes against your own repository.
Some developers may settle with testing in production as the risks tend to be low, but the more intrepid may find it easier to iterate separately.

The general overview of what you will need to do:

1. Install Postgres. Look online for any help with installing and setting up Postgres (particularly if you need to create a user and set up permissions).
2. Create a database: `createdb triagebot`
3. Provide a way for GitHub to access the Triagebot webserver.
   There are various ways to do this (such as placing it behind a proxy, or poking holes in your firewall).
   Or, you can use a service such as https://ngrok.com/ to access on your local dev machine via localhost.
   Installation is fairly simple, though requires setting up a (free) account.
   Run the command `ngrok http 8000` to forward to port 8000 on localhost.
4. Create a GitHub repo to run some tests on.
5. Configure the webhook in your GitHub repo.
   I recommend at least skimming the [GitHub webhook documentation](https://docs.github.com/en/developers/webhooks-and-events/webhooks/about-webhooks) if you are not familiar with webhooks. In short:

   1. Go to the settings page.
   2. Go to the webhook section.
   3. Click "Add webhook"
   4. Include the settings:

      - Payload URL: This is the URL to your Triagebot server, for example http://7e9ea9dc.ngrok.io/github-hook. This URL is displayed when you ran the `ngrok` command above.
      - Content type: application/json
      - Secret: Enter a shared secret (some longish random text)
      - Events: "Send me everything"
6. Configure the `.env` file:

   1. Copy `.env.sample` to `.env`
   2. `GITHUB_API_TOKEN`: This is a token needed for Triagebot to send requests to GitHub. Go to GitHub Settings > Developer Settings > Personal Access Token, and create a new token. The `repo` permission should be sufficient.
      If this is not set, Triagebot will also look in `~/.gitconfig` in the `github.oauth-token` setting.
   3. `DATABASE_URL`: This is the URL to the Postgres database. Something like `postgres://eric@localhost/triagebot` should work, replacing `eric` with your username.
   4. `GITHUB_WEBHOOK_SECRET`: Enter the secret you entered in the webhook above.
   5. `RUST_LOG`: Set this to `debug`.

7. Run `cargo run --bin triagebot`. This starts the http server listening on port 8000.
8. Add a `triagebot.toml` file to the main branch of your GitHub repo with whichever services you want to try out.
9. Try interacting with your repo, such as issuing `@rustbot-yn` commands or interacting with PRs and issues (depending on which services you enabled in `triagebot.toml`). Watch the logs from the server to see what's going on.

## License

Triagebot is distributed under the terms of both the MIT license and the
Apache License (Version 2.0).

See [LICENSE-APACHE](LICENSE-APACHE) and [LICENSE-MIT](LICENSE-MIT) for details.
