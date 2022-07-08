## GitOps101-backend

During GitOpsDays2022 Weaveworks ran a [GitOps101](https://github.com/kubernetes101/gitopsdays) course which leveraged [GitHub Codespaces](https://github.com/features/codespaces).

This course is great but unfortunately has a hard requirement in that you need to sign up for codespaces and that might not be accessible for everyone. This is an effort to give organisations the capability to reuse the course but on their own infrastructure. It is not designed to provide a production level development platform (HA, durable, recoverable etc) and comes with no guarantees that it will work in all circumstances.

GitOps101-backend was born of the need to provide a multitenant development platform which can be:

- centralised: easier to run/maintain
- colocated: deployed on private/internal infrastructure
- consistent: each 'tenant' gets an identical base image with minimal tooling (git, docker)
- reliable: hard isolation and performance guarantees through the use of microvms
- secure: each tenant provides only a public key and their identifier. only they can log into their environment ensuring anything they use (secrets,  env vars) stay with them

GitOps101-backend leverages [Weaveworks Ignite](https://github.com/weaveworks/ignite) with the ignite/firecracker backend providing virtualised environments per developer/tenant.

### development requirements:

- [vagrant](https://www.vagrantup.com)
- [virtualbox](https://www.virtualbox.org)

### production requirements

- A KVM capable Linux Host (build and tested on Ubuntu 20.04 and 22.04, ymmv)
- An internally routable host IP. The host will port-forward each micro-vm's SSH ip
- VScode installed locally withthe Remote-SSH and Remote-Container Extensions enabled.
- An account with the Git Provider of your choice (Github,Gitlab etc)
- A repo which could be used for enabling developer access via Gitops. The example sister repo to this is [gitopsdays-101-inventory](https://github.com/fire-ant/gitopsdays-101-inventory)
### development demo
install [vagrant](https://www.vagrantup.com) 

get a developer environment up and running with
```
vagrant up
```
in the root of this repository.

#### setup

the local repository is mounted in the vagrant VM so you can either run the commands after ```vagrant ssh``` or open two terminals, one local and one to stream logs.

to setup a micro-vm run the included `vmctl` command (see instructions at [inventory-setup](https://github.com/fire-ant/gitopsdays-101-inventory#inventory-setup)). you can use this inside the vagrant vm in conjunction with `ignite run -f /etc/firecracker/manifest/<your team>/<your vm>/vm.yaml` to decalratively provision development environments. 

the first time you install this will need to pull in images but subsequent builds are cached so you can provision 100's of environments in milliseconds.

If any vms get stuck (there are a few funky situations you might find yourself in when developing) you can use:
```
vagrant ssh -c "sudo rm -rf /var/lib/firecracker/*"
```
to remove any trace of the firecracker vms.

### testing

Once logged in per the previous instructions you can enjoy a cloud native developer sandbox inside a vm.

Each developer environment will come with docker and git installed so once you have logged into the VM, either locally or from outside vagrant (and prefereably through VSCode) you will be able to clone the examle demo [GitOps101](https://github.com/kubernetes101/gitopsdays) and enjoy a fully automate build of kd and installation of gitiops developer tooling.

### GitOps demo

to use this backend in a GitOps like manner we will avail of the `ignited gitops` tooling.
In this instance we cannot use the direct/generic gitops implementation because there is no way to sync keys to be copied into the vms.
What we can do is combine `ignited gitops daemon` with [git-sync](https://github.com/kubernetes/git-sync) to get the best of both git driven infarstructure and file sync securely to the host.

On the host: you will need to run:
```
ignited gitops daemon &
```
which will start the daemon in the background.
for ease, git-sync can be run from this directory using the included [docker-compose])(./docker-compose.yml).
for authentication feel free to extend the file to your needs, following the authentication guide [here](https://github.com/kubernetes/git-sync#flags-which-configure-authentication)

I have included a same
### APPENDIX:

if you'd like to extend this idea these are the two most useful and pertinent confgiuration documents Ive found

[ignite-configuration](https://github.com/weaveworks/ignite/blob/main/docs/ignite-configuration.md)

[vm-config](https://github.com/weaveworks/ignite/blob/main/docs/declarative-config.md)
