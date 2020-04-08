# aviary.sh

Distributed configuration management in bash

```
$ av apply
Fetching inventory...
Applying module nginx
Applying module memcached
Done
```

## Principles / How it works

Aviary.sh follows some guiding principles:

- bash is just fine (yes, it is)
- minimize levels of abstraction
- each host takes care of itself

Each host periodically fetches the latest version of the inventory to see what roles should it be performing.  Given whatever roles, the inventory also describes modules (services, programs, etc) that need to be installed and running in order to fulfill the role, and the host configures itself accordingly.


## Concepts

**Inventory** - Git repository where you keep configuration about your servers and what-all they should be doing.  Consists of hosts, roles, and modules.

**Host** - Server virtual or not with a hostname.

**Role** - High-level function that you define (e.g., "application server", or "database server") to be assumed by the host.  Multiple roles may be applied to a host.

**Module** - Service or program (e.g., "node", or "postgres") required to fulfill a role.  A role will usually be comprised of many modules.


## Inventory

The inventory is the git repository where you keep configuration about your servers -- what roles they play, what services they run, etc.  

There are three top-level directories: `hosts`, `roles`, and `modules`.

### Hosts

The hosts directory contains a directory for each host.  In each host directory live the files:
  - `roles` - a list of roles to be fulfilled by the host
  - `modules` - a list of ad-hoc modules to be applied on the host
  - `variables` - list of bash variable assignments local to the host

### Roles

The roles directory contains a directory for each role.  In each role directory live the files:
  - `modules` - a list of modules required to fulfill the given role

### Modules

The modules directory contains a directory for each module.  In each module directory live the files:
  - `apply` - idempotent bash script that will ensure the given service or program is installed and running
  - `variables` - list of bash variable assignments local to the module
  - any other files (templates, configuration files, etc) necessary to support the `apply` script


## av

The command line tool is called `av`.  

```
av - manage configuration for your hosts

Usage:
  av [options] [command] [command-args]

Options:
  --help                   Show this help message
  --version                Show version
  --log-level <level>      Set the log level (trace,debug,info,warn,critical) [default: info]
  --force                  Run even if a pause or run lock is set
  --no-fetch               Don't fetch the inventory

Commands:
  status                   Report the status of the last run of \`apply\` [default]
  fetch                    Update local database by fetching from upstream
  apply                    Fetch and apply roles and their associated modules on this host 
  recover                  Reset run lock file after a failure
  pause                    Set the pause lock to avoid periodic runs while debugging
  resume                   Resume periodic runs after a pause
EOF

```



