Docker works in a client server model, the thing with which we interact  is called a docker client and this client sends the request to the docker server which is running in the background . The docker server is just a background service called docker daemon. 
when we install docker on our local system we are installing both docker client and docker server in that.

And we can also configure docker in such a way that it will use a server in someother place as docker server and client as our local machine

`$env:DOCKER_HOST="tcp://<REMOTE_IP>:2376"`

just by using this command we can change the docker host address and it will be able to talk to a docker host situated in some other place

