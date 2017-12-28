#!groovy
pipeline {
    options {
        //enable timestamps in console output
        timestamps()
    }
    triggers {
        //polling SCM with cron syntax for new changes to trigger a build, default: every minute
        pollSCM('* * * * *')
    }
    // This configures the agent, where the pipeline will be executed. The default should be `none` so
    // that `Approval` stage does not block an executor on the master node.
    agent {
        label "master"
    }
    stages {
        stage('INIT') {
            steps {
                script {
                    //load pipeline configuration
                    config = load 'config/config.jenkins'
                    packageJSON = readJSON file: 'package.json'
                    //!create a long release name for archiving with job name, version, build number
                    // and commit id, e. g. PetClinic_1.3.1_12_e4655456j
                    releaseName = "${packageJSON.name}_${packageJSON.version}_${env.BUILD_NUMBER}_${env.GIT_COMMIT}"
                    //!set Build name with unique identifier with version and build number id, e. g. "1.3.1_12"
                    currentBuild.displayName = "${packageJSON.version}_${env.BUILD_NUMBER}"
                    //install npm dependencies
                    sh "npm install"
                }
            }
        }
        stage('SCAN') {
            steps {
                parallel(
                    SONAR: {
                        script{
                            //!- unit/integration test
                            sh "npm test"
                            //!- Source Code Check with SonarQube
                            //TODO
                        }
                    },
                    SECURITY: {
                        script {
                            //TODO maybe with sonar as well
                            echo "TODO"
                        }
                    },
                    OSLC: {
                        //!- Open Source License Compliance Test
                        echo "TODO"
                    }
                )
            }
        }
        stage('BUILD') {
            steps {
                script {
                    //create docker image
                    echo "TODO"
                    //def customImage = docker.build("my-image:${env.BUILD_ID}")
                    //customImage.push('latest')
                }
            }
        }
        stage('UAT') {
            steps {
                parallel(
                    FUNCTIONAL: {
                        script {
                            //deploy environment for functional tests
                            echo "TODO"
                            //start webdriver.io test
                            //destroy environment for functional tests
                        }
                    },
                    PERFORMANCE: {
                        script {
                            //deploy environment for performance tests
                            //start octoperf test
                            echo "TODO"
                            //destroy environment for performance tests
                        }
                    },
                    EXPLORATIVE: {
                        //deploy environment for explorative tests
                        echo "TODO"
                    }
                )
            }
        }
        stage('APPROVAL') {
            when {
                //only commits to master should be deployed to production (this conditions needs a multi-branch-pipeline)
                branch 'master'
            }
            steps {
                //approval from product owner
                input(message:'Go Live?', ok: 'Fire', submitter: config.approver)
                //destroy explorative environment
            }
        }
        stage('PROD') {
            when {
                //only commits to master should be deployed to production (this conditions needs a multi-branch-pipeline)
                branch 'master'
            }
            steps {
                // This step aborts builds if they reach this step after newer builds
                milestone(ordinal: 0, label: 'PROD')
                script {
                    //lock PROD environment since you can only deploy once at a time
                    lock(resource: "$JOB_NAME-prod_env"){
                        echo "TODO"
                        //Deployment (per node)
                        //- Flightcheck (per node)
                        //- Check Monitoringevents and Logfiles (per node)
                    }
                }
            }
        }
    }
    post {
        failure {
            //!Notify team and abbort Change if needed
            echo "TODO"
        }
        always {
            //!delete workspace
            echo "TODO"
            //node('master') {
                //dir("jobs/${env.JOB_NAME}/${env.BUILD_NUMBER}") {
                    //deleteDir()
                //}
            //}
        }
    }
}