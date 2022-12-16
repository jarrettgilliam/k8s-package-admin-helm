# Kubernetes Package Administration with Helm

[Source](https://app.pluralsight.com/library/courses/kubernetes-package-administration-helm/table-of-contents)

## Helm Commands

To list the version of helm you're running

    helm version --short

To list helm environment variables

    helm env

To search for anything on `helm`, you need to have at least on repository. None are set up by default. Let's add the official stable repository.

*Note: this has been deprecated, use [ArtifactHUB](https://artifacthub.io/) instead.*

    helm repo add stable https://charts.helm.sh/stable

At any point you can show which repositories are set up

    helm repo list

To update the list of packages available in a repository

    helm repo update

Now you can search for a package with `helm`

    helm search repo stable/mysql

To include version information, include the `--versions` flag

    helm search repo stable/mysql --versions

Here's how to get more information about a given chart

    helm show chart stable/mysql

You can also look at the README file

    helm show readme stable/mysql

You can also list default values

    helm show values stable/mysql

Before installing, you can do a dry run debug install

    helm install mysql stable/mysql --dry-run --debug 

To install a package, run `helm install` followed by a name for the release and then the repo_name/chart_name. Since we don't specify a version, it will install the latest

    helm install mysql stable/mysql

To install a specific version, include the `--version` flag

    helm install mysql stable/mysql --version 1.6.3

To upgrade a release

    helm upgrade mysql stable/mysql --version 1.6.4

To upgrade or install (great for scripts)

    helm upgrade --install mysql stable/mysql --version 1.6.4

To rollback a release to a specific revision number

    helm rollback mysql 1

To list releases in the current namespace and show their status, run `helm list`

    helm list

To include uninstalled releases, add the `--all` flag

    helm list --all

To get more information about a release instance

    helm status mysql

To list the information about all the Kubernetes objects that were created by combining the templates in the chard with the default values of the chart

    helm get manifest mysql

Get custom value information

    helm get values mysql

Gets more information about a chart. In this instance it shows how to connect to the database.

    helm get notes mysql

Get all information about a helm release instance

    helm get all mysql

Get release instance history

    helm history mysql

Helm history is stored as secrets in k8s which is pretty nice

    kubectl get secrets

Uninstall the helm release (but keep history)

    helm uninstall mysql --keep-history

Now delete everything about the release (including history)

    helm delete mysql

To download and dissect a chart

    helm pull stable/mysql --untar

To create a helm chart of your own

    helm create ourchart

To locally render the k8s yaml files and see what they would look like

    helm template ourchart

To deploy (install) this chart

    helm install ourchart ./ourchart

Verify the deployed image

    kubectl get deployment -o jsonpath='{ .items[*].spec.template.spec.containers[*].image }'

Now upgrade the `containerImage`

    helm upgrade ourchart ./ourchart --set containerImage=nginx:1.18

to package this chart for a helm repository

    helm package ./ourchart --destination ./charts

Now let's push our packaged chart to a helm repository. We can deploy [ChartMuseum](https://chartmuseum.com/) to our local Kubernetes cluster to test with

    helm install chartmuseum stable/chartmuseum --set env.open.DISABLE_API=false

This will spit out some notes telling us our to open the port to allow pushing packaged charts

    export POD_NAME=$(kubectl get pods --namespace default -l "app=chartmuseum" -l "release=chartmuseum" -o jsonpath="{.items[0].metadata.name}")
    echo http://127.0.0.1:8080/
    kubectl port-forward $POD_NAME 8080:8080 --namespace default

Now add ChartMuseum as a repository

    helm repo add chartmuseum http://127.0.0.1:8080

Now we can finally push our package to ChartMuseum using curl from the `./charts` directory

    curl --data-binary "@ourchart-0.1.0.tgz" http://localhost:8080/api/charts

Now update our package listing and search for our chart

    helm repo update
    helm search repo chartmuseum/ourchart

To get the location of our helm repositories

    helm --help

To generate an index manually for a helm repository (from the `./charts` folder)

    helm repo index .
