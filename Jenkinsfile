pipeline {
    agent any
    
    environment{
        PGHOST=credentials('postgres-host')
        PGPORT=credentials('postgres-port')
        PGUSER=credentials('postgres-user')
        PGPASSWORD=credentials('postgres-password')
        PGDATABASE=credentials('postgres-dbname')
    }
    
    stages {
        stage('Create Tables') {
            steps {
                echo "Create Users table";
                sh '''
                psql -h $PGHOST -p $PGPORT -U $PGUSER -d $PGDATABASE -c "
                CREATE TABLE IF NOT EXISTS users(
                id SERIAL PRIMARY KEY,
                name VARCHAR(100) NOT NULL,
                email VARCHAR(150) UNIQUE NOT NULL,
                active BOOLEAN DEFAULT true,
                created_at TIMESTAMP DEFAULT NOW()
                );"
                '''
                echo "Create Orders table";
                sh '''
                psql -h $PGHOST -p $PGPORT -U $PGUSER -d $PGDATABASE -c "
                CREATE TABLE IF NOT EXISTS orders(
                id SERIAL PRIMARY KEY,
                product VARCHAR(100) NOT NULL,
                user_id INT NOT NULL,
                quantity INT DEFAULT 1 CHECK (quantity > 0),
                created_at TIMESTAMP DEFAULT NOW(),
                FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE 
                );"
                '''
                echo "Create Credentials table";
                sh '''
                psql -h $PGHOST -p $PGPORT -U $PGUSER -d $PGDATABASE -c "
                CREATE TABLE IF NOT EXISTS credentials(
                user_id INT PRIMARY KEY,
                login VARCHAR(100) UNIQUE NOT NULL,
                password_hash TEXT NOT NULL,
                FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE ON UPDATE CASCADE
                );"
                '''
            }
        }
    }
    post{
        always{
            echo 'Cleaning up tables...'
            sh '''
                psql -h $PGHOST -p $PGPORT -U $PGUSER -d $PGDATABASE -c "
                DROP TABLE IF EXISTS users CASCADE;
                DROP TABLE IF EXISTS orders CASCADE;
                DROP TABLE IF EXISTS credentials CASCADE;"
            '''
        }
    }
}
