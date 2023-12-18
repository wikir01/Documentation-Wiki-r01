Pour passer en ENABLE, il faut taper system-view

## Ajouter un vlan sur un trunk

```
show current-configuration interface Bridge-Aggregation <numero trunk>
system-view
vlan <numero vlan à rajouter>
name <nom du vlan>
quit
interface Bridge-Aggregation <numero trunk>
port trunk permit vlan <numero vlan à rajouter>
quit
save
```
