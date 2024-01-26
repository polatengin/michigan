# Deploy a dotnet MVC app to IIS using an FTP step in GitHub Actions

This is a sample project to demonstrate how to deploy a dotnet MVC app to IIS using an FTP step in GitHub Actions.

## Getting Started

In order to differentiate `DEV` and `PROD` environments, there are two _Environments_ created on the repository settings:

- `DEV` environment is triggered when a pull request is created.
- `PROD` environment is triggered on every push to the `main` branch.

> **Note:** For the sake of simplicity, _GitHub Actions_ have been configured to be triggered manually, by having a `workflow_dispatch` event. This can be changed to a `push` event on the `main` branch and a `pull_request` event on the `main` branch.
>
> ```yaml
> on:
>   workflow_dispatch:
> ```
>
> ```yaml
> on:
>   push:
>     branches: [ main ]
> ```
>
> ```yaml
> on:
>   pull_request:
> ```
>
> For more information, see [Events that trigger workflows](https://docs.github.com/en/actions/reference/events-that-trigger-workflows).

There are four _Environments Secrets_ created for each _Environments_ on the repository settings:

- `FTP_SERVER`
- `FTP_USERNAME`
- `FTP_PASSWORD`
- `FTP_FOLDER`

Based on the _Environment_ that is being deployed, the corresponding _Environment Secret_ is used in the workflow.

```yaml
environment:
  name: DEV
```

First two steps of the workflows are to build and publish the dotnet MVC app.

```yaml
- name: Build dotnet project
  working-directory: ./src
  run: |
    dotnet restore
    dotnet build --configuration Release --no-restore

- name: Publish dotnet project
  working-directory: ./src
  run: dotnet publish --configuration Release --no-build --no-restore --output ./deploy
```

> **Note:** `dotnet publish` command is used to publish the dotnet MVC app to the `./deploy` folder.

Next step is to log the diff between what exists in the `./deploy` folder and what exists on the FTP server.

```yaml
- name: 📃 Print what changes will be made
  uses: SamKirkland/FTP-Deploy-Action@v4.3.4
  with:
    local-dir: ./src/deploy/
    server: ${{ secrets.FTP_SERVER }}
    username: ${{ secrets.FTP_USERNAME }}
    password: ${{ secrets.FTP_PASSWORD }}
    server-dir: ${{ secrets.FTP_FOLDER }}
    dry-run: true
```

> **Note:** `dry-run` option is used to log the diff between what exists in the `./deploy` folder and what exists on the FTP server.
