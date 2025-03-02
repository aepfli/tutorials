
## Create your first project
Duration: 5:00

A project in Keptn is the logical unit that can hold multiple (micro)services. Therefore, it is the starting point for each Keptn installation.

To get all files you need for this tutorial, please clone the example repo to your local machine.

<!-- command -->

```
git clone --branch 0.15.0 https://github.com/keptn/examples.git --single-branch

cd examples/onboarding-carts
```


Create a new project for your services using the `keptn create project` command. In this example, the project is called *sockshop*. Before executing the following command, make sure you are in the `examples/onboarding-carts` folder.

**Recommended:** Create a new project with Git upstream:

To configure a Git upstream for this tutorial, the Git user (`--git-user`), an access token (`--git-token`), and the remote URL (`--git-remote-url`) are required. If a requirement is not met, go to [the Keptn documentation](https://keptn.sh/docs/0.17.x/manage/git_upstream/) where instructions for GitHub, GitLab, and Bitbucket are provided.

Let's define the variables before running the command:

```
GIT_USER=gitusername
GIT_TOKEN=gittoken
GIT_REMOTE_URL=remoteurl
```

Now let's create the project using the `keptn create project` command.

```
keptn create project sockshop --shipyard=./shipyard.yaml --git-user=$GIT_USER --git-token=$GIT_TOKEN --git-remote-url=$GIT_REMOTE_URL
```

Negative
: Please note that the Git repo **must not** be initialized - it has to be an empty repository without any branch or commit. [See the docs for more details](https://keptn.sh/docs/0.17.x/manage/git_upstream/).

**Alternatively:** If you don't want to manually create a Git repository and upstream, you can use the automatic provisioning feature. For that follow the [keptn-gitea-provisioner-service](https://github.com/keptn-sandbox/keptn-gitea-provisioner-service) install instructions.

After setting up the `keptn-gitea-provisioner-service` you can create a project without providing your Git credentials.

<!-- command -->
```
keptn create project sockshop --shipyard=./shipyard.yaml
```

For creating the project, the tutorial relies on a `shipyard.yaml` file as shown below:

```
apiVersion: "spec.keptn.sh/0.2.2"
kind: "Shipyard"
metadata:
  name: "shipyard-sockshop"
spec:
  stages:
    - name: "dev"
      sequences:
        - name: "delivery"
          tasks:
            - name: "deployment"
              properties:
                deploymentstrategy: "direct"
            - name: "test"
              properties:
                teststrategy: "functional"
            - name: "evaluation"
            - name: "release"
        - name: "delivery-direct"
          tasks:
            - name: "deployment"
              properties:
                deploymentstrategy: "direct"
            - name: "release"

    - name: "staging"
      sequences:
        - name: "delivery"
          triggeredOn:
            - event: "dev.delivery.finished"
          tasks:
            - name: "deployment"
              properties:
                deploymentstrategy: "blue_green_service"
            - name: "test"
              properties:
                teststrategy: "performance"
            - name: "evaluation"
            - name: "release"
        - name: "rollback"
          triggeredOn:
            - event: "staging.delivery.finished"
              selector:
                match:
                  result: "fail"
          tasks:
            - name: "rollback"
        - name: "delivery-direct"
          triggeredOn:
            - event: "dev.delivery-direct.finished"
          tasks:
            - name: "deployment"
              properties:
                deploymentstrategy: "direct"
            - name: "release"

    - name: "production"
      sequences:
        - name: "delivery"
          triggeredOn:
            - event: "staging.delivery.finished"
          tasks:
            - name: "deployment"
              properties:
                deploymentstrategy: "blue_green_service"
            - name: "release"
        - name: "rollback"
          triggeredOn:
            - event: "production.delivery.finished"
              selector:
                match:
                  result: "fail"
          tasks:
            - name: "rollback"
        - name: "delivery-direct"
          triggeredOn:
            - event: "staging.delivery-direct.finished"
          tasks:
            - name: "deployment"
              properties:
                deploymentstrategy: "direct"
            - name: "release"

        - name: "remediation"
          triggeredOn:
            - event: "production.remediation.finished"
              selector:
                match:
                  evaluation.result: "fail"
          tasks:
            - name: "get-action"
            - name: "action"
            - name: "evaluation"
              triggeredAfter: "15m"
              properties:
                timeframe: "15m"

```


This shipyard contains three stages: dev, staging, and production. Later in the tutorial, deployments will be made in three corresponding Kubernetes namespaces: `sockshop-dev`, `sockshop-staging`, and `sockshop-production`.

* **dev** will have a direct (big bang) deployment strategy and functional tests are executed
* **staging** will have a blue/green deployment strategy with automated approvals for passing quality gates as well as quality gates which result in warnings. As configured, performance tests are executed.
* **production** will have a blue/green deployment strategy without any further testing. Approvals are done automatically for passed quality gates but manual approval is needed for quality gate evaluations that result in a warning. The configured remediation strategy is used for self-healing in production.


Positive
: To learn more about a *shipyard* file, please take a look at the [Shipyard specification](https://github.com/keptn/spec/blob/master/shipyard.md).

Let's take a look at the project that we have just created in the Keptn's Bridge. To access it, visit the URL contained in `$KEPTN_BRIDGE_URL` using the command:

<!-- command -->
```
echo $KEPTN_BRIDGE_URL
```

You can view the Keptn Bridge credentials using the following commands: 

<!-- command -->
```
echo Username: $(kubectl get secret -n keptn bridge-credentials -o jsonpath="{.data.BASIC_AUTH_USERNAME}" | base64 --decode)
echo Password: $(kubectl get secret -n keptn bridge-credentials -o jsonpath="{.data.BASIC_AUTH_PASSWORD}" | base64 --decode)
```

You will find the just created project in the bridge with all stages.
![bridge](./assets/bridge-new.png)
![bridge](./assets/bridge-empty-env.png)
