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

        kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo       # It worked for me

Log in to the UI with:

     Username: admin
     Password: <output from the command>