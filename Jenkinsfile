pipeline {
    agent any
    parameters {
        // Can be overidden in the job
        string(name: 'NODEJS_VERSION', defaultValue: 'Node 20.x', description: 'Version of NodeJS')
        string(name: 'DEPLOY_USER', defaultValue: 'publisher', description: 'User login for the test node')
        string(name: 'DEPLOY_NODE', defaultValue: 'app-staging', description: 'Hostname for the test node')
   }
    stages {
        // Setup the required NodeJS on the agent
        stage('setup') {
            steps {             
                nodejs(nodeJSInstallationName: "${params.NODEJS_VERSION}") {
                    sh 'npm config ls'
                    sh 'npm install'         
                }
            }            
        }
        // Catch typos and recommended best practice.
        stage('static-analysis') {
            steps {
                nodejs(nodeJSInstallationName: "${params.NODEJS_VERSION}") {                       
                    sh 'npm run lint || true'                 
                }
            }
        }
        stage('unit-test') {
            steps {
                nodejs(nodeJSInstallationName: "${params.NODEJS_VERSION}") {
                    sh 'rm -f test-results.xml'
                    sh 'npm run test:unit'
                    sh 'npm run test:coverage'
                }
            }
            post {
                always {
                    junit 'test-results.xml'
                    clover(cloverReportDir: 'coverage', cloverReportFileName: 'clover.xml',
                        // optional, default is: method=70, conditional=80, statement=80
                        healthyTarget: [methodCoverage: 70, conditionalCoverage: 80, statementCoverage: 80],
                        // optional, default is none
                        unhealthyTarget: [methodCoverage: 50, conditionalCoverage: 50, statementCoverage: 50],
                        // optional, default is none
                        failingTarget: [methodCoverage: 0, conditionalCoverage: 0, statementCoverage: 0]
                    )
                }
            }
        }        
        stage('build') {            
            steps {                                             
                nodejs(nodeJSInstallationName: "${params.NODEJS_VERSION}") {   
                    // TODO: Patch the build number
                    sh 'npm run build'                    
                }
            }
            post {
                // Traditional CI will also publish build artifact. This is omitted since the build tags are used
                // for non-compiled based NodeJS based application.
            }
        }        

        stage('deploy') {
            steps {
                withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'jenkins-app-staging', \
                                                             keyFileVariable: 'SSH_KEY_FOR_APP_STAGING')]) {                                                    
                  sh "ssh-keyscan ${params.DEPLOY_NODE} -p 22 > ~/.ssh/known_hosts"
                  sh "scp -i ${SSH_KEY_FOR_APP_STAGING} -r dist/* ${params.DEPLOY_USER}@${params.DEPLOY_NODE}:/public"
                }                  
            }
        }
    }
}
