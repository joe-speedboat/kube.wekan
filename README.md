# Setup Wekan on k3s with Helm

## BUGS in v7.94 (latest)
* Admin Panel > Layout
  * Display Authentication Method: No
  * Default Authentication Method: LDAP
  * Save > Haning forever, does not save
* WeKan Container Var
  * DEFAULT_AUTHENTICATION_METHOD: ldap
  * Does not have any effect

## HowTo Setup
```
genpasswd ()
{ 
    local l=$1;
    [ "$l" == "" ] && l=32;
    tr -dc A-Za-z0-9-_. < /dev/urandom | head -c ${l} | xargs
}

URL=task.domain.tld
MAIL_FROM='wekan@domain.tld'
MAIL_URL="smtp://wekan:Task1t@mail.domain.tld:587/?secureConnection=true"
MONGO_APP=$(genpasswd)
MONGO_ROOT=$(genpasswd)

VALS=values.yml_freeipa

LDAP_HOST=freeipa.domain.tld
LDAP_BASEDN="dc=domain,dc=tld"
LDAP_AUTHENTIFICATION_USERDN='uid=wekan-bind,cn=users,cn=compat,dc=domain,dc=tld'
LDAP_AUTHENTIFICATION_PASSWORD='WeKANbind123...change_ME'
LDAP_GROUP_FILTER_GROUP_NAME='wekan_app_user_group'
LDAP_SYNC_ADMIN_GROUPS='wekan_app_admin_group'

test -f custom_vars && . custom_vars && echo "WARNING: custom_vars sourced"

test -f myvalues.yaml && echo myvalues.yaml is present, remove first && sleep 5
test -f myvalues.yaml && exit 1

cp -av $VALS myvalues.yaml
sed -i \
  -e "s|__URL__|${URL}|g" \
  -e "s|__MAIL_FROM__|${MAIL_FROM}|g" \
  -e "s|__MAIL_URL__|${MAIL_URL}|g" \
  -e "s|__MONGO_ROOT__|${MONGO_ROOT}|g" \
  -e "s|__MONGO_APP__|${MONGO_APP}|g" \
  -e "s|__LDAP_HOST__|${LDAP_HOST}|g" \
  -e "s|__LDAP_BASEDN__|${LDAP_BASEDN}|g" \
  -e "s|__LDAP_AUTHENTIFICATION_USERDN__|${LDAP_AUTHENTIFICATION_USERDN}|g" \
  -e "s|__LDAP_AUTHENTIFICATION_PASSWORD__|${LDAP_AUTHENTIFICATION_PASSWORD}|g" \
  -e "s|__LDAP_GROUP_FILTER_GROUP_NAME__|${LDAP_GROUP_FILTER_GROUP_NAME}|g" \
  -e "s|__LDAP_SYNC_ADMIN_GROUPS__|${LDAP_SYNC_ADMIN_GROUPS}|g" \
  myvalues.yaml

cat myvalues.yaml

NS=wekan
kubectl create namespace $NS
kubectl config set-context --current --namespace=$NS

helm install wekan wekan/wekan -f myvalues.yaml
```
