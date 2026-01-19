# gitops-cluster
this is a gitops root repository with argocd to manage all argocd applications

## 1. App of Apps (Orchestration)
## 2. AppProject  (Gouvernance)
## 3. RBAC (Qui peut faire quoi)
### 3.1 Creation d'un user pour l'equipe guestbook
Dans un premier, on veut ajouter un user "guestbook-dev", pour le faire,
argocd propose de modifier la configmap "argocd-cm" en ajoutant la configuration ci dessous:
```yaml
data:
  accounts.guestbook-dev: login
  accounts.guestbook-dev.enabled: "true"
```
###

### 3.2. Creation d'un role RBAC pour l’équipe guestbook
On va définir un rôle pour le user créé ci haut qui:
* peut voir
* peut sync
* peut opérer
* uniquement sur l’app guestbook
* uniquement dans le project guestbook-project
```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.default: role:readonly

  policy.csv: |
    # Rôle dev-guestbook
    p, role:dev-guestbook, applications, get, guestbook-project/guestbook, allow
    p, role:dev-guestbook, applications, list, guestbook-project/guestbook, allow
    p, role:dev-guestbook, applications, sync, guestbook-project/guestbook, allow
    p, role:dev-guestbook, applications, action/*, guestbook-project/guestbook, allow

    # Mapper un utilisateur local (exemple)
    g, guestbook-dev, role:dev-guestbook
```
### 3.3. Redemarrage de argocd
* Redemarrage de argocd-server deployment
```bash
kubectl -n argocd rollout restart deployment argocd-server
```
* connection en admim
```bash
argocd login localhost:8080 --username admin --insecure
```
* Check users
  avec la sortie attendu
``` bash
$ argocd account list
NAME           ENABLED  CAPABILITIES
admin          true     login
guestbook-dev  true     login
```
### 3.4. Setting du password
```
argocd account update-password --account guestbook-dev
```
