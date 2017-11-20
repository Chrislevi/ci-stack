### CI/CD Stack Overview ###
> Click on the links for guides
* [**Java**]() - default-jre                        (v1.8-56)
* [**Nexus**](docs/cicd/nexus-toolkit.md) - *Repository Manager*                (v3.3.0-01)
* [**Docker**](docs/cicd/docker-toolkit.md) - *Container Engine*                 (17.05.0-ce)
* [**Gitlab**](docs/cicd/gitlab-toolkit.md) - *Source Code Management*           (v9.3.0)
* [**Juseppe**]() - *Jenkins Plugin Update-center*    (v1.0)
* [**Jenkins**](docs/cicd/jenkins-toolkit.md) - *Continuous Integration Platform* (v2.32)

## Trigger variables ##
* `jenkins_purge: bool`   # jenkins cleanup all
* `nexus_purge: bool`     # nexus cleanup all 
* `juseppe_localdir_plugins: path/to/plugins` # Import plugins from local cache dir
* `docker_archive: bool` # Archive containers to .tar(Must run with internet)
 
### CI/CD Installation ### 
* Edit `inventory/hosts`, and set ansible_host vars with the desired IP (i.e `gitlab ansible_host='10.0.0.1'`)
* Create `/root/.vaultpass.txt` with the vault password inside. (Ask the TEAMLEADER)

> inventory/hosts EXAMPLE
```yamlex
[deploy-servers]
infra ansible_host=localhost ansible_connection=local

# Fill out default server IPs -> ansible_host

# group sections
[nexus-server]
nexus ansible_host='10.0.0.1'

[jenkins-server]
jenkins ansible_host='10.0.0.2'

[gitlab-server]
gitlab ansible_host='10.0.0.1'

[ci-nodes:children]
gitlab-server
nexus-server
jenkins-server
```
* Run '`ansible-plabook playbooks/reqs.yml`'
* Run '`ansible-plabook playbooks/ci.yml`' to install all components


> CLI example
```bash
cd /opt/ci
vim /opt/ci/inventory/hosts
echo "YOU_VAULT_PASS" > /root/.vaultpass.txt
ansible-plabook playbooks/reqs.yml
ansible-plabook playbooks/ci.yml
```
#### Closed-Network CI/CD ###
* Edit inventory/hosts.ci, and set ansible_host vars with the desired IP (SEE example in previous section)
* Place correct binaries/containers inside /prereqs (Archiving done outside and saved to prereqs/*-docker.tar)
* Create `/root/.vaultpass.txt` with the vault password inside. (Ask the TEAMLEADER)
* Define juseppe_localdir_plugins variable to the path containing jenkins Plugins. (Add to 80_ci groupvar)
* Run './deploy.sh init'
* Run './deploy.sh reqs'

> CLI example
```bash
cd /opt/ci
vim inventory/hosts
echo "YOU_VAULT_PASS" > /root/.vaultpass.txt
rsync -avzP /path/to/binaries/* /opt/ci/reqs/
echo -e "\njuseppe_localdir_plugins: 'path/to/jenkins/plugins'" >> /opt/ci/playbooks/group_vars/all/99_general
ansible-plabook playbooks/reqs.yml
ansible-plabook playbooks/ci.yml
```
### Ansible-Vault ###
`98_vault` is an encrypted file which contains sensitive variables such as passwords

* To edit playbooks/group_vars/all/98_vault run `ansible-vault edit playbooks/group_vars/all/98_vault`
* To view playbooks/group_vars/all/98_vault run `ansible-vault view playbooks/group_vars/all/98_vault`

* You can ReEncrypt the file with a different key by running `ansible-vault rekey file1.yml file2.yml`
### CI/CD Management ###

> 10.0.0.1 example for our CI/CD server

 * ***Gitlab administration***  - `http://10.0.0.1:7080`
 * ***Nexus administration***   - `http://10.0.0.1:8081`
 * ***Jenkins administration*** - `http://10.0.0.1:8008`
 * ***Juseppe UpdateCenter***   - `http://10.0.0.1:9090/update-center.json`

### Issues & Pull requests ###
 * Issues:
 *   - Provide a snippet of the relevant logs - ***Mandatory***
 *   - Provide reproduction steps - ***Mandatory***
 *   - Provide code ref if relevant. (e.g Line number/Task name)
 *   - Add a suggested workaround if a known one exists.
 *   - Please state the correct 'Phase' label to each issue accordingly (e.g 'Base' Label) ***Mandatory***

 * Pull Requests:
 *   - Keep the PR name short and relevant
 *   - Point the relevant reviewers
 *   - Keep the discussion in the PR for documentation
 *   - Delete merged branches, Lets keep this git clean and steady. ***Mandatory***

> Issue EXAMPLE:
 ```bash
    Title: Packer initialize step [bug][image] # <---- Assign labels
    Comment: 
               ## Descrition
             - *Bug description*

               ## Log trace
               ```bash
             - *LOG TRACE* # Only the relevant snippet!
               ```

               ## Steps to reproduce
             - Reproduction steps: ./deploy.sh image iso

               ## Possible solution
             - Workaround: Get packer customized iso from different PC  
  ```
### Other sources ###

