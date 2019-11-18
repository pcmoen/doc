# Deploy your application

{% hint style="warning" %}
This article is about a new beta version of NAIS deploy.
It is not yet available to all users, marked by the absence of a team API key.
Get beta access at #pig_deployment on Slack.
{% endhint %}

This section will take you through the deployment of your application using _NAIS deploy_.

NAIS deploy enables you to deploy your application from any continuous integration platform.
Our primary supported platform is GitHub Actions, but you can also deploy from CircleCI, Travis CI, Jenkins, or other tools.

## How it works

In your build pipeline, you create a deployment request using the _NAIS deploy tool_.
Your application will be deployed to Kubernetes, and the deploy tool will wait until your deployment is rolled out, gets an error, or a timeout occurs.
Underway, deployment statuses are continually posted back to _GitHub Deployment API_.
Deployment logs can be viewed on _Kibana_. The link to the logs will be provided by the deploy tool.

## Set it up

1. Your application must have a repository on GitHub.
2. Your GitHub team must have _admin_ access on that repository.
3. Your GitHub team's identifier must match the _Kubernetes team label_ in your `nais.yaml`. There is an example file below.
4. Obtain a _team API key_ from [Vault](https://vault.adeo.no) under the path `/apikey/nais-deploy/<YOUR_TEAM>`. Save the key as a secret named `NAIS_DEPLOY_APIKEY` in your GitHub repository.
5. Follow the guide below titled [performing the deployment](#performing-the-deployment).
6. When things break, see [HELP!](#help).

## Deploy with GitHub Actions

The easiest way of deploying your application to NAIS is using GitHub Actions.
This example workflow will build a Docker container, push it to Github Package Registry,
and deploy the application to the `dev-fss` NAIS cluster.

Start by creating a folder for your workflows in the root of your application repository.
Create a workflow YAML file. You can use our example as a starting point and adjust the
code as needed.

```bash
mkdir -p .github/workflows
touch .github/workflows/deploy.yaml
```

Put the following code into their respective files.
Then, commit your new files and push. This will trigger the workflow, and you can follow its progress
under the _Actions_ tab on your GitHub repository page. You're done!

{% code-tabs %}

{% code-tabs-item title=".github/workflows/deploy.yaml" %}

The example `deploy.yaml` sets up one _workflow_ which triggers every time code is pushed to the repository.
First, the `build` job is triggered. This job will check out the source code, and build a Docker container.
That container is then pushed to _GitHub Package Registry_.

Next, the `deploy` job is triggered, but only if the code was pushed to the `master` branch AND the `build` job succeeded.
The deploy job checks out the source code, creates a template variable file with the Docker image name and tag,
and uses the _NAIS deploy GitHub action_ to create a deployment in the `dev-fss` cluster.

Recommended reading: [Automating your workflow with GitHub Actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions)
is GitHub's official documentation on GitHub Actions. If the code below doesn't make sense,
refer to this documentation for a better understanding.

```yaml
name: Build, push, and deploy

on: [push]

env:
  IMAGE: docker.pkg.github.com/${{ github.repository }}/myapplication:${{ github.sha }}

jobs:

  build:
    name: Build and push Docker container
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - name: Build and publish Docker image
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      run: |
        docker build --tag ${IMAGE} .
        docker login docker.pkg.github.com -u ${GITHUB_REPOSITORY} -p ${GITHUB_TOKEN}
        docker push ${IMAGE}

  deploy:
    name: Deploy to NAIS
    needs: build
    if: github.ref == 'refs/heads/master'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - uses: nais/deploy/actions/deploy@v1
      env:
        APIKEY: ${{ secrets.NAIS_DEPLOY_APIKEY }}
        CLUSTER: dev-fss
        RESOURCE: nais.yml
```
{% endcode-tabs-item %}

{% code-tabs-item title="nais.yml" %}

In this `nais.yaml` file, `{{ image }}` will be replaced by the `$IMAGE` environment variable set in the workflow.
Other environment variables will not be injected, but must be put into a template variables file.

See [deploy action implementation](https://github.com/nais/deploy/blob/master/actions/deploy/entrypoint.sh)
to understand how this is implemented.

```yaml
apiVersion: nais.io/v1alpha1
kind: Application
metadata:
  name: myapplication
  namespace: default
  labels:
    team: myteam
spec:
  image: {{ image }}
```

After `{{ image }}` has been replaced, it will look like:

```yaml
image: docker.pkg.github.com/navikt/myrepository/myapplication:417dcaa2c839b9da72e0189e2cfdd4e90e9cc6fd
```

{% endcode-tabs-item %}

{% code-tabs-item title="Dockerfile" %}

You have to write your own Dockerfile, but if you're just trying this out,
you can use the following example file.

```dockerfile
FROM nginx
```
{% endcode-tabs-item %}

{% endcode-tabs %}

### Advanced configuration

| Environment variable | Default | Description |
| :--- | :--- | :--- |
| APIKEY | (required) | NAIS deploy API key. Obtained from Vault. |
| CLUSTER | (required) | Which [NAIS cluster](../basics/clusters.md) to deploy into. |
| DRY_RUN | `false` | If `true`, run templating and validate input, but do not actually make any requests. |
| OWNER | (auto-detect) | Owner of the repository making the request. |
| PRINT_PAYLOAD | `false` | If `true`, print templated resources to standard output. |
| QUIET | `false` | If `true`, suppress all informational messages. |
| REPOSITORY | (auto-detect) | Name of the repository making the request. |
| RESOURCE | (required) | File containing a Kubernetes resource. Must be JSON or YAML format. |
| TEAM | (auto-detect) | Team making the deployment. |
| VARS | `/dev/null` | File containing template variables. Will be interpolated with the `$RESOURCE` file. Must be JSON or YAML format. |

## Deploy with other CI

You can still use NAIS deploy even if not using GitHub Actions.
Our deployment command line tool supports all CI tools such as Jenkins, Travis or Circle.
Use our Docker image to make deployments.

Example usage:

```
docker login docker.pkg.github.com -u $GITHUB_REPOSITORY -p $GITHUB_TOKEN
docker run -it --rm docker.pkg.github.com/nais/deploy/deploy:v1 \
  /app/deploy \
    --apikey="$NAIS_DEPLOY_APIKEY" \
    --cluster="$CLUSTER" \
    --owner="$OWNER" \
    --repository="$REPOSITORY" \
    --resource="/path/to/nais.yml" \
    --vars="/path/to/resources" \
    --wait=true \
    ;
```

Syntax:

```
--apikey string          NAIS Deploy API key.
--cluster string         NAIS cluster to deploy into.
--deploy-server string   URL to API server. (default "https://deployment.prod-sbs.nais.io")
--dry-run                Run templating, but don't actually make any requests.
--owner string           Owner of GitHub repository. (default "navikt")
--print-payload          Print templated resources to standard output.
--quiet                  Suppress printing of informational messages except errors.
--ref string             Git commit hash, tag, or branch of the code being deployed. (default "master")
--repository string      Name of GitHub repository.
--resource strings       File with Kubernetes resource. Can be specified multiple times.
--team string            Team making the deployment. Auto-detected if possible.
--vars string            File containing template variables.
--wait                   Block until deployment reaches final state (success, failure, error).
```

## Templating

Templates use [Handlebars](https://handlebarsjs.com/) 3.0 syntax.
Both the template and variable file supports either YAML or JSON syntax.

Handlebars content may be escaped in one of two ways, inline escapes or raw block helpers.
Inline escapes created by prefixing a mustache block with \\. Raw blocks are created using {{{{ mustache braces.

Either:
```
\{{escaped}}
```

Or:
```
{{{{raw}}}}
  {{escaped}}
{{{{/raw}}}}
```

## Build status badge

Use the following URL to create a small badge for your workflow/action.

```text
https://github.com/{github_id}/{repository}/workflows/{workflow_name}/badge.svg
```

## HELP!

Don't panic!

Your deployment status page can be found on Github, under the repository you are deploying from. The status page will
live at the URL `https://github.com/navikt/<YOUR_REPOSITORY>/deployments`. Links to deployment logs are available from
this page.

Generally speaking, if the deployment status is anything else than `success`, `queued`, or `pending`, it means that your
deployment failed. _Check the logs_ and _compare messages to the table below_ before asking for support.

If everything fails, and you checked your logs, you can ask for help in the
[\#nais-deployment](https://nav-it.slack.com/messages/CHEQ22Q94) channel on Slack.

| Message | Action |
| :--- | :--- |
| You don't have access to apikey/. | Send a pull request to the _vault-iac_ repository, adding your team to the `terraform/teams` directory. |
| the repository 'foo/bar' does not have access to deploy as team 'Baz' | Is your team name in _lowercase_ everywhere? |
| Repository _foo/bar_ is not registered | Please read the [registering your repository](#registering-your-repository) section. |
| Deployment status `error` | There is an error with your request. The reason should be specified in the error message. |
| Deployment status `failure` | Your application didn't pass its health checks during the 5 minute startup window. It is probably stuck in a crash loop due to mis-configuration. Check your application logs using `kubectl logs <POD>` and event logs using `kubectl describe app <APP>` |
| Deployment is stuck at `queued` | The deployment hasn't been picked up by the worker process. Did you specify a [supported cluster](clusters.md) with `--cluster=<CLUSTER>`? |
| team `foo` does not exist in Azure AD | Your team is not [registered in the team portal](teams.md). |
| "tobac.nais.io" denied the request: user 'system:serviceaccount:default:serviceuser-FOO' has no access to team 'BAR' | The resource you are deploying already exists in the cluster, and the `team` label differs. You must manually change the existing team label (_BAR_) using `kubectl` so that it matches the resource you are deploying (_FOO_). |