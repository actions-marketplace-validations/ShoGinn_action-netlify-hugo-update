# Update HUGO Version to latest in Netlify TOML

A GitHub action for keeping the hugo version in netlify.toml up to date.

_special thanks to [@platenio](https://github.com/platenio/action-netlify-toml-update-hugo)_

Checks the latest version via `curl` then updates the `netlify.toml`

Particularly useful when paired with the [create-pull-request][1] action to automatically submit a PR with the changes to the repository or as a step before building/testing a hugo site.

## Usage

```yaml
      - uses: actions/checkout@v3
      - name: Update Hugo Version for Netlify
        uses: shoginn/action-netlify-update-hugo@v1
```

### Action Inputs

|        Name         | Description                                              |     Default      |
| :-----------------: | :------------------------------------------------------- | :--------------: |
| `netlify-toml-path` | Path to the `netlify.toml` config file from project root | `./netlify.toml` |

### Action Outputs

The following output can be used by subsequent workflow steps.

- `updatedConfig` - Whether or not this action updated the `netlify.toml` config file
- `latestVersion` - The latest version of hugo available if installed via this action

### Action Behavior

The default behavior of this action is to check the version of Hugo, compare all version pins for Hugo in `netlify.toml` against that version, and update the `netlify.toml` config file to use the latest version.

#### With Alternate Config Path

If the `netlify-toml-path` parameter is specified the action will look for an update that file instead of `./netlify.toml`.

## Reference Example

The following workflow explicates all of the parameters for reference purposes.
Check the [defaults](#action-inputs) to avoid setting inputs unnecessarily.

```yaml
## This will not run as is!
jobs:
  buildWithLatestHugo:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Update Hugo Version For Netlify
        id: update-hugo-version-netlify
        uses: shoginn/action-netlify-update-hugo@v1
        with:
          netlify-toml-path: ./site/netlify.toml
```

## Repository Update PR Example

The following workflow is an example of using this action to keep your hugo project repository's dependencies up to date on a schedule; it uses the [create-pull-request][1] action for the last step.

```yaml
name: Update Hugo Netlify Toml

on:
  workflow_dispatch:
  schedule:
    - cron: "0 0 * * *"

jobs:
  update_hugo_netlify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Update Hugo Version for Netlify
        id: update-hugo-version-netlify
        uses: shoginn/action-netlify-update-hugo@v1
      - name: Create Pull Request
        id: createpr
        uses: peter-evans/create-pull-request@v4
        with:
          commit-message: "chore(config): Update Hugo version in Netlify TOML"
          title: "chore(config): Update Hugo version in Netlify TOML"
          body: |
            Updated Hugo version in netlify config file to: `${{ steps.update-hugo-version-netlify.outputs.latestVersion }}`

            Auto-generated by [create-pull-request][1] and [update-hugo-netlify-toml][1]

            [1]: https://github.com/peter-evans/create-pull-request
            [2]: https://github.com/shoginn/action-netlify-hugo-update
          branch: update-hugo-netlify-toml
          base: main
          delete-branch: true
      - name: Check outputs
        run: |
         echo "Pull Request Number - ${{ steps.createpr.outputs.pull-request-number }}"
         echo "Pull Request URL - ${{ steps.createpr.outputs.pull-request-url }}"
```

## License

[MIT](LICENSE)

[1]: https://github.com/peter-evans/create-pull-request
