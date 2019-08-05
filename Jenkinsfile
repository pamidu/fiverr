pipeline {
	agent any
    environment {        //DOCKERHUB = credentials('dockerhub-credentials')
        DOCKERHUB_USR = "pamidu@live.com"
        DOCKERHUB_PSW = "Pol-19901222"
    }

    stages {
        stage('Find app name to build') {
            steps {
                script {
                    if (REF != "") {
                        VALUESFILE = sh(returnStdout: true, script:'git show --name-only --pretty=""')
                        LIST = VALUESFILE.split('\n')
                        def MAP = [:]
                        for(String file in LIST) {
                            MAP.put(file.split('/')[0], "build");
                        }
                        APPS = MAP.keySet()
                        echo "${APPS}"
                    }
                }
                echo "Changes in:${VALUESFILE}"
                echo "application to build:${APPS}"
            }
        }
        
        stage('Build and push docker image') {
            steps {
                container('docker') {
                    script {
                        sh 'docker login -u $DOCKERHUB_USR -p $DOCKERHUB_PSW'
                        for(String app in APPS) {
                            TO_BUILD = sh (
                                script: "ls ${app}/Dockerfile",
                                returnStatus: true
                            )
                            if (TO_BUILD == 0) {
                                env.APP = app
                                env.GIT_SHA = sh(returnStdout: true, script: "git rev-parse --short HEAD")
                                sh '''docker build -t pamidu/app-mono-${APP}:${GIT_SHA} ./${APP}/
                                docker push pamidu/app-mono-${APP}:${GIT_SHA}
                                docker rmi pamidu/app-mono-${APP}:${GIT_SHA}
                                '''
                            }
                        }
                        sh 'docker logout'
                    }
                }
            }
        }
    }
    post {
        cleanup {
            deleteDir()
        }
    }
}
