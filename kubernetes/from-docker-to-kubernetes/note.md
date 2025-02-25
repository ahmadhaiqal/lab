# This Note taking

## How to get what user group and id to eliminate user access on Pod
1. first Enter to k9s
2. Shell into the Pod
3. use this command
```
cat /etc/passwd
```

## Managing secret with SOP and Age on Flux
1. install sop and Age
```
brew install sops age

age-keygen -o age.agekey

cat age.agekey

```
2. create the secret in kubectl
```
kubectl create secret generic test-secret \
--from-literal=user=admin \
--from-literal=password=mischa \
--dry-run=client \
-o yaml > test-secret.yaml
```

3. export public key generated at step 1
```
export AGE_PUBLIC=age1zd5n9zx0dsdwdggjuvz32ppngn45gk0tcnwlwfs53ev7szefmdfqarudsl
```
4. implement the key into the SOP and the kubectl file generated at step 2 
```
sops --age=$AGE_PUBLIC \
--encrypt --encrypted-regex '^(data|stringData)$' --in-place test-secret.yaml

cat test-secret.yaml
```
5. Add the private key to the cluster(* this step for initial when setup the cluster)
```
cat age.agekey |
kubectl create secret generic sops-age \
--namespace=flux-system \
--from-file=age.agekey=/dev/stdin


# In the clusters/staging directory, add:

#.sops.yaml

creation_rules:
  - path_regex: .*.yaml
    encrypted_regex: ^(data|stringData)$
    age: age1zd5n9zx0dsdwdggjuvz32ppngn45gk0tcnwlwfs53ev7szefmdfqarudsl


# in the apps.yaml Kustomization file, add under spec:
# (same level as prune: true)

decryption:
    provider: sops
    secretRef:
      name: sops-age
```

6. Setup for Cloudflare
```
kubectl create secret generic tunnel-credentials \
--from-file=credentials.json=59c97568-9eda-4dbf-9857-b6cf9cf91259.json --dry-run=client -o yaml > cloudflare-secret.yaml

# Commit and restart deployment.
```
