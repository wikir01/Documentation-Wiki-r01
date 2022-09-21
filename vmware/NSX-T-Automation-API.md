
Depuis Postman (pour explorer l'API):
-------------------------------------

### Get all NSX-T Config:

GET https://nsxmgr/policy/api/v1/infra?filter=type-

![](/nsx-t/postman_nsx-t_all.png)

### Get all Security Policies

GET https://nsxmgr/policy/api/v1/infra/domains/default/security-policies

![](/nsx-t/security_policies.png)

### Create Security Policy and Rules

PATCH https://nsxmgr/policy/api/v1/infra/

Body:
