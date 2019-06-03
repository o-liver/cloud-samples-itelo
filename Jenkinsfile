

node(){
  stage('Prepare')   {
      deleteDir()
      checkout scm
      setupCommonPipelineEnvironment script:this
  }

  stage('Build')   {
      mtaBuild script:this
  }

  stage('Deploy')   {
      cleanupPreviousDeployment()
      cloudFoundryDeploy script:this, deployTool:'mtaDeployPlugin'
  }
}

def cleanupPreviousDeployment() {
    try {
        def app_id = (readYaml (file: 'mta.yaml')).ID
        dockerExecute(script: this, dockerImage: "s4sdk/docker-cf-cli") {
            def config = commonPipelineEnvironment.configuration.steps.cloudFoundryDeploy
            withCredentials([usernamePassword(
                credentialsId: config.cloudFoundry.credentialsId,
                passwordVariable: 'password',
                usernameVariable: 'username'
            )]) {
                sh """#!/bin/bash
                    set +x
                    set -e
                    cf api ${config.cloudFoundry.apiEndpoint}
                    cf login -u ${username} -p '${password}' -a ${config.cloudFoundry.apiEndpoint} -o \"${config.cloudFoundry.org}\" -s \"${config.cloudFoundry.space}\"
                    cf install-plugin multiapps -f
                    cf plugins
                    cf undeploy ${app_id} --delete-services -f""" //TODO: retrieve ID from mta.yaml
                sh "cf logout"
            }
        }
    } catch (Exception e) {
      echo "Exception handled ; )"
    }
}
