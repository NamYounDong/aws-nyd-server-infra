pipeline {
    agent any

    stages {
        stage('Checkout infra repo') {
            steps {
                // infra 전용 repo 사용 (실제 주소로 교체)
                git(
                    branch: 'main',
                    credentialsId: 'GIT_SSH_PRIVATE_KEY',     // SSH key
                    url: 'git@github.com:NamYounDong/aws-nyd-server-infra.git'
                )
            }
        }

        stage('Apply .env values for infra') {
            steps {
                // Secret text 3개를 환경변수로 받아오기
                withCredentials([
                    string(credentialsId: 'MARIADB_ROOT_PASSWORD', variable: 'MARIADB_ROOT_PASSWORD'),
                    string(credentialsId: 'REDIS_PASSWORD',   variable: 'REDIS_PASSWORD'),
                    string(credentialsId: 'TZ',         variable: 'TZ')
                ]) {
                    sh '''
                    cd infra

                    # 필요하다면 .env 파일 생성 (docker-compose에서 ${...} 참조 사용하는 경우)
                    cat > .env <<EOF
MARIADB_ROOT_PASSWORD=${MARIADB_ROOT_PASSWORD}
REDIS_PASSWORD=${REDIS_PASSWORD}
TZ=${TZ}
EOF

                    echo "[infra] .env 내용:"
                    cat .env
                    '''
                }
            }
        }

        stage('Deploy infra (MariaDB + Redis)') {
            steps {
                sh '''
                cd infra

                # DB/Redis만 재기동 (NPM/ Jenkins는 여기서 건드리지 않는 게 안전)
                docker compose pull nyd-mariadb nyd-redis || true
                docker compose up -d nyd-mariadb nyd-redis
                '''
            }
        }
    }
}
