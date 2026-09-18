# Keep the backup core independent of host platforms

CloudBarrel will define backup behavior in a portable core, while plain Linux, OpenMediaVault, CasaOS, and constrained routers remain integrations. This avoids making backup correctness depend on one NAS project and keeps platform churn isolated from Jobs, Runs, History, retention, Integrity, and restore behavior.
