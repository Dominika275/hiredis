pipeline {
    agent any

    stages {
        stage('0. Clean & Checkout') {
            steps {
                cleanWs()
                checkout scm
            }
        }

        stage('1. Build (BLDR)') {
            steps {
                echo 'Budowanie obrazu buildowego (BLDR)'
                sh 'docker build -t hiredis-bldr -f Dockerfile.build .'
            }
        }

        stage('2. Test') {
            steps {
                echo 'Uruchamianie testów jednostkowych...'
                sh 'docker run --rm hiredis-bldr make test'
            }
        }

        stage('3. Prepare Artifact') {
            steps {
                echo 'Wyciąganie pliku binarnego z obrazu BLDR...'
                sh 'docker rm -f temp-container || true'
                sh 'docker create --name temp-container hiredis-bldr'
                sh 'docker cp temp-container:/app/libhiredis.so ./libhiredis.so'
                sh 'docker rm temp-container'
                
                sh 'tar -cvzf hiredis-paczka.tar.gz libhiredis.so'
            }
        }

        stage('4. Deploy (Sandbox Verification)') {
            steps {
                echo 'Wdrożenie i weryfikacja w środowisku sandboxowym...'
                sh 'docker run --rm -v $(pwd)/libhiredis.so:/usr/lib/libhiredis.so debian:slim ls -lh /usr/lib/libhiredis.so'
            }
        }

        stage('5. Publish') {
            steps {
                echo 'Publikacja artefaktów do historii builda...'
                archiveArtifacts artifacts: 'hiredis-paczka.tar.gz', fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Sprzątanie środowiska (repeatability)...'
            sh 'docker image prune -f'
            
            sh 'echo "Logi procesu CI/CD" > build_log.txt'
            archiveArtifacts artifacts: 'build_log.txt'
        }
    }
}
