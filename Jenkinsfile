

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
      undeployAppIfPresent()
      cloudFoundryDeploy script:this, deployTool:'mtaDeployPlugin'
  }
}

def undeployAppIfPresent() {
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
                """
            def appExists = (sh(returnStatus: true, script: "cf mta ${app_id}")) == 0
            if (appExists) {
              sh """#!/bin/bash
                  cf undeploy ${app_id} --delete-services -f
                  """
            }
            sh "cf logout"
        }
    }
}
