# Deploy observability stack

## Intro

This repository contains some instructions and examples for setting up an observability cluster using Plural and connecting a workload cluster to ship metrics to the observability cluster. Fields that should be changed with user input in the instructions or files are marked between `< >`.

## Pre

- Create a new service account on app.plural.sh (email doesn't need to exist)
- Create a git repository on GitHub and clone it locally
- Run `plural login --service-account <service-account-email>`
- Inside the git repository, run `plural init` and fill in the cloud provider configs

## Install the software

- Install Mimir by running `plural bundle install mimir mimir-azure`
- Install Loki by running `plural bundle install loki loki-azure`
- Install Trace Shield by running `plural bundle install trace-shield trace-shield-azure`
- Install Grafana by running `plural bundle install grafana azure-grafana`

## Deploying the cluster

- Run `plural build` to build the artifacts for the deployment
- Run `plural deploy` to deploy the cluster
- Commit the changes to git and push upstream

## Integrate Grafana with Trace Shield

- Login to Trace Shield by going to it's frontend hostname
- Go to GraphiQL at `<traceshield-hostname>/graphiql` to get to API GUI to create OAuth2 Clients, tenants, etc
- Use GraphiQL to create a new OAuth2 Client for Grafana and save the Client ID and Client Secret from the response (see `grafana-oauth2-client.graphql` for the GraphQL mutation)
- Modify `grafana/helm/grafana/values.yaml` in the Git deployment repo to configure Grafana to allow logging in with Trace Shield (see `grafana-values.yaml` for an example of the values file)
- Run `plural deploy` and commit and push the changes to GitHub

## Workload cluster setup

- Create an OAuth2 Client for the tenant using GraphiQL (see `tenant-oauth2-client.graphql` an example of the GraphQL mutation)
- Create a new tenant using GraphiQL and add the OAuth2 Client as a `metricsWriter` (see `tenant.graphql` for the GraphQL mutation)
- Deploy [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) helm chart with the values from file `prom-stack-workload-cluster-values.yaml`
- Install [Grafana Agent Operator](https://github.com/grafana/helm-charts/tree/main/charts/agent-operator) helm chart in the workload cluster
- Deploy the agent resources configured to ship metrics to the Mimir public hostname and configure OAuth2 authentication for the remote write. See folder `grafana-agent-resources` for some examples for the resources
