pipeline {
    agent any

    parameters {
        string(name: 'TF_WORKSPACE', defaultValue: 'dev', description: 'Workspace file to use for deployment')
        string(name: 'version', defaultValue: '0.13.3', description: 'Version variable to pass to Terraform')
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }
    
    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        TF_IN_AUTOMATION      = '1'
    }

    stages {
        stage('BuildInfo') {
            steps {
                    echo "Running Buid num: ${env.BUILD_ID} on Jenkins ${env.JENKINS_URL}"
                    echo "BUILD_NUMBER :: ${env.BUILD_NUMBER}"
                    echo "BUILD_ID :: ${env.BUILD_ID}"
                    echo "BUILD_DISPLAY_NAME :: ${env.BUILD_DISPLAY_NAME}"
                    echo "JOB_NAME :: ${env.JOB_NAME}"
                    echo "JOB_BASE_NAME :: ${env.JOB_BASE_NAME}"
                    echo "BUILD_TAG :: ${env.BUILD_TAG}"
                    echo "EXECUTOR_NUMBER :: ${env.EXECUTOR_NUMBER}"
                    echo "NODE_NAME :: ${env.NODE_NAME}"
                    echo "NODE_LABELS :: ${env.NODE_LABELS}"
                    echo "WORKSPACE :: ${env.WORKSPACE}"
                    echo "JENKINS_HOME :: ${env.JENKINS_HOME}"
                    echo "JENKINS_URL :: ${env.JENKINS_URL}"
                    echo "BUILD_URL ::${env.BUILD_URL}"
                    echo "JOB_URL :: ${env.JOB_URL}"
    
                }
            }
        stage('Plan') {
            steps {
                script {
                    currentBuild.displayName = params.version
                }
                //dir("${env.WORKSPACE}/Terraform-with-Jenkins"){  //directory steps paramters to change the directory(if you have terraform in a dir in git)
                  //https://www.jenkins.io/doc/pipeline/steps/workflow-basic-steps/#dir-change-current-directory
                  //In normal git configuration we do not require to change the directory
                  sh 'pwd'
                  sh 'terraform init -input=false'
                  //sh 'terraform workspace select ${TF_WORKSPACE}'
                  sh 'terraform workspace new $TF_WORKSPACE || true'
                  sh "terraform plan -input=false -out tfplan --version=${params.version} --var-file=${params.TF_WORKSPACE}.tfvars"
                  sh 'terraform show -no-color tfplan > tfplan.txt'
                //}
            }
        }

        stage('Approval') {
            when {
                not {
                    equals expected: true, actual: params.autoApprove
                }
            }

            steps {
                script {
                    def plan = readFile 'tfplan.txt'
                    input message: "Do you want to apply the plan?",
                        parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                }
            }
        }

        stage('Apply') {
            steps {
                
                sh "terraform apply -input=false tfplan"
               
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'tfplan.txt'
        }
    }
}


// This Jenkinsfile depends on a couple of parameters:

// TF_WORKSPACE - string, specifies the Terraform workspace to use
// version - string, passed to terraform plan (you might want to remove/add to/swap this for other variables)
// autoApprove - boolean, if true skips the approval process immediately runs terraform apply
// You'll probably want to change the TF_WORKSPACE variables and the vars passed into terraform plan