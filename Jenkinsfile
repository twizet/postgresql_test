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
        stage('Creating Indexes'){
            steps{
                echo "Creating Indexes for easy SELECT";
                sh '''
                psql -h $PGHOST -p $PGPORT -U $PGUSER -d $PGDATABASE -c "
                CREATE INDEX idx_users_email ON users(email);
                CREATE INDEX idx_active_users ON users(email) WHERE active = true;
                CREATE INDEX idx_created_at ON orders(created_at);
                CREATE UNIQUE INDEX idx_user_login ON credentials(login);
                "
                '''
            }
        }
        stage('Add data'){
            steps{
                echo "Adding new data";
                sh '''
                psql -h $PGHOST -p $PGPORT -U $PGUSER -d $PGDATABASE -c "
                INSERT INTO users (name, email, active) VALUES
                                  ('Vasia','vassia@gmail.com',false),
                                  ('Kolia','kolich12@gmail.com',true),
                                  ('Katia','ekatrina@gmail.com',true),
                                  ('Sofia','sof4a@gmail.com',true);
                SELECT * FROM users;
                INSERT INTO credentials (login, password_hash, user_id) VALUES
                                        ('vasia', 'vasiawd3i323', 1),
                                        ('kolich', 'kolikanabolik2', 2),
                                        ('katin', 'kotioa221', 3),
                                        ('fastsofa', 'biwewwef11231', 4);
                SELECT * FROM credentials;
                INSERT INTO orders (product, quantity, user_id) VALUES
                                   ('grusha', 4, 2),
                                   ('borukva', 1, 1),
                                   ('chipsy', 1, 4),
                                   ('cebula', 12, 3);
                SELECT * FROM orders;
                "
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
