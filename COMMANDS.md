## Create

Create does exactly what you think it does - it helps you spin up containers. Because this command is so feature-rich, please check out [advanced create](ADVANCED.md#create) once you are familiar with the basics.

For simple standalone Splunk environments, you can run the following command:

```
# All of these commands will create a single standalone Splunk instance
$ orca create
$ orca create --standalone 1
$ orca create --so 1
```

Note that the arguments `--standalone` and `--so` do the same thing. These will be referred to as the long-form \(`--standalone`\) and short-form \(`--so`\) of arguments.

For distributed Splunk environments, you can run the following command:

```
# Both these commands will create a distributed environment with 3 search heads and 3 indexers
$ orca create --search-heads 3 --indexers 3
$ orca create --sh 3 --idx 3
```

As you create the distributed environment, you may notice that you'll get more than the 6 containers expected. This is because, by default, ORCA will automatically cluster roles as it sees fit. If you specify more than 3 search heads, ORCA will create a search head cluster with an additional container that acts as a deployer. If you specify more than 3 indexers, ORCA will create an indexer cluster with an additional container that acts as a cluster master.

As ORCA runs, it will output the real-time Ansible output for every role it finds while ending in the final deployment access information \(stack name, container name, SplunkWeb/SplunkD URLs, etc.\). For scripting, you may not need all of this information and instead only require the deployment topology. ORCA provides the means to redirect the Ansible output if necessary, as well as format the output in a JSON blob. See the examples below:

```
# This will write the Ansible output to a file
$ orca --printer json --ansible-log ./ansible_output.txt create

# This will write the Ansible output to /dev/null using the short-form argument
$ orca --printer json -al /dev/null create
```

---

## Destroy

The destroy command lets you remove deployments from UCP. This should be done when you are finished with your environment and no longer need it - note that because UCP is shared infrastructure for all of products, deployments are automatically destroyed **every 24 hours**. If you need a deployment to be up for longer, let the ORCA team know.

Destroy accepts a single stack name to remove. For example, with a stack name `nwang170828231141aluly`, you can delete by running:

```
$ orca destroy nwang170828231141aluly
```

The next simplest form of destroy is removing all deployments created with your username. This can be done with:

```
$ orca destroy --all
```

Additionally, destroy can be combined with the --custom-label flag to further determine what deployments or parts of deployments need to be destroyed. The custom label is taken in combination with the username, so the different users can use the same label in their stacks without affecting each other. The following will destroy all containers with the label "job-build-url"

```
$ orca destroy --custom-label job-build-url --all
```

##### Optional Arguments

| Long-form | Description |
| :---: | :---: |
| --custom-label | Destroy all containers that match a custom label, provided during create-time |
| --container-id | Destroy container by a specific container ID |
| --no-warn | Do not prompt a warning if destroying another user's stacks |
| --all | Removes all containers created by you |

---

## Exec

The exec command allows users to run a command within a container. For example, with a stack name `nwang170828231141aluly` and a container name `nwang170828231141aluly_so1_1`, you can launch a bash shell inside the container \(essentially SSH\) using:

```
$ orca exec nwang170828231141aluly /bin/bash
$ orca exec nwang170828231141aluly_so1_1 bash
```

If you need to provide arguments to the command to be run in the container, then use quotes around the parameters as such:

```
$ orca exec nwang170828231141aluly_so1_1 "echo hi"
hi
```

##### Optional Arguments

| Long-form | Short-form | Description |
| :---: | :---: | :---: |
| --interactive | -it, --it | Specify interactive, TTY-enabled exec session |

---

## Provision

The provision command configures the Splunk environment with a given deployment ID using Ansible. You can run any custom Ansible playbooks on your stack using the provision command with the --playbooks option like:

```
$ orca provision STACK_NAME --playbooks play1.yml,play2.yml
```

For instance, with a stack name `nwang170828231141aluly` and a custom Ansible playbook `custom_play.yml` \(contents listed below\) in the current working directory, I can use ORCA to run the playbook over the given stack as seen below:

```
$ cat custom_play.yml
---
- hosts: orca_role_standalone
  tasks:

      - name: Ping the server
        ping:
```

```
$ orca provision nwang170828231141aluly --playbooks custom_play.yml

PLAY [orca_role_standalone] ***************************************************

TASK [Gathering Facts] ********************************************************
Monday 28 August 2017  23:22:04 +0000 (0:00:00.031)       0:00:00.031 ********* 
ok: [nwang170828231141aluly_so1_1]

TASK [Ping the server] ********************************************************
Monday 28 August 2017  23:22:05 +0000 (0:00:00.962)       0:00:00.993 ********* 
ok: [nwang170828231141aluly_so1_1]

PLAY RECAP ********************************************************************
nwang170828231141aluly_so1_1 : ok=2    changed=0    unreachable=0    failed=0   

Monday 28 August 2017  23:22:06 +0000 (0:00:00.332)       0:00:01.326 ********* 
=============================================================================== 
Gathering Facts --------------------------------------------------------- 0.96s
Ping the server --------------------------------------------------------- 0.33s
```

##### Optional Arguments

| Long-form | Short-form | Scenario-form | Description |
| :---: | :---: | :---: | :---: |
| --playbooks | --aps | playbooks = play1.yml | Define custom ansible playbooks to use \(run in order\) |
| --scenario | --sc | Stanza name in orca.conf | Use one or more scenarios with playbooks and ansible parameters defined |

---

## Config

The config command is used to authenticate your user with UCP. Config should be run on your first-time use, anytime you want to pull the latest NFR license, or when upgrading to a new release of ORCA. In general, most issues you see can probably be resolve by running config again and filling out the prompt:

```
$ orca config
```

##### Optional Arguments

| Long-form | Scenario-form | Description |
| :---: | :---: | :---: |
| --username | username | AD username |
| --password | N/A | AD password |
| --organization | organization | UCP team/organization \(not used\) |
| --ucp-host | docker\_host | UCP host \(ucp.splunk.com\) |

---

## Show

The show command gives you information about all your current deployments, containers, scenarios, and more. This can be done by any of the following commands:

```
$ orca show deployments
$ orca show containers
$ orca show config
$ orca show scenarios
$ orca show ansible-hosts
```

You can also filter based on various options. For instance, with a stack name `nwang170828231141aluly` I can show only the containers that are part of that stack with:

```
$ orca show containers --deployment-id nwang170828231141aluly
===============================================================================
==== Ansible Plays:  (Playbooks were called in the following order)
===============================================================================
-------------------------------------------------------------------------------
===== Target Playbook: /usr/lib/python2.7/site-packages/splunk_orca/ansible/site.yml 
===== Target Playbook Variables:  async_remote=False cloud_provisioning=None non_async=True async_local=False cloud_mode=False platform=linux splunk_license_master=False splunk_idx_cluster=False cloud=docker splunk_sh_cluster=False 
-------------------------------------------------------------------------------
===============================================================================

===============================================================================
====  ORCA               || Splunk Orchestration and Automation
===============================================================================
====  User                          :: nwang
====  Deployment id                 :: nwang170828231141aluly
====  Splunk Instance (running)     :: Container Name         nwang170828231141aluly_so1_1
====                                :: Platform               x64_debian_8
====                                :: Cloud Provider         docker
====                                :: Splunkweb              http://10.141.65.16:45746 (default: admin/changed)
====                                :: Splunkd                https://10.141.65.16:45744
===============================================================================
```

##### Optional Arguments

| Long-form | Description |
| :---: | :---: |
| --filter | Any filter that can be applied to the returned parameters |
| --username | Username to inspect somebody else's container |
| --deployment-id | Specify which deployment-id the user wishes to see details on this stack |
| --custom-label | Specify which custom label user want to filter on |
| --captain | Display the Search Head captain of the stack |
| --cluster-master | Display the Cluster Master of the stack |

---

## Copy

The copy command provides the ability to copy files from your current working directory into a specific container, or vice versa. See the following example for how this works with a simple file `sample.txt`:

```
$ orca copy --source sample.txt --destination /root/ to nwang170828231141aluly_so1_1
$ orca copy --source ./path/to/sample.txt --destination /root/ to nwang170828231141aluly_so1_1

$ orca copy --source /root/sample.txt destination ./new.txt from nwang170828231141aluly_so1_1
```

You can also copy directories. For example, take a directory named `src` with contents `sample.txt` in the following command:

```
$ ls src
sample.txt
$ orca copy --source src --destination /opt/ to nwang170828231141aluly_so1_1
$ orca exec nwang170828231141aluly_so1_1 bash
root@so1:/opt/splunk# cd /opt/src
root@so1:/opt/src# ls
sample.txt
```

**NOTE:** If you're using ORCA [as a container](SETUP.md#container-installation), absolute paths will not work as the source of your "copy to", or the destination of your "copy from". Although your CLI command is passed into a container, files you reference via absolute path on your host machine will have different paths when referenced inside of the container. However, if you use ORCA [as a Python package](SETUP.md#pypi-installation), absolute paths won't be a problem.

##### Optional Arguments

| Long-form | Description |
| :---: | :---: |
| --source | Define the source location |
| --destination | Define the destination location |

---

## Start

The start command allows you to start a stopped container. This is mainly built to simulate bringing a node back after some downtime. Given a stack name `nwang170828231141aluly` with container name `nwang170828231141aluly_so1_1`, the start command can be used as such:

```
# This will start ALL containers in the deployment
$ orca start nwang170828231141aluly

# This will start a single container by name
$ orca start nwang170828231141aluly_so1_1
```

---

## Stop

The stop command allows you to stop a running container. This is mainly built to simulate bringing a node down. Given a stack name `nwang170828231141aluly` with container name `nwang170828231141aluly_so1_1`, the stop command can be used as such:

```
# This will stop ALL containers in the deployment
$ orca stop nwang170828231141aluly

# This will stop a single container by name
$ orca stop nwang170828231141aluly_so1_1
```

---

## Upgrade

The upgrade command is used to upgrade Splunk in your deployment. For instance, if you're running any migration testing from 6.3.0 to 6.4.0, you can first create a stack with 6.3.0 then upgrade to 6.4.0.

Upgrade can be performed by passing either Splunk build-hash, Splunk version, Splunk branch \(from [http://releases.sv.splunk.com](http://releases.sv.splunk.com)\), or even using a local build from your current working directory.

```
$ orca upgrade nwang170828231141aluly --splunk-build=f2c836328108
$ orca upgrade nwang170828231141aluly --splunk-version=6.4.0
$ orca upgrade nwang170828231141aluly --splunk-branch=current
$ orca upgrade nwang170828231141aluly --local-build ./splunk.tgz
```

Upgrade also allows you to update the license using a license in your current working directory across your Splunk cluster with a simple argument:

```
$ orca upgrade nwang170828231141aluly --splunk-local-license new.lic
```

##### Optional Arguments

| Long-form | Description |
| :---: | :---: |
| --splunk-build | Specify which build \(defined by Git hash or P4changenum\) Splunk should be upgraded to |
| --splunk-version | Specify which version Splunk will be upgraded to |
| --splunk-branch | Specify which branch Splunk will be upgraded to |
| --local-build | Specify the local Splunk build to be utilized |
| --splunk-local-license | Specify a particular Splunk license to use |

---

## Build

Build is a powerful command to assist you in creating your own Splunk Docker image that is compatible with ORCA. You can use this command to build your own Splunk image and push it to Artifactory, making it quicker for others to use your image for testing and validation. While the same effect can be achieved with the --dynamic flag during create, build will remove some overhead in provisioning time during start-up. Be aware that this does come at the cost of waiting for the image build to finish and push, so you should use this only when you expect or require repeated access to the same Splunk image.

```
$ orca build splunk_container --splunk-branch current
```

Build can also be used to publish pypi modules, orca images, or dynamic Splunk images as such:

```
$ orca build pypi
$ orca build orca_container
$ orca build dynamic_splunk_container
```

##### Optional Arguments

| Long-form | Description |
| :---: | :---: |
| --splunk-build | Specify which build \(defined by Git hash or P4changenum\) Splunk should be upgraded to |
| --splunk-version | Specify which version Splunk will be upgraded to |
| --splunk-branch | Specify which branch Splunk will be upgraded to |
| --local-build | Specify the local Splunk build to be utilized |
| --splunk-ansible-version | Specify which Ansible version you want as part of the images |
| --platform | Specify image platform to use as the basis of the Splunk image |
| --full-build | Build a splunk container from scratch \(this takes a multiple minutes\) |
| --splunk-image-version | Specify which image version to be built. If not set, it will use splunk\_image\_version from orca.conf |
| --build-latest | Build the image tagged as latest. This should only be used by ORCA development or RE. |
| --repo-username | Upload repo username.  IF you are already authenticated to the repo you probably wont need this |
| --repo-password | Upload repo password. |
| --skip-dynamic | If building a platform, skip building the DYNAMIC container. |
| --launch-code | See below |

---

##### Launch Code

Building latest images and pushing to the repo should be only done by ORCA dev or RE. To enable this operation, you must provide the --launch-code with this argument. If you think you need to do this, please consult with the ORCA dev team. However, you can build latest to your local docker repo without it.

## Test

ORCA test lets you run test suites and collect test results. It provides two mechanisms for collecting tests to run: gather tests on the fly using Rubikscube \(a test collection tool\), or explicitly accepting Rubikscube JSON \(file or string\) as argument.ƒ

Here is an example of running tests collected on the fly from a specified test suite.

```
#Run all tests under myapp/mytest/testsuite1 in the current working directory.
orca test run --source myapp --test-path mytest/testsuite1
```

When you execute the command above, ORCA will create a new stack that includes a test runner container to run tests, similar to running the orca create command. After all test execution is complete and test results collected, ORCA will automatically destroy the stack.

To run your tests against an existing stack already created by ORCA \(orca create\) instead of creating an ephemeral stack, use the `deployment-id` parameter. ORCA will create the test runner containers and connect them to the deployment network if needed. After testing is complete, ORCA will deploy the test runner containers but the stack will remain intact.

**NOTE:** The existing stack must be a cluster of distributed containers. Single-instance deployment is not supported since ORCA needs to distribute test runs across multiple containers.

```
#Run all tests under myapp/mytest/testsuite1 against a stack
orca test run --source myapp --test-path mytest/testsuite1 --deployment-id johndoe1801240536446tiw4
```

If you want to collect test code by having Rubikscube filter tests using test markers, pass the `testcube-marker` argument, for example:

```
orca test run --source myapp --test-path mytest/testsuite1 --deployment-id johndoe1801240536446tiw4 --testcube-marker "-m \"not docker_issue\""
```

To pass arguments in JSON format into the test runner container to run the tests, use the `runner-args` parameter, for example:

```
orca test run --source myapp --test-path mytest/testsuite1  --runner-args "py.test"
```

Running tests by explicitly specifying Rubikscube json takes two steps.

You first use the `gather` positional argument to collect test code only, either using test markers, or not.

```
#Not using testcube-marker
orca test gather --source johndoe/myapp --test-path mytest/testsuite1 

#Using testcube-marker
orca test gather --source myapp --test-path mytest/testsuite1 --testcube-marker "-m \"not docker_issue\""
```

After the Rubikscube json file is generated, you can pass the json file or json string to orca test to execute the collected tests, for example:

```
#Passing the Rubikscube json file 
orca test run --source myapp --test-path mytest/testsuite1 --rubiks-json myapp/test/ci_jobs.json
#Passing the Rubikscube json string 
orca test run --source myapp --test-path mytest/testsuite1 --json '[{"test_count": 1,"aggregate_report_name":"test1","pytest_args":"","TEST_TYPE":"testcube","test_path":"test1","NUM_OF_VM": 1},{"test_count": 1,"aggregate_report_name": "test2","pytest_args": "","TEST_TYPE": "testcube","test_path": "test2"}]'
```

##### Usage

Collect tests

```
orca test gather --source <relative_path_to_source_code> --test-path <relative_path_to_test_suite> 
[--testcube-marker <testcube_markers_json_file>]
```

Run tests

```
orca test run (--source <relative_path_to_source_code> --test-path <relative_path_to_test_suite> 
[--testcube-marker <testcube_markers_json_file>] | 
(--rubiks-json <rubikscube_json_file> | --json <Rubiskcube_json_string>))
[--deployment-id <orca_stack_id>] [--runner-args <runner_args_json>] 
[--requirement-folder <requirements_file_folder>] [--requirement-file <requirement_filename>]
```

##### Arguments

| Long-form | Scenario-form | Description |
| :---: | :---: | :---: |
| run |  | Use this argument to run tests. |
| gather |  | Use this argument to collect tests to run and generate a Rubikscube json file. |
| --deployment-id | deployment\_id | Optionally specify an existing stack to use for running tests. If not specified, ORCA will create a new stack with a default test runner container dedicated to running tests. If the existing stack does not include any test runner container, a default test runner container will be added to the stack \(and removed when complete\). This is argument can only be used for running a single test suite. |
| --runner-args | runner\_args | Optional arguments in JSON format to pass into the test runner container to run the tests |
| --source | source | Provide the local path to the top-level of the source code to be tested, either provided in .tgz format or a sub-directory under the current working directory. Defaults to the current local directory if not specified. |
| --test-path | test\_path | Provide the path to the directory from which tests will be executed, relative to the directory specified in the source parameter. ORCA test concatenates the source and test-path paths to get the full path from which to run tests in the test runner container. If runner\_args is provided, test-path must also be provided. |
| --rubiks-json | rubikscube | JSON file containing collected tests produced by Rubikscube. Rubikscube cannot be used in conjunction with any other commands. If you pass this argument, testcube-marker will be ignored. |
| --json | json | JSON string containing collected tests produced by Rubikscube. Rubikscube cannot be used in conjunction with any other commands. If you pass this argument, testcube-marker will be ignored. |
| --testcube-marker | testcube\_marker | Optionally specify a JSON file containing test markers to be passed to Rubikscube to collect tests and generate json. |
| --requirement-folder | requirement\_folder | Optionally specify the folder where the requirement file is located. If not specified, orca test looks for the requirements file in the current working directory. |
| --requirement-file | requirement\_file | Optionally specify the name of the requirements file. The default is requirements.txt. |

---

## Help

Help shows additional information on commands and arguments directly in your shell. For example, to get help on the "create" command, you can run:

```
# To see any optional arguments or available commands...
orca [--help/-h]
orca help

# To see specific arguments for a given command...
orca create [--help/-h]
orca help create
```



