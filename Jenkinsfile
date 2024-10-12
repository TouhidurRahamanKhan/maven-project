pipeline{

    agent any 

    tools {
      maven 'my-maven'
     }

environment {
  NAME = "touhid"
}

parameters {
      //string defaultValue: 'khan', name: 'LAST_NAME'
      choice choices: ['dev', 'prod'], name: 'select_environment'
}

    stages{
        stage('build'){
           steps{
             script {
                file = load('script.groovy')
                file.hello()
             }

              sh 'mvn clean package -DskipTests=true'
           }
        }

        stage('Test')
        {
            parallel{
                stage('Test A')
                {
                    agent any
                    steps{
                        echo "Test A running"
                        sh 'mvn test'
                    }
                }

                stage('Test B')
                {
                    agent any
                    steps{
                        echo "Test B running"
                        sh 'mvn test'
                    }
                }
            }

            post{
            success{
                echo 'Test Complete'
                dir('webapp/target/')
                {
                     stash name: "maven-build", includes: "*.war"
                }
            }
           }
        }

        stage('deploy_dev')
    {
        when { expression {params.select_environment == 'dev'}
        beforeAgent true}
        agent any
        steps
        {
            dir("/var/www/html")
            {
                unstash "maven-build"
            }
            sh """
            cd /var/www/html/
            jar -xvf webapp.war
            """
        }
    }


     stage('deploy_prod')
    {
        when { expression {params.select_environment == 'prod'}
        beforeAgent true}
        agent any
        steps
        {

          timeout(time:5, unit:'DAYS'){
                input message: 'Deployment approved?'
             }

            dir("/var/www/html")
            {
                unstash "maven-build"
            }
            sh """
            cd /var/www/html/
            jar -xvf webapp.war
            """
        }
    }
    
    }
}