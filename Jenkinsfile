pipeline
{
    agent any
    {
        node
        {
            label 'customWorkspace - bgapp'
            customWorkspace '/projects/bgapp/web'
        }    
    }
  
    environment
    {
        MYSQL_ROOT_PASSWORD = ''
        PROJECT_ROOT = ''
    }

    stages
    {
        stage('Clean Workspace') 
        {
            steps
            {
                cleanWs()
            }
        }
        stage('Clone the Bgapp project')
        {
            steps
            {
                sh '''
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
                sh '''
                    cd /projects/bgapp
                    docker container rm -f db web || true
                    docker container run -d --net app-network --name db -e $MYSQL_ROOT_PASSWORD bgapp-db
                    docker container run -d --net app-network --name web -p 8082:80 -v $PROJECT_ROOT:/var/www/html bgapp-web

                    '''
            }
        }
    }
}