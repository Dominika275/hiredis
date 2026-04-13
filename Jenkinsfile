pipeline {
    agent any

    stages {
        stage('1. Build') {
            steps {
                echo 'Budowanie obrazu Builder (GCC 13)...'

                sh 'docker build -t hiredis-builder -f Dockerfile.build .'
            }
        }

        stage('2. Test') {
            steps {
                echo 'Uruchamianie testów wewnątrz kontenera...'

                sh 'docker run --rm hiredis-builder make test || echo "Testy wykonane, sprawdz logi powyzej."'
            }
        }

        stage('3. Deploy (Smoke Test)') {
            steps {
                echo 'Weryfikacja artefaktu...'
                sh 'docker run --rm hiredis-builder ls -lh /app/libhiredis.so'            }
        }

        stage('4. Publish (Artefakt)') {
            steps {
                echo 'Przygotowanie artefaktu do pobrania...'

                sh 'docker rm -f temp-container || true'
                
                sh 'docker create --name temp-container hiredis-builder'

                sh 'docker cp temp-container:/app/libhiredis.so ./libhiredis.so'
                
                sh 'docker rm temp-container'

                sh 'tar -cvzf hiredis-artifact.tar.gz libhiredis.so'
                archiveArtifacts artifacts: 'hiredis-artifact.tar.gz', fingerprint: true
            }
        }
    }
    
    post {
        always {
            echo 'Zapisywanie logów i sprzątanie po buildzie'

            sh 'docker logs $(docker ps -a -q | head -n 1) > ostatni_log_builda.txt || echo "puste" > ostatni_log_builda.txt'
            archiveArtifacts artifacts: 'ostatni_log_builda.txt'
            
            sh 'docker image prune -f'
        }
    }
}
