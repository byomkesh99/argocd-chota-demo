# ArgoCD - Chota DEMO

## Exiting Cluster

Kind Cluster

## Installation of ArgoCD

     kubectl create namespace argocd
     
     kubectl apply -n argocd \
       --server-side --force-conflicts \
       -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Verify:

     kubectl get pods -n argocd

Outputs:

     Byom-->k get pods -n argocd

      NAME                                                READY   STATUS    RESTARTS   AGE
      argocd-application-controller-0                     1/1     Running   0          29s
      argocd-applicationset-controller-7c7cd47f9d-h8lnr   1/1     Running   0          29s
      argocd-dex-server-6698896998-7hvqs                  1/1     Running   0          29s
      argocd-notifications-controller-644457c445-qf9z7    1/1     Running   0          29s
      argocd-redis-6bcd6966f-skwml                        1/1     Running   0          29s
      argocd-repo-server-6f6657d5cb-wtvfg                 1/1     Running   0          29s
      argocd-server-664495b675-k9d98                      1/1     Running   0          29s

For local testing, expose the UI with port-forwarding:

     kubectl port-forward svc/argocd-server -n argocd 8080:443

Open

    https://localhost:8080      # In browser, you will see the first ArgoCD Welcome page

Get the Argo CD admin password

        argocd admin initial-password -n argocd  # It does not work for me because argocd tool was not installed
        brew install argocd  # Tried next time but the below kubectl commands worked for me

        kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo       # It worked for me

Log in to the UI with:

     Username: admin
     Password: <output from the command>


Then change the password:

     argocd login localhost:8080      # This is require if you wanted to see application status via argocd CLI - Its useful
     argocd account update-password

Current File Structure 

     argocd-chota-demo/
     ├── argocd-app.yaml       <-- here
     ├── readme.md
     ├── steps_followed.md
     └── apps/
         └── demo-app/
             ├── deployment.yaml
             └── service.yaml


Nginx Deployment File:

File: deployment.yaml

     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: demo-app
       labels:
         app: demo-app
     spec:
       replicas: 2
       selector:
         matchLabels:
           app: demo-app
       template:
         metadata:
           labels:
             app: demo-app
         spec:
           containers:
             - name: demo-app
               image: nginx:1.27
               ports:
                 - containerPort: 80

Nginx Service File

File: service.yaml

     apiVersion: v1
     kind: Service
     metadata:
       name: demo-app
       labels:
         app: demo-app
     spec:
       type: ClusterIP
       selector:
         app: demo-app
       ports:
         - name: http
           port: 80
           targetPort: 80


ArgoCD Application YAML file

File: argocd-app.yaml

     apiVersion: argoproj.io/v1alpha1
     kind: Application
     metadata:
       name: demo-app
       namespace: argocd
     spec:
       project: default
       source:
         repoURL: https://github.com/byomkesh99/argocd-chota-demo.git
         targetRevision: main
         path: apps/demo-app
       destination:
         server: https://kubernetes.default.svc
         namespace: demo
       syncPolicy:
         syncOptions:
           - CreateNamespace=true

### IMPORTANT

If you wanted to Enable automatic sync
Update argocd-app.yaml:

      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true

PUSH everything into your GIT Repo

      git init
      git add .
      git commit -m "Initial Argo CD demo app"
      git branch -M main
      git remote add origin https://github.com/byomkesh99/argocd-mini-demo.git
      git push -u origin-argocd main     # I have multiple origin

NOW apply ArgoCD Application kubernetes file

     kubectl apply -n argocd -f argocd-app.yaml

Argo CD will now show the application as OutOfSync, because the resources exist in Git but have not yet been deployed to the cluster.

Sync manually
  In the Argo CD UI:
   Open Applications.
   Click demo-app.
   Click Sync.
   Click Synchronize.
   Or use the CLI:

      argocd app sync demo-app

If the "argocd" cli command does not work try the following "kubectl" command to check the application status

      kubectl get applications -n argocd

You should see:

     NAME       SYNC STATUS   HEALTH STATUS
     demo-app   OutOfSync     Missing

Check the result

     kubectl get pods -n demo
     kubectl get svc -n demo

Since I am  using kind-cluster, expose the Argo CD server with port-forwarding:

     kubectl port-forward svc/argocd-server -n argocd 8080:443

Keep that terminal open. In another terminal, log in:

     argocd login localhost:8080 --username admin --insecure

When prompted, use the initial admin password:

      argocd admin initial-password -n argocd

Then check the app:

      argocd app list
      argocd app get demo-app

To roll back to the previous Git revision:

     argocd app history demo-app
     
     Output:
     SOURCE  https://github.com/byomkesh99/argocd-chota-demo.git
     ID      DATE                            REVISION
     0       2026-10-13 16:54:02 +0200 CEST  main (f990d65)
     1       2026-10-13 16:59:57 +0200 CEST  main (f990d65)
     2       2026-10-13 17:53:04 +0200 CEST  main (b46b57c)
     3       2026-10-13 18:04:08 +0200 CEST  main (b46b57c)
     
     argocd app rollback demo-app <previous-deployment-id>   # Id from the aboube output


Health Checks: Confirm it is healthy

      kubectl get pods -n demo
      kubectl get deployment demo-app -n demo
      kubectl get svc demo-app -n demo
      argocd app get demo-app

      # One of the output
      Health Status: Progressing
      To
      Health Status: Healthy

Access the app locally --
Your Service is ClusterIP, so it is not exposed outside the cluster. For a quick local test, run:

      kubectl port-forward svc/demo-app -n demo 8081:80

Then open in Browser:

      http://localhost:8081

## We need to implement the following Possible next exercises

* Dev/staging/prod environments using Kustomize base + overlays.
* Automatic sync and self-healing for each environment.
* App of Apps pattern to manage multiple applications from one parent Application.
* Private Git repository integration
* RBAC and Argo CD Projects to restrict which teams can deploy where.
* Secrets management using Sealed Secrets, External Secrets Operator, or Vault.
* Progressive delivery using Argo Rollouts.
* Notifications to Slack, email, or webhook when sync or health changes.

## Extending this project - check document - project_idea.md file
