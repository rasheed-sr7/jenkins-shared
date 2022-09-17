#!/usr/bin/env groovy
pipeline{
    agent{
        node {
            label "master"
        }
    }
    environment {
        CI = 'false'
    }
    parameters {
	    extendedChoice(defaultValue: 'jdk-8,jdk-11', description: '', multiSelectDelimiter: ',', name: 'jdk_version', quoteValue: false, saveJSONParameterToFile: false, type: 'PT_CHECKBOX', value: 'jdk-8,jdk-11,jdk-17', visibleItemCount: 3 )
        listGitBranches branchFilter: 'refs/heads/(develop.*)', credentialsId: 'GitLab_credentials', defaultValue: 'develop', name: 'BRANCH_NAME', quickFilterEnabled: true, remoteURL: 'https://kosh.customerxps.com/clari5/platform/cc.git', selectedValue: 'NONE', sortMode: 'NONE', tagFilter: '*', type: 'PT_BRANCH'
    }
    stages{
        stage('CleanUp') {
            steps{
                echo "========executing cleanup========"
                script {
                    sh '''
                        rm -rf /home/jenkins/cx/git/cc-develop-x-jdk17/clari5/platform/cc
                        rm -rf /home/jenkins/cx/git/cc-develop-x-jdk11/clari5/platform/cc
                        rm -rf /home/jenkins/cx/git/cc-develop-x-jdk8/clari5/platform/cc
                    '''
                }
            }
        }
        stage("Parallel checkout") {
            parallel {
                stage("cc-develop-x-checkout-jdk17"){
                    when { expression { params.jdk_version.contains('jdk-17') } }
                    tools{
                        jdk "OpenJDK17"
                    }
                    steps{
                        ws('/home/jenkins/cx/git/cc-develop-x-jdk17') {
                            echo "========executing cc-JDK17========"
                            script{
                                try {
                                    sh '''
                                        java -version
                                        set +e
                                        . ${HOME}/.bashrc
                                        . ${HOME}/.bash_profile
                                        export CX_SMG_DEV="/home/jenkins/cx/git/cc-develop-x-jdk17"
                                        smg clari5:platform:cc
                                        smgco ${BRANCH_NAME}
                                        git pull
                                        smg clari5:platform:cc
                                        smgco ${BRANCH_NAME}
                                        git pull origin ${BRANCH_NAME}
                                        smg clari5:platform:cc
                                        git status
                                    '''
                                }
                                catch (err) {
                                    echo "Caught: ${err}"
                                    currentBuild.result = 'FAILURE'
                                }
                            }
                        }
                    }
                }
                stage("cc-develop-x-checkout-jdk11"){
                    when { expression { params.jdk_version.contains('jdk-11') } }
                    tools{
                        jdk "OpenJDK11"
                    }
                    steps{
                        ws('/home/jenkins/cx/git/cc-develop-x-jdk11') {
                            echo "========executing cc-JDK11========"
                            script{
                                try {
                                    sh '''
                                        java -version
                                        set +e
                                        . ${HOME}/.bashrc
                                        . ${HOME}/.bash_profile
                                        export CX_SMG_DEV="/home/jenkins/cx/git/cc-develop-x-jdk11"
                                        smg clari5:platform:cc
                                        smgco ${BRANCH_NAME}
                                        git pull
                                        smg clari5:platform:cc
                                        smgco ${BRANCH_NAME}
                                        git pull origin ${BRANCH_NAME}
                                        smg clari5:platform:cc
                                        git status
                                    '''
                                }
                                catch (err) {
                                    echo "Caught: ${err}"
                                    currentBuild.result = 'FAILURE'
                                }
                            }
                        }
                    }
                }
                stage("cc-develop-x-checkout-JDK8"){
                    when { expression { params.jdk_version.contains('jdk-8') } }
                    tools{
                        jdk "JDK8"
                    }
                    steps{
                        ws('/home/jenkins/cx/git/cc-develop-x-jdk8') {
                            echo "========executing cc-JDK8========"
                            script{
                                try {
                                    sh '''
                                        java -version
                                        set +e
                                        . ${HOME}/.bashrc
                                        . ${HOME}/.bash_profile
                                        export CX_SMG_DEV="/home/jenkins/cx/git/cc-develop-x-jdk8"
                                        smg clari5:platform:cc
                                        smgco ${BRANCH_NAME}
                                        git pull
                                        smg clari5:platform:cc
                                        smgco ${BRANCH_NAME}
                                        git pull origin ${BRANCH_NAME}
                                        smg clari5:platform:cc
                                        git status
                                    '''
                                }
                                catch (err) {
                                    echo "Caught: ${err}"
                                    currentBuild.result = 'FAILURE'
                                }
                            }
                        }
                    }
                }
            }
        }
        stage("Parallel build") {
            parallel {
                stage("cc-develop-jdk17"){
                    when { expression { params.jdk_version.contains('jdk-17') } }
                    tools{
                        jdk "OpenJDK17"
                    }
                    steps{
                        ws('/home/jenkins/cx/git/cc-develop-x-jdk17') {
                            echo "========executing cc-JDK17========"
                            script{
                                try {
                                    withCredentials([usernameColonPassword(credentialsId: 'Build_user', variable: 'USERPASS')]) {
                                        sh '''
                                            java -version
                                            set +e
                                            . ${HOME}/.bashrc
                                            . ${HOME}/.bash_profile
                                            export CX_SMG_DEV="/home/jenkins/cx/git/cc-develop-x-jdk17"
                                            smg clari5:platform:cc
                                            smgco ${BRANCH_NAME}
                                            git pull
                                            smg clari5:platform:cc
                                            smgco ${BRANCH_NAME}
                                            git pull origin ${BRANCH_NAME}
                                            smg clari5:platform:cc
                                            git status
                                            fbuild.sh
                                            creinstaller.sh
                                            CC=$(ls cc-platform*.bash)
                                            curl -v --user "$USERPASS" --upload-file ${CC} https://nexusrepo.customerxps.com/repository/package-images/modules/cc/${CX_SMG_VERSION}/${CC}
                                        '''
                                    }
                                }
                                catch (err) {
                                    echo "Caught: ${err}"
                                    currentBuild.result = 'FAILURE'
                                }
                            }
                        }
                    }
                    post{
                        always{
                            echo "========always========"
                            cleanWs()
                        }
                        success{
                            echo "========cc-JDK17 executed successfully========"
                            slackSend (color: '#00FF00', message: "SUCCESSFUL: Stage ' cc-JDK17 [${env.BUILD_NUMBER}]' (${env.BUILD_URL})", channel:'#cmr-49x-bots', teamDomain: 'cxps-aml', tokenCredentialId: 'cmr-49x-bots')
                        }
                        failure{
                            echo "========cc-JDK17 execution failed========"
                            slackSend (color: '#FF0000', message: "FAILED: Stage ' cc-JDK17 [${env.BUILD_NUMBER}]' (${env.BUILD_URL})", channel:'#cmr-49x-bots', teamDomain: 'cxps-aml', tokenCredentialId: 'cmr-49x-bots')
                        }
                    }
                }
                stage("cc-develop-jdk11"){
                    when { expression { params.jdk_version.contains('jdk-11') } }
                    tools{
                        jdk "OpenJDK11"
                    }
                    steps{
                        ws('/home/jenkins/cx/git/cc-develop-x-jdk11') {
                            echo "========executing cc-JDK11========"
                            script{
                                try {
                                    withCredentials([usernameColonPassword(credentialsId: 'Build_user', variable: 'USERPASS')]) {
                                        sh '''
                                            java -version
                                            set +e
                                            . ${HOME}/.bashrc
                                            . ${HOME}/.bash_profile
                                            export CX_SMG_DEV="/home/jenkins/cx/git/cc-develop-x-jdk11"
                                            smg clari5:platform:cc
                                            smgco ${BRANCH_NAME}
                                            git pull
                                            smg clari5:platform:cc
                                            smgco ${BRANCH_NAME}
                                            git pull origin ${BRANCH_NAME}
                                            smg clari5:platform:cc
                                            git status
                                            fbuild.sh
                                            creinstaller.sh
                                            CC=$(ls cc-platform*.bash)
                                            curl -v --user "$USERPASS" --upload-file ${CC} https://nexusrepo.customerxps.com/repository/package-images/modules/cc/${CX_SMG_VERSION}/${CC}
                                        '''
                                    }
                                }
                                catch (err) {
                                    echo "Caught: ${err}"
                                    currentBuild.result = 'FAILURE'
                                }
                            }
                        }
                    }
                    post{
                        always{
                            echo "========always========"
                            cleanWs()
                        }
                        success{
                            echo "========cc-JDK11 executed successfully========"
                            slackSend (color: '#00FF00', message: "SUCCESSFUL: Stage ' cc-JDK11 [${env.BUILD_NUMBER}]' (${env.BUILD_URL})", channel:'#cmr-49x-bots', teamDomain: 'cxps-aml', tokenCredentialId: 'cmr-49x-bots')
                        }
                        failure{
                            echo "========cc-JDK11 execution failed========"
                            slackSend (color: '#FF0000', message: "FAILED: Stage ' cc-JDK11 [${env.BUILD_NUMBER}]' (${env.BUILD_URL})", channel:'#cmr-49x-bots', teamDomain: 'cxps-aml', tokenCredentialId: 'cmr-49x-bots')
                        }
                    }
                }
                stage("cc-develop-JDK8"){
                    when { expression { params.jdk_version.contains('jdk-8') } }
                    tools{
                        jdk "JDK8"
                    }
                    steps{
                        ws('/home/jenkins/cx/git/cc-develop-x-jdk8') {
                            echo "========executing cc-JDK8========"
                            script{
                                try {
                                    withCredentials([usernameColonPassword(credentialsId: 'Build_user', variable: 'USERPASS')]) {
                                        sh '''
                                            java -version
                                            set +e
                                            . ${HOME}/.bashrc
                                            . ${HOME}/.bash_profile
                                            export CX_SMG_DEV="/home/jenkins/cx/git/cc-develop-x-jdk8"
                                            smg clari5:platform:cc
                                            smgco ${BRANCH_NAME}
                                            git pull
                                            smg clari5:platform:cc
                                            smgco ${BRANCH_NAME}
                                            git pull origin ${BRANCH_NAME}
                                            smg clari5:platform:cc
                                            git status
                                            fbuild.sh
                                            creinstaller.sh
                                            CC=$(ls cc-platform*.bash)
                                            curl -v --user "$USERPASS" --upload-file ${CC} https://nexusrepo.customerxps.com/repository/package-images/modules/cc/${CX_SMG_VERSION}/${CC}
                                        '''
                                    }
                                }
                                catch (err) {
                                    echo "Caught: ${err}"
                                    currentBuild.result = 'FAILURE'
                                }
                            }
                        }
                    }
                    post{
                        always{
                            echo "========always========"
                            cleanWs()
                        }
                        success{
                            echo "========cc-JDK8 executed successfully========"
                            slackSend (color: '#00FF00', message: "SUCCESSFUL: Stage ' cc-JDK8 [${env.BUILD_NUMBER}]' (${env.BUILD_URL})", channel:'#cmr-49x-bots', teamDomain: 'cxps-aml', tokenCredentialId: 'cmr-49x-bots')
                        }
                        failure{
                            echo "========cc-JDK8 execution failed========"
                            slackSend (color: '#FF0000', message: "FAILED: Stage ' cc-JDK8 [${env.BUILD_NUMBER}]' (${env.BUILD_URL})", channel:'#cmr-49x-bots', teamDomain: 'cxps-aml', tokenCredentialId: 'cmr-49x-bots')
                        }
                    }
                }
            }
        }
    }
    post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})", channel:'#cmr-49x-bots', teamDomain: 'cxps-aml', tokenCredentialId: 'cmr-49x-bots')
        }
        failure{
            echo "========pipeline execution failed========"
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})", channel:'#cmr-49x-bots', teamDomain: 'cxps-aml', tokenCredentialId: 'cmr-49x-bots')
        }
    }
}
