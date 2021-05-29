# secrethubwarden

[![Project Status: WIP – Initial development is in progress, but there has not yet been a stable, usable release suitable for the public.](https://www.repostatus.org/badges/latest/wip.svg)](https://www.repostatus.org/#wip)

Copy secrets from [Bitwarden](https://bitwarden.com) to [GitHub repository secrets](https://docs.github.com/en/actions/reference/encrypted-secrets) with this bash script.

Inspired by [envwarden](https://github.com/envwarden/envwarden).

## Installation

- Install the [Bitwarden CLI `bw`](https://bitwarden.com/help/article/cli/)
- Install the [GitHub CLI `gh`](https://github.com/cli/cli)
- Install the [JSON CLI processor `jq`](https://stedolan.github.io/jq/)
- `cp secrethubwarden /usr/local/bin`

### Homebrew installation of dependencies

`brew install bitwarden-cli gh jq`

## Usage

1. Create a login vault entry or a secure note in Bitwarden and give it a unique name.
2. Create a [repository secret](https://docs.github.com/en/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository) in GitHub.
3. Write a `.secrethubwarden` file in the `.env` format with `GITHUB_SECRET_NAME=bitwarden_vault_entry_name` on each line.
4. Execute `secrethubwarden` to fetch the secrets from Bitwarden and write them to GitHub.

The script will complain if you are not logged in to Bitwarden (`bw login`) or GitHub (`gh auth login`).

For convenience sake during testing you might want to unlock your Bitwarden vault (`bw unlock`) and store the session key temporarily in your environment (`export BW_SESSION="..."`). Don't forget to lock afterwards (`bw lock`).

### Example `.secrethubwarden` file

```env
MY_SECRET_PASSWORD=secrethubwarden Example Password
MY_SECRET_NOTE=ecb15895-f4ea-428d-bf5d-ad3700483945
```

### About Bitwarden vault items

There are two ways to address Bitwarden vault item on the CLI: Searching by name or giving its item id (a unique GUID). The item id is not exposed in any of the GUI clients, but it can be found through the CLI.

You can search an item like this:

```bash
bw get item <query>
```

The Bitwarden CLI `get` command can only return a single result. If your query would return multiple results, it will return an error.

You can find out its item id like this:

```bash
bw get item <query> | jq .id
```

If an newly created item is not showing up, run `bw sync` to synchronize your CLI client with the current online vault.

## FAQ

### Is this safe?

`secrethubwarden` is a simple bash script that wraps around the Bitwarden and GitHub CLIs. You can inspect it to make sure it's secure and doesn't leak your secrets in any way. I tried to keep it as simple as possible, and also secure. I also tried to follow the Bash best practices as good as possible ([ShellCheck](https://www.shellcheck.net) is running on every push to this repository.)

### What's this for?

If you're keeping credentials out of your code (See [Twelve Factor Apps Factor #3](https://12factor.net))and inject them during deployment into an `.env` file from the GitHub secrets, you might have noticed how tedious this can be. You need to store your `.env` file somewhere, as it can only be written but not read from GitHub.

So you store your `.env` in a password manager like Bitwarden. But any change in the file now has to be updated in the password manager and manually copied to the GitHub secrets. Which is both manual work and error prone.

This script can keep all secrets conveniently updated.

### What's with the name?

I... Don't let me name things.

## Disclaimer

- `secrethubwarden` is not affiliated or connected to Bitwarden or its creators 8bit Solutions LLC in any way.
- `secrethubwarden` is not affiliated or connected to GitHub or its creators GitHub Inc. in any way.

## Author

Created by [Christian Studer](mailto:cstuder@existenz.ch), [Bureau für digitale Existenz](https://bureau.existenz.ch).

## License

[MIT](LICENSE)
