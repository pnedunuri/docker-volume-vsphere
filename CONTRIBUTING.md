# Contributing

## Topics
* [Code contribution guidelines](#code-contribution-guidelines)
* [Bug filing guidelines](#bug-filing-guidelines)
* [Testing and CI](#testing-and-ci)

## Code Contribution guidelines
### Dev Setup and debugging help
Read the [FAQ on the Wiki](https://github.com/vmware/docker-volume-vsphere/wiki#faq)
### Pull Requests
* Create a fork or branch (if you can) and make your changes
   * Branch should be suffixed with github user id: (branch name).(github user id) Example: mydevbranch.kerneltime
   * If you want to trigger CI test runs for pushing into a branch prefix the branch with runci/ Example: runci/mylatestchange.kerneltime
   * If you want to skip running CI test e.g. *Documentation change/Inline comment change* or when the change is in `WIP` or `PREVIEW` phase, add `[CI SKIP]` or `[SKIP CI]` to the PR title.
* Each PR must be accompanied with unit/integration tests
* Add detailed description to pull request including reference to issues.
* Add details of tests in "Testing Done".
* Locally run integration tests.
* Squash your commits before you publish pull request.
* If there are any documentation changes then they must be in the same PR.
* We don't have formal coding conventions at this point.
* Add `[WIP]` or `[PREVIEW]` to the PR title to let reviewers know that the PR is not ready to be merged.
Make sure your code follows same style and convention as existing code.

See  [Typical Developer Workflow](#typical-developer-workflow) to get started.


### Merge Approvals:
* Pull request requires a minimum of 2 approvals, given via "Ship it",
"LGTM" or "+1" comments.
* Author is responsible for resolving conflicts, if any, and merging pull request.
* After merge, you must ensure integration tests pass successfully.
Failure to pass test would result in reverting a change.

Do not hesitate to ask your colleagues if you need help or have questions.
Post your question to Telegram or drop a line to cna-storage@vmware.com

## Cutting a new release

Once a release has been tagged, the CI system will start a build and push the binaries to GitHub Releases page. Creating the release on GitHub is a manual process but the binaries for it will be auto generated.

Typical steps followed.

Tag a release
```
git tag -a -m "0.11 Release for Jan 2017" 0.11
```

Push the tag
```
git push origin 0.11
```

Check to see if the new release shows up on GitHub and the CI build has started.


Generate the change log
```
docker run -v `pwd`:/data --rm muccg/github-changelog-generator -u vmware -p docker-volume-vsphere -t <github token> --exclude-labels wontfix,invalid,duplicate,could-not-reproduce
```

Manually eye ball the list to make sure Issues are relevant to the release (Some times labels such as wontfix have not been applied to an Issue)

Head to GitHub and author a new release add the changelog for the tag created.

**Note**: Some manual steps are required before publishing new release as shown below.
1. Download deliverables from Github release page
2. Remove VIB/DEB/RPMs from the ```Downloads``` sections
3. Perform steps from internal Confluence page to sign the VIB. Update [vDVS_bulletin.xml](https://github.com/vmware/docker-volume-vsphere/blob/master/docs/misc/vDVS_bulletin.xml#L19) to keep it current with the release
4. Head to [Bintray](https://bintray.com/vmware/product/vDVS/view) to publish signed VIB
5. Push managed plugin to docker store
6. Add ```Downloads``` section with direct links; take [Release 0.13](https://github.com/vmware/docker-volume-vsphere/releases/tag/0.13) as the reference

### Publish vDVS managed plugin to Docker Store
**Note**: not automated as of 04/04/17

To push plugin image
```
DOCKER_HUB_REPO=vmware EXTRA_TAG= VERSION_TAG=latest make all
DOCKER_HUB_REPO=vmware EXTRA_TAG= VERSION_TAG=<version_tag> make all
```

### Publish signed VIB to Bintray
1. Create a new version to upload [signed VIB](https://bintray.com/vmware/vDVS/VIB)
2. Upload files at newly created version page
3. Publish the newly uploaded VIB by marking it ```public```

Update documentation following steps listed below.

## Documentation

Documentation is published to [GitHub
Pages](https://vmware.github.io/docker-volume-vsphere/) using
[mkdocs](http://www.mkdocs.org/).

1. Documentation is updated each time a release is tagged.
2. The latest documentation for the master can be found in
   [docs](https://github.com/vmware/docker-volume-vsphere/tree/master/docs) in
   markdown format

To update documentation
```
# 1. Checkout the gh-pages branch
git checkout gh-pages
# 2. Go to jekyll-docs directory
cd jekyll-docs
# 3. Build the jekyll site
docker run --rm --volume=$(pwd):/srv/jekyll -it jekyll/jekyll:stable jekyll build
# 4. Remove the old site and copy the new one.
rm -rvf ../documentation
mv _site ../documentation
# 6. Push to GitHub
git add documentation
git commit
git push origin gh-pages
```

## Bug filing guidelines
* Search for duplicates!
* Include crisp summary of issue in "Title"
* Suggested template:

```
Environment Details: (#VMs, ESX, Datastore, Application etc)

Steps to Reproduce:
1.
2.
3.

Expected Result:

Actual Result:

Triage:
Copy-paste relevant snippets from log file, code. Use github markdown language for basic formatting.

```
## Typical Developer Workflow
### Build, test, rinse, repeat
Use `make` or `make dockerbuild` to build.

Build environment is described in README.md. The result of the build is a set
of binaries in ./build directory.

In order to test locally, you'd need a test setup. Local test setup automation is planned but but not done yet, so currently the environment has to be set up manually.

Test environment  typically consist of 1 ESX and 3 guest VMs running inside of the
ESX. We require ESX 6.0 and later,
and a Linux VM running  Docker 1.13+ enabled for  plain text TCP connection, i.e.
Docker Daemon running with "-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock"
options. Note that both tcp: and unix: need to be present
Please check  "Configuring and running Docker"
(https://docs.docker.com/engine/admin/configuring/)  page on how to configure this - also there is a github link at the bottom, for systemd config files.

If test environment has Docker running on Photon OS VMs, VMs should be configured to accept traffic on port 2375. Edit /etc/systemd/scripts/iptables, and add following rule at the end of the file. Restart iptable service after updating iptables file.

```
#Enable docker connections
iptables -A INPUT -p tcp --dport 2375 -j ACCEPT
```

To deploy the plugin and test code onto a test environment we support a set of
Makefile targets. There targets rely on environment variables to point to the
correct environment.

Environment variables:
- You **need** to set ESX and VM1, VM2 and VM3 environment variables

- The build will use your `username` (the output of `whoami`) to decide on the `DOCKER_HUB_REPO` name to complete our move to use [managed plugin](https://github.com/vmware/docker-volume-vsphere/blob/master/plugin/Makefile#L30).
If you want to user another DockerHub repository you need to set `DOCKER_HUB_REPO` as environment variable.

- Test verification is extended using gvmomi integration and `govc` cli is **required to set** following environment variables.
  - `GOVC_INSECURE` as `1`
  - `GOVC_URL` same as `ESX IP`
  - `GOVC_USERNAME` & `GOVC_PASSWORD`: user credentials logging in to `ESX IP`

- You **need** to set following environment variables to run swarm cluster related tests. You **need** to configure swarm cluster inorder to run swarm related testcase otherwise there will be a test failure.
  - `MANAGER1` - swarm cluster manager node IP
  - `WORKER1` & `WORKER2` - swarm cluster worker node IP

**Note**: You need to manually remove older rpm/deb installation from your test VMs. With [PR 1163](https://github.com/vmware/docker-volume-vsphere/pull/1163), our build/deployment script start using managed plugin approach.

Examples:
```
# Build and deploy the code. Also deploy (but do not run) tests
DOCKER_HUB_REPO=cnastorage ESX=10.20.105.54 VM_IP=10.20.105.201 make deploy-all
```

or

```
# clean, build, deploy, test and clean again
export ESX=10.20.105.54
export VM1=10.20.105.121
export VM2=10.20.104.210
export VM2=10.20.104.241
export DOCKER_HUB_REPO=cnastorage
export GOVC_INSECURE=1
export GOVC_URL=10.20.105.54
export GOVC_USERNAME=root
export GOVC_PASSWORD=<ESX login passwd>
export MANAGER1=10.20.105.122
export WORKER1=10.20.105.123
export WORKER2=10.20.105.124

make all
```

Or just put the 'export' statement in your ~/.bash_profile and run

```
# just build
make
# build, deploy, test
make build-all deploy-all test-all
# only remote test
make testremote
```

If the code needs to run in debugger or the console output is desired.
Login to the machine kill the binary and re-run it manually.

```
Standard invocation on ESX:
python -B /usr/lib/vmware/vmdkops/bin/vmdk_ops.py

Standard invocation on VM: (as root)
/usr/local/bin/docker-volume-vsphere
```

To remove the code from the testbed, use the same steps as above (i.e define
ESX, VM1, VM2 and VM3) and use the following make targets:

```
# remove stuff from the build
make clean
# remove stuff from ESX
make clean-esx
# remove stuff from VMs
make clean-vm
# clean all (all 3 steps above)
make clean-all
```

If additional python scripts are added to the ESX code, update the vib description file to include them.

```
./esx_service/descriptor.xml
```
### git, branch management and pull requests
This section is for developers working directly on our git repo,
and is intended to clarify recommended process for internal developers.

We use git and GitHub, and follow customary process for dev branches and pull
requests. The text below assumes familiarity with git concepts. If you need a
refresher on git, there is plenty of good info on the web,
e.g. http://rogerdudler.github.io/git-guide/is.

The typical source management workflow is as follows:
* in a local repo clone, create a branch off master and work on it
(e.g. `git checkout -b <your_branch> master`)
* when local test is done and it's time to validate with CI tests, push your branch
to GitHub and make sure CI passes
(e.g. `git push origin <your_branch>`)
* when you branch is ready, rebase to latest master
and squash commit if needed (e.g. `git fetch origin; git rebase -i origin/master`).
Each commit should have
a distinct functionality you want to handle as a unit of work. Then re-push your
branch, with `git push origin <your_branch>`, or, if needed,
`git push -f origin <your_branch>`
* create a PR on github using already pushed branch. Please make sure the title,
description and "tested:" section are accurate.
* When applying review comments, create a new commit so the diff can be easily seen
by reviewers.
* When ready to merge (see "Merge Approvals" section above), squash multiple
"review" commits (if any) into one, rebase to master and re-push.
This is done to make sure CI still passes.
* Then merge the PR.
* With any questions/issues about the steps, telegram to cna-storage

## Managing GO Dependencies

Use [gvt](https://github.com/FiloSottile/gvt) and check in the dependency.
Example:
```
make gvt # Start docker image used to build which includes gvt
gvt fetch github.com/docker/go-plugins-helpers/volume
git add vendor
git commit -m "Added new dependency go-plugins-helpers"
exit
```

## Testing and CI

The work flow for coding looks like this

- Each checkin into a branch on the official repo will run the full set of
  tests.

- On Pull Requests the full set of tests will be run.
  - The logs for the build and test will be posted to the CI server.
  - Due to security concerns the artifacts will not be published.
    https://github.com/drone/drone/issues/1476

- Each commit into the master operation will run the full set of tests
  part of the CI system.
  - On success a docker image consisting of only the shippable docker
    plugin is posted to docker hub. The image is tagged with latest
    (and <branch>-<build>). Any one can pull this image and run it to
    use the docker plugin.

A typical workflow for a developer should be.

- Create a branch, push changes and make sure tests do not break as reported
  by the CI system.
- When ready post a PR. This will trigger a full set of tests on ESX. After all
  the tests pass and the review is complete the PR will be merged in. If the PR
  depends on new code checked into master, merge in the changes as a rebase and
  push the changes to the branch.
- When there is a failure unrelated to your PR, you may want to *Restart* the failed CI run instead of pushing dummy commit to trigger CI run.
- When the PR is merged in the CI system will re-run the tests against the master.
  On success a new Docker image will be ready for customers to deploy (This is only
  for the docker plugin, the ESX code needs to be shipped separately).

## CI System

We are using the CI system that has been up by the CNA folks (@casualjim, @frapposelli & @mhagen-vmware).
The CI system is based on https://drone.io/  URL for the server is  https://ci.vmware.run/
To be able to change the integration between CI and GitHub, first become admin using the self serve portal.

Behind the firewall there is a HW node running ESX that hosts 2 ESX VMs (v6.0u2 and v6.5). Each VM in turn contains a VSAN datastore as well as a VMFS datastore with 2 VMs per datastore.

There is a webhook setup between github repo and the CI server. The CI server uses
.drone.yml file to drive the CI workflow.

The credentials for Docker Hub deployment is secured using http://readme.drone.io/usage/secrets/

### Running CI/CD on a dev setup
Each developer can run tests part of the CI/CD system locally in their sandbox.

* Setup:
The current CI/CD workflow assumes that the testbed consists of:
   - One ESX
   - 2 Linux VMs running on the same datastore in the ESX.

* Install [Drone CLI](https://github.com/drone/drone-cli)
```
curl http://downloads.drone.io/drone-cli/drone_linux_amd64.tar.gz | tar zx
sudo install -t /usr/local/bin drone
```

* If not already done, checkout source code in $GOPATH
```
mkdir -p $GOPATH/src/github.com/vmware
cd $GOPATH/src/github.com/vmware
git clone https://github.com/vmware/docker-volume-vsphere.git
```

* Setup ssh keys on linux nodes & ESX

Linux:
```
export NODE=root@10.20.105.54
cat ~/.ssh/id_rsa.pub | ssh $NODE  "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"
```

ESX:
```
cat ~/.ssh/id_rsa.pub | ssh $NODE " cat >> /etc/ssh/keys-root/authorized_keys"
```
Test SSH keys, login form the drone node should not require typing in a password.

* Run drone exec

```
cd $GOPATH/src/github.com/vmware/docker-volume-vsphere/
drone exec --trusted --yaml .drone.dev.yml -i ~/.ssh/id_rsa -e VM1=<ip VM1> -e VM2=<ip VM2> -e ESX=<ip ESX>
```

## Capturing ESX Code Coverage
Coverage is captured using make coverage target (On CI it is called using drone script).
User can configure the setup and use this target to see the coverage.
### Setup ESX box to install python coverage package
* Install https://coverage.readthedocs.io/en/coverage-4.4.1/ on your machine using pip <br />
```Desktop$ pip install coverage```
- scp the coverage package dir installed on ur machine i.e. <Python Location>/site-packages/coverage to /lib64/python3.5/site-packages/ on ESX box. <br />
eg:- ```Desktop$ scp -r /Library/Python/3.5/site-packages/coverage root@$ESX:/lib64/python3.5/site-packages/```
- scp the coverage binary i.e. /usr/local/bin/coverage to /bin/ on ESX box <br />
eg:- ```Desktop$ scp /usr/local/bin/coverage root@$ESX:/bin/```
* On ESX create sitecustomize.py inside ```/lib64/python3.5/``` <br />
The content of sitecustomize.py is
```
import coverage
coverage.process_startup()
```
* On ESX create coverage.rc file in ESX home dir as ```vi /coverage.rc``` <br />
The content of coverage.rc is
```
[run]
omit=*_test.py
parallel=True
source=/usr/lib/vmware/vmdkops
```
* On ESX modify ```/etc/security/pam_env.conf``` and add following line.
```
COVERAGE_PROCESS_START DEFAULT=/coverage.rc
```
