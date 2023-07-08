### part-1-             The index file

- The index file is a yaml file called `index.yaml`. It contains some metadata about the package, including the contents of a chart's Chart.yaml file. A valid chart repository must have an index file. The index file contains information about each chart in the chart repository. The `helm repo index` command will generate an index file based on a given local directory that contains packaged charts.

This is the index file of my-charts:

```yaml
apiVersion: v1
entries:
  clarus-chart:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2021-12-07T11:59:09.466396276+03:00"
    description: A Helm chart for Kubernetes
    digest: 712c46edcd85b167881bb644d8de4391eee9acd76aabb75fa2f6e53bedd1c872
    name: clarus-chart
    type: application
    urls:
    - http://<public ip>:8080/clarus-chart-0.1.0.tgz
    version: 0.1.0
generated: "2021-12-07T11:59:09.466104188+03:00"
```

- Now we can upload the chart and the index file to our chart repository using a sync tool or manually.

```bash
cd my-charts
curl --data-binary "@clarus-chart-0.1.0.tgz" http://<public ip>:8080/api/charts
```

- Now we're going to update all the repositories. It's going to connect all the repositories and check is there any new chart.

```bash
helm search repo mylocalrepo
helm repo update
helm search repo mylocalrepo
```

- Let's see how to maintain the chart version. In `clarus-chart/Chart.yaml`, set the `version` value to `0.1.1`and then package the chart.

```bash
helm package clarus-chart
mv clarus-chart-0.1.1.tgz my-charts
helm repo index my-charts --url http://<public-ip>:8080
```

- Upload the new version of the chart and the index file to our chart repository using a sync tool or manually.

```bash
cd my-charts
curl --data-binary "@clarus-chart-0.1.1.tgz" http://<public ip>:8080/api/charts
```

- Let's update all the repositories.

```bash
helm search repo mylocalrepo
helm repo update
helm search repo mylocalrepo
```

- Check all versions of the chart repo with `-l` flag.

```bash
helm search repo mylocalrepo -l
```

- We can also use the [helm-push plugin](https://github.com/chartmuseum/helm-push):

- Install the helm-push plugin. Firstly check the helm plugins.

```
helm plugin ls
helm plugin install https://github.com/chartmuseum/helm-push.git
```

- In clarus-chart/Chart.yaml, set the `version` value to `0.2.0`and push the chart.

```bash
cd
helm cm-push clarus-chart mylocalrepo
```

- Update and search the mylocalrepo.

```bash
helm search repo mylocalrepo
helm repo update
helm search repo mylocalrepo
```

- We can also change the version with the --version flag.

```bash
helm cm-push clarus-chart mylocalrepo --version="1.2.3"
```

- Update and search the mylocalrepo.

```bash
helm search repo mylocalrepo
helm repo update
helm search repo mylocalrepo
```

- Let's install our chart into the Kubernetes cluster.

- Update the `clarus-chart/templates/configmap.yaml` as below.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
  myvalue: "clarus-chart configmap example"
  course: {{ .Values.course }}
  topic: {{ .Values.lesson.topic }}
  time: {{ now | date "2006.01.02" | quote }} 
  count: "first"
```

- Push the chart again.

```bash
helm cm-push clarus-chart mylocalrepo --version="1.2.4"
```

- Install the chart.

```bash
helm repo update
helm search repo mylocalrepo
helm install from-local-repo mylocalrepo/clarus-chart
```

- Check the configmap.

```bash
kubectl get cm
kubectl describe cm from-local-repo-config
```

- This time we will update the release.

- Update the `clarus-chart/templates/configmap.yaml` as below.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
  myvalue: "clarus-chart configmap example"
  course: {{ .Values.course }}
  topic: {{ .Values.lesson.topic }}
  time: {{ now | date "2006.01.02" | quote }} 
  count: "second"
```

- Push the chart again.

```bash
helm cm-push clarus-chart mylocalrepo --version="1.2.5"
helm repo update
```

- Update the release.

```bash
helm upgrade from-local-repo mylocalrepo/clarus-chart
```

- Check the configmap.

```bash
kubectl get cm
kubectl describe cm from-local-repo-config
```

- We can check the available versions with `helm history` command.

```bash
helm history from-local-repo
```

- We can upgrade the release to any version with "--version" flag.

```bash
helm upgrade from-local-repo mylocalrepo/clarus-chart --version 1.2.4
```

- Check the configmap.

```bash
kubectl get cm
kubectl describe cm from-local-repo-config
```

- We can rollback our release with `helm rollback` command.

```bash
helm history from-local-repo
helm rollback from-local-repo 1
```

- Check the configmap.

```bash
kubectl get cm
kubectl describe cm from-local-repo-config
```

- Uninstall the release.

```bash
helm uninstall from-local-repo
```

- Remove the localrepo.

```bash
helm repo remove mylocalrepo
```

## Part-2-           Set up a Helm v3 chart repository in Github

- Create a GitHub repo and name it `mygithubrepo`.

- Produce GitHub Apps Personal access tokens. Go to <your avatar> --> Settings --> Developer settings and click Personal access tokens. Make sure to copy your personal access token now. You won’t be able to see it again!

- Create a GitHub repository locally and push it.

```bash
mkdir mygithubrepo
cd mygithubrepo
echo "# mygithubrepo" >> README.md
git init
git add README.md
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/<your github name>/mygithubrepo.git
git push -u origin main
```

- Create a new chart with following command.

```bash
cd ..
helm create demogitrepo
```

- Package the repo under the `mygithubrepo` folder.

```bash
cd mygithubrepo
helm package ../demogitrepo
```

- Generate an index file in the current directory.

```bash
helm repo index .
```

- Commit and push the repo.

```bash
git add .
git commit -m "demogitrepo chart is added"
git push
```

- Add this repo to your repos. Go to <your repo> --> README.md and click Raw. Copy to address without README.md like below. This will be repo url.

```
https://raw.githubusercontent.com/<github-user-name>/mygithubrepo/main
```

- List the repos and add mygithubrepo.

```bash
helm repo list
helm repo add --username <github-user-name> --password <personel-access-token> my-github-repo 'https://raw.githubusercontent.com/<github-user-name>/mygithubrepo/main'
helm repo list
```

- Let's search the repo.

```bash
helm search repo my-github-repo
```

- Add new charts the repo.

```bash
cd ..
helm create second-chart
cd mygithubrepo
helm package ../second-chart
helm repo index .
git add .
git commit -m "second chart is added"
git push
```

- Update and search the repo.

```bash
helm repo update
helm search repo my-github-repo
```

- Create a release from my-github-repo

```bash
helm install github-repo-release my-github-repo/second-chart
```

- Check the objects.

```bash
kubectl get deployment
kubectl get svc
```

- Uninstall the release.

```bash
helm uninstall github-repo-release
```#my-helm-repo
