# k8s-Probes


##   1. Liveness Probe,Detects if the app is alive. If fails → restart container.
##   2. Readiness Probe,Detects if app is ready to serve traffic. If fails → remove from Service endpoints.
##   3. Startup Probe,Prevents premature restarts for slow-boot apps.
