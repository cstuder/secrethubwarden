# secrethubwarden

[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)

Synchronize secrets from [Bitwarden](https://bitwarden.com) to [GitHub repository secrets](https://docs.github.com/en/actions/reference/encrypted-secrets) with this Bash script.

Inspired by [envwarden](https://github.com/envwarden/envwarden).

## Installation

- Install the [Bitwarden CLI `bw`](https://bitwarden.com/help/article/cli/)
- Install the [GitHub CLI `gh`](https://github.com/cli/cli)
- Install the [JSON CLI processor `jq`](https://stedolan.github.io/jq/)
- `cp secrethubwarden /usr/local/bin`

### Homebrew installation of dependencies

`brew install bitwarden-cli gh jq`

## Usage

1. Create a login entry or a secure note in Bitwarden and give it a unique name.
2. Write a `.secrethubwarden` file in the `.env` format with `GITHUB_SECRET_NAME=bitwarden_vault_entry_name` on each line.
3. Execute `secrethubwarden` to fetch the secrets from Bitwarden and write them to GitHub.

The script will complain if you are not logged in to Bitwarden (`bw login`) or GitHub (`gh auth login`).

If a GitHub repository secret does not exist, it will be created.

During testing you might want to unlock your Bitwarden vault (`bw unlock`) and store the session key temporarily in your environment (`export BW_SESSION="..."`). Don't forget to lock afterwards (`bw lock`).

### Example `.secrethubwarden` file

```env
MY_SECRET_PASSWORD=secrethubwarden Example Password Name
MY_SECRET_NOTE=ecb15895-f4ea-428d-bf5d-ad3700483945
```

### Adressing Bitwarden vault items

There are two ways to address Bitwarden vault items on the CLI: Searching by name or giving its item ID (a unique GUID). The item ID is not exposed in any of the GUI clients, but it can be found through the CLI.

You can search an item like this:

```bash
bw get item <query>
```

The Bitwarden CLI `get` command can only return a single result. If your query would return multiple results, it will generate an error.

The alternative is to use the `bw list items --search <query>` command for searching multiple items.

You can find the item id like this:

```bash
bw get item <query> | jq .id
```

If a newly created item is not showing up, run `bw sync` to synchronize your CLI client with the current online vault.

## FAQ

### What is `secrethubwarden` for?

It is good practice to keep credentials out of your code (See [Twelve Factor Apps Factor #3](https://12factor.net)) and inject them during deployment into the build from GitHub secrets, for example in the form of an `.env` file. This means you need to store your unencrypted `.env` file somewhere, as it can only be written, but not read from GitHub.

So you store your `.env` in a password manager like Bitwarden. But any change in the file now has to be updated in the password manager and manually copied to the GitHub secrets. Which is manual work, tedious and error prone.

This script can keep all secrets conveniently updated. It is not intended as a CI/CD script, but is used before launching a CI/CD process.

### Is it safe?

`secrethubwarden` is a Bash script that wraps around the Bitwarden and GitHub CLIs. You can inspect it to make sure it is secure and does not leak your secrets in any way. I tried to keep it as simple as possible, and also secure. I also tried to follow the Bash best practices as good as possible, i.e. [ShellCheck](https://www.shellcheck.net) is running on every push to this repository.

### Writing an `.env` file in a GitHub Action

```yaml
- name: Write .env
  run: |
    echo $ENV_FILE | tr ' ' '\n' > .env
  shell: bash
  env:
    ENV_FILE: ${{secrets.DOTENV}}
```

Note: Writing a multiline secret string directly into a file replaces all newlines with spaces. The `tr` command converts them back to newlines. The disadvantage is that the secret itself cannot contain any spaces.

### Pros & Cons

- Pro: Interactive guidance.
- Pro: Colorful output.

- Con: Not atomic, no transactions. If a problem occurs during the execution, only half the secrets might get updated.
- Con: It is kinda slow.

### What does the error message `mac failed` mean?

Nothing, ignore it. (Some Bitwarden issue, irrelevant for this script, I think.)

### What's with the name?

Don't let me name things.

## Disclaimer

- `secrethubwarden` is not affiliated or connected to Bitwarden or its creators 8bit Solutions LLC in any way.
- `secrethubwarden` is not affiliated or connected to GitHub or its creators GitHub Inc. in any way.

## Author

Created by [Christian Studer](mailto:cstuder@existenz.ch), [Bureau für digitale Existenz](https://bureau.existenz.ch).

## Credits

- Inspired by [envwarden](https://github.com/envwarden/envwarden).
- Based on the [minimal safe Bash template](https://betterdev.blog/minimal-safe-bash-script-template/).

## License

[MIT](LICENSE)
