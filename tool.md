## tool ##
Docker:
  - About image:
    - Use Dockerfile to build a image 
    - Pull image from Hub
    - make changes in container and save it as a new image 
      docker commit -m= -a= [container ID] [new image name]
  - About containner:
    - docker run -it -w [path] [image]       //create interactively a container and the working dictionary set as path
    - docker run [image] [command] //create a container and exec the command 
    - docker attach [container ID] //attach a container 
    - docker exec [container ID] [command]// let the container to exec the command
    - docker start [container]     //restart a container which is close 
    - exit                         //close the container 
    - ctrl+p ctrl+q                //hang up the container which is still running
  - About data
    - --mount type=bind,source=a,target=b //mount the a(from host) to b(from container)
    - -v [a]:[b]                          //the same functionality as above
  
