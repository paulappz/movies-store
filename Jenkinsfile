def imageName = 'paulappz/movies-store'
// def registry = 'https://registry.gbnlcicd.com'
def registry = '530364773324.dkr.ecr.eu-west-2.amazonaws.com' 
def region = 'eu-west-2'

node('workers'){
    stage('Checkout'){
        checkout scm
    }

    def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")

    stage('Tests'){
        parallel(
            'Quality Tests': {
                sh "docker run --rm ${imageName}-test npm run lint"
            },
            'Integration Tests': {
                sh "docker run --rm ${imageName}-test npm run test"
            },
            'Coverage Reports': {
                sh "docker run --rm -v $PWD/coverage:/app/coverage ${imageName}-test npm run coverage-html"
                publishHTML (target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: "$PWD/coverage",
                    reportFiles: "index.html",
                    reportName: "Coverage Report"
                ])
            }
        )
    }

    stage('Build'){
        docker.build(imageName)
    }

    stage('Push'){
       sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${registry}/${imageName}"
     
       sh "  docker build -t ${imageName} . "
       sh " docker tag ${imageName}:latest ${registry}/${imageName}:latest"
       sh "docker push ${registry}/${imageName}:latest"

         //  imageBuild.push(commitID()) 
         //       if (env.BRANCH_NAME == 'develop') {
         //   imageBuild.push('develop')
         //        } 
    
}

    stage('Analyze'){
           def scannedImage = "${registry}/${imageName}:latest ${workspace}/Dockerfile"
           writeFile file: 'images', text: scannedImage
            anchore name: 'images'
    }
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}