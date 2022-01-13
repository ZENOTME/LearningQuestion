## tool ##
**Docker**:
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
  
**Tmux**:
  - Tmux is to use detach the session from window
  - tmux        //Create a session and a window
  - tmux detach //close the window detach the session from window 
  - tmux ls     //list the running session
  - tmux attach -t [id] //attach the session[id]
  - ctrl+b+%    //split the session, but they are seemly the same session. 
  - [More information](http://www.ruanyifeng.com/blog/2019/10/tmux.html)
