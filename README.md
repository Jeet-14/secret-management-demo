# secret-management-demo
secret-management-demo using vault + eso

Step 1: Spin up local k8s cluster with KinD/minikube cluster

Step 2: Add helm repo for hashicorp (https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-raft-deployment-guide#setup-helm-repo)

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com 
helm search repo hashicorp/vault 
helm install vault hashicorp/vault --namespace vault --create-namespace
```

Step 3: Login into vault pod, init and unseal

login into vault pod
```bash
kubectl exec -it vault-0 -n vault -- sh
```

Init vault using following
```bash
vault operator init 

Unseal Key 1: 69EFBGsxxxxxxxm545mKNUmUPLs/9f 
Unseal Key 2: hlPQMxxxxxxxxxxxxL+d5WQEMZiQbJ 
Unseal Key 3: TucIxxxxxxxxxxxxxxxs/yOnGYlMN1 
Unseal Key 4: kCJKxxxxxxxxxxxxxMsHSUOAGHEFe/ 
Unseal Key 5: a5rRxxxxxxxxxxxxxxxxxmXmMMwhN9TS 
..

Initial Root Token: hvs.D76XxxxxxZMQ51 
...
..
```

Unseal the vault
```bash
/ $ vault operator unseal 

Unseal Key (will be hidden):  

Key                Value 

---                ----- 

Seal Type          shamir 
Initialized        true 
Sealed             true 
Total Shares       5 
Threshold          3 
Unseal Progress    1/3 
Unseal Nonce       f36a7e65-0632-9a8d-223c-b2b76e63ebb1 
Version            1.19.0 
Build Date         2025-03-04T12:36:40Z 
Storage Type       file 
HA Enabled         false 

..
```
On third time sealed should be 'false'
```bash
Unseal Key (will be hidden):  

Key             Value 

---             ----- 

Seal Type       shamir 
Initialized     true 
Sealed          false 
```

Step 4: Login into vault, check members, access the UI

```bash
vault login

Token (will be hidden):  

Success! You are now authenticated. The token information displayed below is already stored in the token helper. You do NOT need to run "vault login" again. Future Vault requests will automatically use this token. 

Key                  Value 

---                  ----- 

token                hvs.D76XxxxxxxxxxTZMQ51 
token_accessor       YSfx2SVWyuNnPqV01aTKXdu8 
token_duration       ∞ 
token_renewable      false 
token_policies       ["root"] 
identity_policies    [] 
policies             ["root"]
``` 
check the member lists: 
 
```
vault operator members 

Host Name    API Address               Cluster Address                        Active Node    Version    Upgrade Version    Redundancy Zone    Last Echo 

---------    -----------               ---------------                        -----------    -------    ---------------    ---------------    --------- 

vault-0      http://10.244.0.5:8200    https://vault-0.vault-internal:8201    true           1.19.0     n/a                n/a                n/a 

```

Step 5: Either with UI or with vault pod create a secret:

```bash
vault secrets enable -path my-k8s-secret kv-v2
vault kv put my-k8s-secret/devdb password=s3cr3t
```


Patch svc and port-forward to access the UI:

```bash
kubectl patch svc vault -n vault --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]' 
```
```bash
kubectl port-forward svc/vault -n vault 8200:8200 
```

[Img here]

Step 6: Install ESO with helm

```bash
helm repo add external-secrets https://charts.external-secrets.io
```

```bash
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace
```

Step 7: Now create cr for secretstore and externalsecret with given files.


 
create secretstore using given file and make sure it is ready: [Change this content]

```bash
kubectl get secret/devdb-password -n database-ns  -o jsonpath='{.data.password}' | base64 -d
```

