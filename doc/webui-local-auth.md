/etc/contrail/config.global.js
```
config.orchestration.Manager = 'none'

config.multi_tenancy = {};
config.multi_tenancy.enabled = false;

config.staticAuth = [];
config.staticAuth[0] = {};
config.staticAuth[0].username = 'admin'; 
config.staticAuth[0].password = 'contrail123'; 
config.staticAuth[0].roles = ['superAdmin'];
```

