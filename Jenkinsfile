pipeline
{
    agent
     {
        label {
            label ""
            customWorkspace "/projects/bgapp"
        }
    }
    
 //   options([
 //       parameters([
 //           password(name: 'MYSQL_ROOT_PASSWORD', description: 'Encryption key')
 //           string(name: 'PROJECT_ROOT', defaultValue: '/projects/bgapp/web', description: 'PROJECT_ROOT')
 //   ])  
//]) 

   parameters
    {
        string(name: 'PROJECT_ROOT', defaultValue: '/projects/bgapp/web', description: 'PROJECT_ROOT')
        password(name: 'MYSQL_ROOT_PASSWORD', defaultValue: '12345', description: 'Enter DB password')
    }

    //environment
    //{
    //    MYSQL_ROOT_PASSWORD = '12345'
       // PROJECT_ROOT = '/projects/bgapp/web'
    //}

    stages
    {
        
        stage('Clone the Bgapp project')
        {
            steps
            {   
                sh '''
                    sudo rm -fr /projects/bgapp
                    cd /projects
                    git clone https://github.com/gogotomov/bgapp.git
                    echo 'Bgapp project was successfully cloned !!!'
               '''
            }
        }

        stage('Build the images')
        {
            steps
            {
                echo 'Master: Build Slave'

                script
                {
                    sh '''
                    cd /projects/bgapp
                    docker image build -t bgapp-web -f Dockerfile.web .
                    docker image build -t bgapp-db -f Dockerfile.db .
                    '''
                }
            }
        }

        stage('Create network')
        {
            steps
            {
                sh 'docker network ls | grep app-network || docker network create app-network'
            }
        }

        stage('Create containers')
        {
            steps
            {
                wrap([$class: 'MaskPasswordsBuildWrapper', varPasswordPairs: [[var: 'DB_PASSWORD', password: "${MYSQL_ROOT_PASSWORD}"]]])
                {
                sh '''
                    cd /projects/bgapp
                    docker container rm -f db web || true
                    docker container run -d --net app-network --name db -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} bgapp-db
                    docker container run -d --net app-network --name web -p 8082:80 -v "${PROJECT_ROOT}":/var/www/html bgapp-web

                   '''
                }
            }
        }
    }
}