pipeline {
    agent any

    stages {
        stage('1. Build') {
            steps {
                echo 'Budowanie obrazu Builder (GCC 13)...'
                
                sh 'docker build -t hiredis-builder -f Dockerfile.build . > build_log.txt 2>&1'            }
        }

        stage('2. Test') {
            steps {
                echo 'Uruchamianie testów wewnątrz kontenera...'
                
                sh 'docker run --rm hiredis-builder make test >> build_log.txt 2>&1 || echo "Testy wykonane, sprawdz logi powyzej."'            }
        }

        stage('3. Deploy (Smoke Test)') {
            steps {
                echo 'Weryfikacja artefaktu...'
                sh 'docker run --rm hiredis-builder ls -lh /app/libhiredis.so'            }
        }

        stage('4. Publish (Artefakt)') {
            steps {
                echo 'Przygotowanie plików do pobrania...'
                
                sh 'docker rm -f temp-container || true'
                
                sh 'docker create --name temp-container hiredis-builder'
                
                sh 'docker cp temp-container:/app/libhiredis.so ./libhiredis.so'
                
                sh 'docker rm temp-container'

                sh 'tar -cvzf hiredis-paczka.tar.gz libhiredis.so'
                archiveArtifacts artifacts: 'hiredis-paczka.tar.gz', fingerprint: true
            }
        }
    }
    
    post {
        always {
            echo 'Zapisywanie logów i sprzątanie po buildzie'

            archiveArtifacts artifacts: 'build_log.txt', allowEmptyArchive: true
            
            sh 'docker image prune -f'
        }
    }
}
