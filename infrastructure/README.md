### provisioning
Ansible is used as a provisioning tool for the host running as controller. The playbook for that is located in `playbooks/controller.yaml`. The playbook targets the hosts under `controllers` in the `ìnventory/inventory.yaml`. To access the host it uses the ssh key referenced in `ìnventory/group_vars/controller` named `ansible_ssh_private_key_file`. The playbook also requires you to provide a password for the grafana (action) user.

#### prerequisites
To install the required packages for your OS:
```
[python3(https://realpython.com/installing-python/)]
[passlib](https://passlib.readthedocs.io/en/stable/install.html)
ansible roles:
* [geerlingguy.nodejs](https://github.com/geerlingguy/ansible-role-nodejs)
* [nickjj.docker](https://github.com/nickjj/ansible-docker)
```

A typical way to run the playbook would be:
```
ansible-playbook -i infrastructure/inventory/inventory.yaml infrastructure/playbooks/controller.yaml --extra-vars "action_user_pw=test"
```

### Architecture

The diagram below describes the production setup.

![Production diagram](prod-infrastructure.png)

Production is comprised of two hosts:
* The `controller` is a Virtual Machine with the `runner`, `datastore` and `dashboard`
* The `minion` is a Bare metal machine to run the benchmarks.

### controller
The controller runs the [runner](../runner/) as a daemon in a docker container and exposes an endpoint to be able to trigger a benchmark run with parameters.
Also the datastore, `InfluxDB`, and the dashboard, `Grafana`, are run inside containers, all using [docker-compose](https://docs.docker.com/compose/).

The runner can be triggered from a remote resource but will, at least, be triggered once a day on a schedule to ensure there is a data point for each day.

The runner is responsible for running the latest benchmark tests on the `minion`. [Ansible](https://www.ansible.com/) is used to manage the provisioning state of the `minion` before each run. The benchmarks write the test results in a file determined by a parameter. After the run, this file is retrieved and parsed by the runner. The results are then written to the datastore as well as stored on the IPFS network.

### minion
This `bare metal` machine will be kept as clean as possible to enable reliable results from the benchmarks. Tests are run over `ssh` from the `controller`.

After the benchmark test is run to get an accurate result the same test will be run using clinic tools to provide in depth analysis.

Tests kept up to date before each run and are always run in sequence. After each test, any generated files and resources will be retrieved by the runner and cleaned up after successful reception..
