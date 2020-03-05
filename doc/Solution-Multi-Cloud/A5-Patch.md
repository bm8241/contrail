* [TOC](Multi-Cloud.md#toc)

## A.5 Patch

```
--- /usr/lib/python2.7/site-packages/cli/topology_cli.py.orig
+++ /usr/lib/python2.7/site-packages/cli/topology_cli.py
@@ -76,8 +76,9 @@
 @click.option('--working_dir', default=CURRENT_WORKING_DIR,
               help='path to output directory where terraform json files will be saved')
 @click.option('--skip_validation', default=False, is_flag=True, help='no validation flag')
+@click.option('--skip_controller_validation', default=False, is_flag=True, help='no validation flag')
 @click.option('--limit', default=None, multiple=True)
-def create(topology, secret, working_dir, skip_validation, limit):
+def create(topology, secret, working_dir, skip_validation, skip_controller_validation, limit):
     """
         \b
         This command executes the generate_topology script that creates
@@ -88,7 +89,7 @@
     logger.debug("args: topology:{topo}, secret: {sec}, skip_validation: {vali}, limit: {limit}"
                  .format(topo=topology, sec=secret, vali=skip_validation, limit=limit))
     files_created = generate_topology.run(
-        topology, secret, skip_validation, path=working_dir, limit=limit)
+        topology, secret, skip_validation, skip_controller_validation, path=working_dir, limit=limit)
     for file in files_created:
         logger.info("Topology created: {}".format(file))
```
```
--- /usr/lib/python2.7/site-packages/cli/deploy.py.orig
+++ /usr/lib/python2.7/site-packages/cli/deploy.py
@@ -116,6 +116,7 @@
               help='secret file')
 @click.option('--skip_validation', default=False, is_flag=True,
               help='Skip topology validation')
+@click.option('--skip_controller_validation', default=False, is_flag=True, help='Skip topology validation')
 @click.option('--working_dir', default=CURRENT_WORKING_DIR,
               help='Directory where terraform will run and where files will be created.'
                    ' Current working directory by default.')
@@ -124,7 +125,7 @@
 @click.option('--retry', default=3,
               help='Limit retry')
 @click.option('--limit', default=None, multiple=True)
-def topology(ctx, topology, secret, skip_validation, working_dir, k8s_deployment, retry, limit):
+def topology(ctx, topology, secret, skip_validation, skip_controller_validation, working_dir, k8s_deployment, retry, limit):
     """
         Create cloud instances from topology.
         \b
@@ -154,7 +155,7 @@
                k8s_deployment=k8s_deployment, limit=limit)
     ctx.invoke(topology_cli.create, topology=topology,
                secret=secret, skip_validation=skip_validation, working_dir=working_dir,
-               limit=limit)
+               limit=limit, skip_controller_validation=skip_controller_validation)
     ctx.invoke(topology_cli.build, secret=secret, working_dir=working_dir, retry=retry,
                limit=limit)
```

