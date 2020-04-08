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

Each host periodically fetches the latest version of the inventory to see what roles should it be performing.  Given whatever roles, the inventory also describes modules (services, programs, etc) that need to be installed and running in order to fulfill the role, and the host configures itself accordingly.  The inventory is a git repo with a specific directory structure and idempotent scripts to apply modules.

## Getting started


#### Installation

Install from the command line, on a box to be managed by aviary.sh:

```bash
curl aviary.sh/install | sudo bash
```


#### Inventory setup

Configure your inventory if you don't have one yet:

```bash
mkdir inventory
cd inventory
mkdir roles modules hosts
git init
git commit -a -m "initial commit"
```

Configure and push your repo to an origin:

```bash
git remote add origin $my_origin_url
git push -u origin
```

Set your inventory url in config:

```bash
echo inventory_git_url=$my_origin_url >> /var/lib/aviary/config
```
Of course the dealings with git will be non-interactive, so you need to either set up ssh keys or access tokens in order to make that work.  In GitHub for example, find "Personal Access Tokens" under your account settings.  Once you have an access token, you can include it in the git url, e.g., `https://<access_token>@github.com/organization/aviary-inventory.git`


#### Modules

Add our first module:

```
mkdir modules/motd
```

Create an idempotent script to configure the message of the day that users will see when they log in to this box.  In the inventory, create `modules/motd/apply` with these contents:

```bash
# inventory/modules/motd/apply
cat <<<EOF > /etc/motd
"Ever make mistakes in life? Let’s make them birds. Yeah, they’re birds now."
--Bob Ross
EOF
```

Now create this host in the inventory, and add the motd module to be applied:

```bash
mkdir hosts/($hostname)
echo motd > hosts/($hostname)/modules
```

It's usually better practice to apply roles (essentially role-specific sets of modules) to hosts, but you can also apply ad-hoc modules directly if you like, as we're doing here.

Now check in these contents and push them up to the inventory repo.

#### Running `av`

To apply our module, run `av apply`.

```bash
# av apply
Fetching inventory...
Applying motd...
Done.
```

Inspect `/etc/motd` to see that our motd module has in fact been applied.

Running `av status` (or just `av`) tells us how the host is configured and what is its status:

```bash
# av status
STATUS OK
```

#### Templates and variables

In order to make configuration files dynamic, we can use template files and variable interpolation.  Templates are {{ moustache }} style, and variables can be configured at various levels of the inventory directory hierarchy in `variables` bash files containing variable assignments.

Let's spruce up our `motd` module.  In the inventory, let's add a template in `modules/motd`:

```
# inventory/modules/motd/motd.template

Welcome to {{ hostname }}

"Ever make mistakes in life? Let’s make them birds. Yeah, they’re birds now."
--Bob Ross
```

Set the `hostname` variable in a `variables` file:

```
# inventory/modules/motd/variables
hostname=$(hostname)
```

Set the `apply` script to interpolate the template:

```
# inventory/modules/motd/apply

source template
source variables

template motd.template > /etc/motd
```


## Concepts

**Inventory** - Git repository where you keep configuration about your servers and what-all they should be doing.  Consists of hosts, roles, and modules.

**Host** - Server virtual or not with a hostname.

**Role** - High-level function that you define (e.g., "application server", or "database server") to be assumed by the host.  Multiple roles may be applied to a host.

**Module** - Service or program (e.g., "node", or "postgres") required to fulfill a role.  A role will usually be comprised of many modules.


## Inventory

The inventory is the git repository where you keep configuration about your servers -- what roles they play, what services they run, etc.  

There are three top-level directories: `hosts`, `roles`, and `modules`.  Files in each directory are newline-delimited text files, unless specified otherwise

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
  status                   Report the status of the last run of `apply` [default]
  fetch                    Update local database by fetching from upstream
  apply                    Fetch and apply roles and their associated modules on this host 
  recover                  Reset run lock file after a failure
  pause                    Set the pause lock to avoid periodic runs while debugging
  resume                   Resume periodic runs after a pause
```



