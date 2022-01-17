## Trap and switch in rCore ##
- when a process trap into the kernel, kernel should store it's context.
- when a kernel prepare to run a applicaion, it should prepare a new context.<br>

### What things should the kernel prepare or store? ###
![avatar](./img/trap_switch_00.png)
### Look at what happen when task switch ###
![avatar](./img/trap_switch_01.png)

