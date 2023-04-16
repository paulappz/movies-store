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
        sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${registry}/${imageName}"
        sh "docker build --build-arg ENVIRONMENT=sandbox --tag ${imageName}:${commitID()} ."
        sh " docker tag ${imageName}:${commitID()} ${registry}/${imageName}:${commitID()}"
    }
    
    stage('Push'){
       sh "docker push ${registry}/${imageName}:${commitID()}"
    }

    stage('Analyze'){
           def scannedImage = "${registry}/${imageName}:${commitID()} ${workspace}/Dockerfile"
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