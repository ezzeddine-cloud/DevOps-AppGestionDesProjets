pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    // La VM Jenkins n'est pas exposée sur Internet : GitHub ne peut pas
    // appeler le webhook directement. Le polling détecte chaque push.
    triggers {
        pollSCM('* * * * *')
    }

    options {
        timestamps()
        skipDefaultCheckout(true)
    }

    stages {
        stage('Récupérer le projet depuis GitHub') {
            steps {
                checkout scm
            }
        }

        stage('Tests unitaires') {
            steps {
                dir('backend') {
                    sh 'mvn -B test'
                }
            }
        }

        stage('Créer le livrable') {
            steps {
                dir('backend') {
                    sh 'mvn -B package -DskipTests'
                }
            }
        }

        stage('Archiver le livrable target') {
            steps {
                archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true, onlyIfSuccessful: true
            }
        }
    }

    post {
        failure {
            mail to: 'vagrant@localhost',
                 subject: "Echec du build ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "La construction du backend a échoué.\nJob : ${env.JOB_NAME}\nBuild : ${env.BUILD_NUMBER}\nDétail : ${env.BUILD_URL}"
        }
    }
}
