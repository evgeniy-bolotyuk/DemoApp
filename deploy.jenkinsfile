
pipeline {
    agent none

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        GIT_REPO = 'https://github.com/evgeniy-bolotyuk/DemoApp.git'
        GIT_BRANCH = 'main'
        
        SOURCE_DIR = 'sources'
        CF_NAME = 'DemoApp.cf'
        V8_VERSION = '8.3.27'
        
        INFOBASE_PATH = 'D:\\Databases\\DemoApp'
        IB_USER = 'Администратор'
    }

    stages {
        stage('Build Configuration') {
            agent {
                label '1c-build'
            }

            stages {
                stage('Clean Build Workspace') {
                    steps {
                        echo 'Cleaning build workspace...'
                        deleteDir()
                    }
                }

                stage('Checkout GitHub') {
                    steps {
                        dir("${env.SOURCE_DIR}") {
                            git branch: "${env.GIT_BRANCH}",
                                url: "${env.GIT_REPO}",
                                credentialsId: 'github-demoapp-token'
                        }
                    }
                }

                stage('Compile configuration file') {
                    steps {
                        sh '''
                            set -eu

                            echo "=== Check Vanessa Runner ==="
                            command -v vrunner

                            echo "=== Check Sources ==="
                            SRC_PATH="$WORKSPACE/$SOURCE_DIR/src/DemoApp"
                            test -d "$SRC_PATH"

                            echo "=== Compile Configuration ==="

                            vrunner cf compile \
                                --src "$SRC_PATH" \
                                --v8version "$V8_VERSION" \
                                --src-format edt \
                                --ibcmd \
                                "$WORKSPACE/$CF_NAME"

                            echo "=== Check CF ==="
                            test -s "$WORKSPACE/$CF_NAME"

                            echo "=== Result ==="
                            ls -lh "$WORKSPACE/$CF_NAME"
                        '''
                    }
                }

                stage('Archive configuration file') {
                    steps {
                        archiveArtifacts(
                            artifacts: "${env.CF_NAME}",
                            fingerprint: true
                        )

                        stash(
                            name: 'configuration-cf',
                            includes: "${env.CF_NAME}"
                        )
                    }
                }
            }
        }

        stage('Deploy Configuration') {
            agent {
                label '1c-deploy'
            }

            stages {
                stage('Clean Deploy Workspace') {
                    steps {
                        echo 'Cleaning deploy workspace...'
                        deleteDir()
                    }
                }

                stage('Download configuration file') {
                    steps {
                        unstash 'configuration-cf'
                        
                        bat '''
                            @echo off
                            chcp 65001
                            
                            echo === Check CF ===
                            if not exist "%WORKSPACE%\\%CF_NAME%" (
                                echo ERROR: Configuration file not found
                                exit /b 1
                            )

                            dir "%WORKSPACE%\\%CF_NAME%"
                        '''
                    }
                }

                stage('Update Infobase') {
                    steps {
                        bat '''
                            @echo off
                            chcp 65001

                            echo === Check Infobase ===
                            if not exist "%INFOBASE_PATH%\\1Cv8.1CD" (
                                echo ERROR: Infobase not found
                                exit /b 1
                            )

                            echo === Load Configuration CF ===
                            vrunner cf load --ibconnection "/F%INFOBASE_PATH%" --db-user "%IB_USER%" --v8version "%V8_VERSION%" "%WORKSPACE%\\%CF_NAME%" 
                            
                            if errorlevel 1 exit /b 1

                            echo === Configuration loaded successfully ===
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }

        always {
            echo 'Pipeline finished.'
        }
    }
}
