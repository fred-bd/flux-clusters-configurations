### Configuring the vault

> Note: all these data was previously configured in my development environment. You need to configure yours too.

```sh
#!/bin/bash
export VAULT_ADDR='http://localhost:8200'

VAULT_JSON=$HOME/projects/tmp/vault.json

ROOT_TOKEN=
DB_HOST=${1:-"192.168.15.10"} # IP of locally PG database

# Unseal Vault (File that we download from vault in first use)
[ -f $VAULT_JSON ] && {
  ROOT_TOKEN=$(cat $VAULT_JSON | jq '.root_token' | xargs)
} || exit 0

vault login -no-print $ROOT_TOKEN

# My cluster kubeconfig
vault secrets enable -path cluster-hosts kv-v2
vault kv put -policy-override=true \
  cluster-hosts/note \
  note-kubeconfig=@$HOME/projects/tmp/my-kubeconfig

# My databases secrets
vault secrets enable -path grafana kv-v2
vault kv put -mount=grafana note-local \
  admin_user=fred \
  admin_password=fran22 \
  admin_email=bede.apps@gmail.com \
  db_host=$DB_HOST:5432 \
  db_name=grafana \
  db_user=grafana \
  db_password=francesca22 

# Local artifactory credentials
vault secrets enable -path helm-repo-credentials kv-v2
vault kv put \
  helm-repo-credentials/chartmuseum \
  user_name=fred
vault kv patch -policy-override=true helm-repo-credentials/chartmuseum \
  user_password="fran" 
```