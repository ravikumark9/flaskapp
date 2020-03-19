node {
stage (‘Prepare environment’) {
git branch: ‘master’, url: ‘git@github.com:ravikumark9/flaskapp.git’
}
stage (‘Build’) {
sh ‘docker build -t testflask .’
}
}
