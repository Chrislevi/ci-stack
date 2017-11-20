# **Jenkins Toolkit** #

## Github Pull Request Plugin ##

**To configure a Github integrated job follow the guide below.**

* Official Documentation Link: https://github.com/jenkinsci/ghprb-plugin/blob/master/README.md
***

## Gitlab Merge Request Plugin

**To configure a Gitlab integrated job follow the guide below.**

* Official Documentation Link: https://medium.com/@teeks99/continuous-integration-with-jenkins-and-gitlab-fa770c62e88a
***

## Declarative Pipelines ##

* Official Documentation Link: https://jenkins.io/doc/

* This is the full syntax for Jenkins Declarative Pipelines as of version 0.8.1.
* Declarative Pipeline is implemented within a Jenkinsfile inside a repository

```python
pipeline {
  // Possible agent configurations - you must have one and only one at the top level.
  agent any
  agent none
  agent {
    label "whatever"
  }
  // Note that you can define a default label for docker and dockerfile in either your Jenkins
  // global configuration, or in the configuration for a folder, like a GitHub Organization Folder.
  // If that's defined, that label will be used when you don't specify one here explicitly.
  agent {
    docker "ubuntu:16.04"
  }
  agent {
    dockerfile true
  }
  agent {
    docker {
      image "ubuntu:16.04"
      label "docker-nodes"
      args "-v /tmp:/tmp"
    }
  }
  agent {
    dockerfile {
      filename "someOtherDockerfile"
      label "docker-nodes"
      args "-v /tmp:/tmp"
    }
  }

  // Environment
  environment {
    FOO = "bar"
    OTHER = "${FOO}baz"
    SOME_CREDENTIALS = credentials('some-id')
  }

  // Tools - only works when *not* on docker or dockerfile agent
  tools {
    // Symbol for tool type and then name of configured tool installation
    maven "maven3.3.9"
    jdk "jdk8"
  }

  options {
    // General Jenkins job properties
    buildDiscarder(logRotator(numToKeepStr:'1'))
    // Declarative-specific options
    skipDefaultCheckout()
    // "wrapper" steps that should wrap the entire build execution
    timestamps()
    timeout(time: 5, unit: 'MINUTES')
    // Plugin specific options
    gitLabConnection('Gitlab-Jenkins')
    gitlabCommitStatus(name: 'Jenkins-CI')
  }

  triggers {
    cron('@daily')
  }

  // Access parameters with 'params.PARAM_NAME' - that'll get you default values too.
  parameters {
    booleanParam(defaultValue: true, description: '', name: 'flag')
    // Newer core versions support "stringParam" as well
    string(defaultValue: '/opt/', description: '', name: 'work_dir')
  }

  stages {
    stage("first stage") {
      // All sections within stage other than steps are optional.
      environment {
        // Overrides or adds to the existing environment
        FOO = "notBar"
      }
      tools {
        // Overrides tools of the same type defined globally
        maven "maven3.3.3"
      }
      agent {
        // Overrides the top-level agent. "agent none" at the stage level does nothing.
        label "some-other-label"
        customWorkspace "${params.work_dir}"
      }

      // Conditional execution of this stage - only run this stage if the when condition is true.
      when {
        // One and only one condition is allowed.

        // Only run if the branch matches this Ant-style pattern
        branch "master"

        // Only run if the environment contains this given variable name with this given value
        environment name: "FOO", value: "notBar"

        // Only run if this Scripted Pipeline expression doesn't return false or null
        expression {
          echo "You can run any Pipeline steps in here"
          return "foo" == "bar"
        }
      }

      // Runs at the end of the stage, depending on whether the conditions are met.
      post {
        // always means, well, always run.
        always {
          echo "Hi there"
        }
        // changed means when the build status is different than the previous build's status.
        changed {
          echo "I'm different"
        }
        // success, failure, unstable all run if the current build status is successful, failed, or unstable, respectively
        success {
          echo "I succeeded"
          archive "**/*"
        }
      }

      // steps is required and is where you put your stage's actual work
      steps {
        echo "I'm doing things, I guess."
        retry(5) {
          echo "Keep trying this if it fails up to 5 times"
        }

        // the script block allows you to run any arbitrary Pipeline script, even if it doesn't fit the Declarative subset.
        script {
          if ("sky" == "blue") {
            echo "You can't actually do loops or if statements etc in Declarative unless you're in a script block!"
          }
        }
      }
    }

    stage("second stage") {
      steps {
        // You can only use the parallel step if it's the *only* step in the stage.
        parallel(
          firstBlock: {
            echo "I'm on one parallel block"
          },
          secondBlock: {
            echo "I'm on the other block"
          }
        )
      }
    }
  }

  post {
    // always means, well, always run.
    always {
      echo "Hi there"
    }
    // changed means when the build status is different than the previous build's status.
    changed {
      echo "I'm different"
    }
    // success, failure, unstable all run if the current build status is successful, failed, or unstable, respectively
    success {
      echo "I succeeded"
      archive "**/*"
    }
  }
}
```
***

# Jenkins Role customization #

### Adding credentials ###

 #### @ defaults/main.yml

* Configure those variables to edit/add more credentials
  * `cj_sshkey_credentials` # Credentials with ssh key injection
  * `cj_credentials` # User password credential

```snakeyaml
cj_sshkey_credentials:
  - {name: "Jenkins", id: "local-ssh-user", username: "root", password: "qwe123", description: "Root SSH key credentials", private_key: "{{ cj_home }}/.ssh/id_rsa"}

cj_credentials:
  - { name: "Github API user", id: "Github-user", username: "JenkinsBot", password: "Aa123456", description: "Github Creds"}
  - { name: "Gitlab API user", id: "Gitlab-user", username: "{{ gitlab_user.username }}", password: "{{ gitlab_pass.password }}", description: "Gitlab creds"}
```

### Adding slaves ###

#### @ defaults/main.yml

* Just add another dictionary key with the according key-value structure:
  * `name` - name of the slave
  * `host` - hostname/ip for connection purposes
  * `launcher_type` - ssh,windows,javaWeb
  * `credential_id` - set the credential_id just like in above section.
  * `home` - OS home endpoint
  * `labels` - labels are name tags which are used within pipelines/jobs to point at which slave to execute the job
  * `executors` - number of executor proccess each slave has, executor per build.

```snakeyaml
cj_nodes_create: true
cj_nodes:
  - { name: "localroot", host: "localhost", launcher_type: "ssh", ssh_port: "22", credential_id: "local-ssh-user", home: "/home/jenkins", labels: "localroot", executors: "3", description: "localroot" }
```

### Plugins ###

 @ ***group_vars/all/81_plugins*** contains the plugin list.

To recreate the list with different plugins, run this snippet after you installed Jenkins with an internet connection

***Note:*** Juseppe is configured to be Jenkins update-center OTB. minor changes is required to install jenkins with Jenkins.io update-center
which holds the world's plugins repository. to enable Jenkins.io update-center set `cj_nexus` variable to false.

```bash
java -jar /var/cache/jenkins/war/WEB-INF/jenkins-cli.jar -s http://localhost:8008 list-plugins \
| awk '{print $1, $NF;}' \
| awk -F " " ' {print " - { name: \""$1 "\","" version: \""  $2 "\", enabled: \"true\" }"} '
```

* Verify the output and replace `81_plugins` with the new set of plugins.

* To Import plugins instead of downloading (Closed network) run:
```bash
ansible-playbook playbooks/juseppe.yml -i inventory/hosts.ci -e "juseppe_localdir_plugins=/path/to/plugins/dir"
```
  Or add the `-e juseppe_localdir_plugins=/path/to/plugins/dir` to the function of deploy_ci @ bin/functions

### Template Pipeline/Jobs ###
* To add more templates:
  * Create a job/pipeline @ http://OurJenkinsIP:8008
  * Configure the job/pipeline to your satisfaction
  * copy the JENKINS_HOME/jobs/JOB_NAME/config.xml to Jenkins template dir @ roles/jenkins/templates/JOB_NAME.xml.j2
  * Edit the JOB_NAME.xml.j2 to be dynamic if necessary
  * Add the JOB_NAME to the `seed_job_list` variable and there you go.

> For example.
```snakeyaml
seed_job_list:
  - JOB_NAME
```

***Note:*** Currently the job trigger mechanism is disabled, to enable just uncomment the remarked lines @ jenkins/tasks/cj-seed.yml
```yamlex
# Trigger after deploy
#- name: "Trigger seed job"
#  shell: >
#    java -jar {{ cj_cli_jar_location }} \
#    -s http://{{ cj_jenkins_hostname }}:{{ cj_http_port }}{{ cj_application_context | default('') }}/ \
#    build -s {{ item }}
#  with_items: "{{ seed_job_list }}"
```