# Mini Project Idea

Project Description: 
I would like use 2 or 4 page tabbed website where I will be filling form of VM details and in the other tab I will see the VM details. lets take 5 field form 1) VM name 2) Ip address 3) Environment 4) Application name 5) Any comment . Thats it.

## Following Architechtire:

      React frontend  →  Node.js/Express API  →  PostgreSQL

The five fields map directly to your requirement:

     * vm_name
     * ip_address
     * environment
     * application_name
     * comments

Planned project folder structure:

      vm-inventory/
      ├── frontend/                 # React app
      │   ├── src/
      │   ├── Dockerfile
      │   └── package.json
      ├── backend/                  # Node.js + Express API
      │   ├── src/
      │   ├── Dockerfile
      │   └── package.json
      └── k8s/
          ├── base/
          │   ├── namespace.yaml
          │   ├── postgres-secret.yaml
          │   ├── postgres-statefulset.yaml
          │   ├── postgres-service.yaml
          │   ├── backend-deployment.yaml
          │   ├── backend-service.yaml
          │   ├── frontend-deployment.yaml
          │   └── frontend-service.yaml
          └── overlays/
              ├── dev/
              └── prod/


The following codes there in GITHub Repo

00-namespace.yaml
01-postgres-secret.yaml
02-postgres-configmap.yaml
03-postgres-statefulset.yaml
04-postgres-service.yaml
05-backend-deployment.yaml    # After "docker build -t byomkesh99/vm-inventory-backend:1.0.0 ." path mention in this code
06-backend-service.yaml

Docker build steps:

      docker build -t byomkesh99/vm-inventory-backend:1.0.0 .
      docker login
      docker push byomkesh99/vm-inventory-backend:1.0.0

Create the Argo CD Application:
      Create argocd-vm-inventory.yaml at the root of your Git repository:

File: argocd-vm-inventory.yaml

     apiVersion: argoproj.io/v1alpha1
     kind: Application
     metadata:
       name: vm-inventory
       namespace: argocd
     spec:
       project: default
       source:
         repoURL: https://github.com/byomkesh99/argocd-chota-demo.git
         targetRevision: main
         path: apps/vm-inventory
       destination:
         server: https://kubernetes.default.svc
         namespace: vm-inventory
       syncPolicy:
         syncOptions:
           - CreateNamespace=true

Push all the newly created files to Github repo.

Then Apply It.

        kubectl apply -n argocd -f argocd-vm-inventory.yaml

Then sync:

        argocd app sync vm-inventory

Verify the deployment and the services

      kubectl get pods -n vm-inventory
      kubectl get svc -n vm-inventory
      kubectl get pvc -n vm-inventory
      argocd app get vm-inventory

Test the API
 Use port-forwarding:

        kubectl port-forward svc/vm-backend -n vm-inventory 3000:3000

In another terminal, test health:

        curl http://localhost:3000/healthz

        OUTPUT:
        {"status":"ok"}

Now tried to push some data with POST Curl command and see the details:

      curl -X POST http://localhost:3000/api/vms \
        -H "Content-Type: application/json" \
        -d '{
          "vm_name": "test-vm-01",
          "ip_address": "10.0.0.10",
          "environment": "dev",
          "application_name": "demo-app",
          "comment": "First test VM"
        }'

Display the VM details:

        curl http://localhost:3000/api/vms

        OUTPUT:
        [{"id":1,"vm_name":"test-vm-01","ip_address":"10.0.0.10","environment":"dev","application_name":"demo-app","comment":"First test VM","created_at":"2026-10-14T10:53:13.056Z"}]% 


## For Frontend we are using React JS

Installation - Here is the installation Path: ....Kubernetes/practice/vm-inventory/

      npm create vite@latest frontend -- --template react
      cd frontend
      npm install

Folder structure look like

      vm-inventory/
      ├── backend/
      └── frontend/
          ├── src/
          ├── index.html
          ├── package.json
          ├── vite.config.js
          ├── Dockerfile
          └── nginx.conf

All the above file exist in the Github repo. Along with that.

Few notes - 1) src/App.jsx file replaced from default conf file 2) src/App.css file replaced with default conf file. 3) frontend/Dockerfile file created for Nginx reverse proxy purpose.

Created frontend/.dockerignore with the followings

      node_modules
      dist
      .git

Docker build and pushed for nginx to run from container:

      docker build -t byomkesh99/vm-inventory-frontend:1.0.0 .
      docker push byomkesh99/vm-inventory-frontend:1.0.0

Then added  Kubernetes manifests

      /apps/vm-inventory/07-frontend-deployment.yaml
      /apps/vm-inventory/08-frontend-service.yaml

Pushed everything to Github

     cd <ArgoCD Folder>
     git add .
     git commit -m "Add React frontend for VM inventory"
     git push origin-argocd main

Refresh SYNC

     argocd app get vm-inventory
     argocd app sync vm-inventory

To Open the website - Use port-forwarding:

     kubectl port-forward svc/vm-frontend -n vm-inventory 8082:80

     And then Open the following link on Browser
     http://localhost:8082

As of now THIS IS OUR WORKFLOW

     Browser → React UI → nginx → Express API → PostgreSQL

So, we are aboe access https://argocd.local/ and http://vm-inventory.local/ perfectly.

Next, our mission to implement Kustomize overlay and separate Dev and Prod env. Dev will be in Dev Kind Cluster and Prod will be in Prod Kind cluster.

By intacting old configuration here we will do the modification.

apps/
├── vm-inventory/               # Existing working application—unchanged
└── vm-inventory-envs/          # New Kustomize configuration
    ├── base/
    └── overlays/
        ├── dev/
        └── prod/

Planned ENV:

| Environment        | Cluster      | Namespace         |
| ------------------ | ------------ | ----------------- |
| Existing baseline  | kind-kind    | vm-inventory      |
| New dev            | kind-kind    | vm-inventory-dev  |
| New simulated prod | kind-vm-prod | vm-inventory-prod |

