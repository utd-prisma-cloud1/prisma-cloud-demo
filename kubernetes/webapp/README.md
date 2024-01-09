# Kubernetes workshop

## Workshop Interaction

In the workshops above you will be launching a two tier webapp. The web app is a Ruby app using Sinatra. The app is a key value storage and retrieval service with a Redis backend. This is based on [k8s-workshop](https://github.com/FairwindsOps/k8s-workshop/blob/master/README.md?plain=1)

### Interacting with the App
* [instance_ip]:[port]/[key]/[value] will set a key value
* [instance_ip]:[port]/[key] will retrieve the value
* [instance_ip]:[port] returns "Hello from Kubernetes"

### Optional Configuration
* When environment variable `CHAOS` is set to `true` then the webapp will die after a randomly generated number of requests between 1 and 100.
