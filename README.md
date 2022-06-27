
---------------------------------------------------------
error:
---------------------------------------------------------
NVIDIA GeForce RTX 3090 with CUDA capability sm_86 is not compatible with the current PyTorch installation.
The current PyTorch install supports CUDA capabilities sm_37 sm_50 sm_60 sm_61 sm_70 sm_75 compute_37.
If you want to use the NVIDIA GeForce RTX 3090 GPU with PyTorch, please check the instructions at https://pytorch.org/get-started/locally/

fix:
install cuda 11.7 (install all with drivers ...)
check that cuda path in sys vars == same as cuda path_117
install torch from site for cuda 11.3 # yes, 11.3 but it works
conda install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch
see server images


---------------------------------------------------------
error:
---------------------------------------------------------
File "C:\Users\Admin\Desktop\Efficient_goal-oriented_push-grasping_synergy\robot.py", line 130, in add_objects
    curr_shape_handle = ret_ints[0]
IndexError: list index out of range
fix:
	close appeared window in coppelia


--------------------------------------------------
error1: robot.py", line 284, in get_obj_mask
    sim_ret, obj_position = vrep.simxGetObjectPosition(self.sim_client, self.object_handles[obj_ind], -1, vrep.simx_opmode_blocking)
    IndexError: list index out of range
reason: only one object in scene and we wanna get his mask
    but we are trying to get other object
NOT fix:
    hide arm in vision sensor
    install 4.2.0 instead 4.3.0 (cause 2 has flag renderable)
experiment "goal_obj_idx" = 1:
    def get_obj_mask(self, obj_ind):
        print('object_handles count = ',len(self.object_handles), ' obj_ind=',obj_ind)
        ==> count=1, obj_ind=2
        sim_ret, obj_position = vrep.simxGetObjectPosition(self.sim_client, self.object_handles[obj_ind], -1, vrep.simx_opmode_blocking)
fix1: set argument goal_obj_idx = 0
    looks ok, in 1.2 no errors anymore
    python main.py --stage grasp_only --num_obj 5 --goal_conditioned --goal_obj_idx 0 --experience_replay --explore_rate_decay --save_visualizations
fix2: goal_obj_idx = 0 is not enough, this will guarantee good result:
    def get_obj_mask(self, obj_ind):
        # Get object pose in simulation
        print('object_handles count = ',len(self.object_handles), ' obj_ind=',obj_ind)
        # fixes error that occurs sometimes, when obj_ind is too big
        if obj_ind >= len(self.object_handles):
            obj_ind = 0
            print('obj_ind fixed, now = ', obj_ind)



---------------------------------------------------------
error2:
    after 120 iterations:
    Exception in thread Thread-2:
    Traceback (most recent call last):
      File "C:\Users\Admin\.conda\envs\eff_env38\lib\threading.py", line 932, in _bootstrap_inner
        self.run()
      File "C:\Users\Admin\.conda\envs\eff_env38\lib\threading.py", line 870, in run
        self._target(*self._args, **self._kwargs)
      File "main.py", line 166, in process_actions
        robot.add_objects()
      File "C:\Users\Admin\Desktop\Efficient_goal-oriented_push-grasping_synergy\robot.py", line 126, in add_objects
        ret_resp,ret_ints,ret_floats,ret_strings,ret_buffer = vrep.simxCallScriptFunction(self.sim_client, 'remoteApiCommandServer',vrep.sim_scripttype_childscript,'importShape',[0,0,255,0], object_position + object_orientation + object_color, [curr_mesh_file, curr_shape_name], bytearray(), vrep.simx_opmode_blocking)
      File "C:\Users\Admin\Desktop\Efficient_goal-oriented_push-grasping_synergy\simulation\vrep.py", line 1393, in simxCallScriptFunction
        ret = c_CallScriptFunction(clientID,scriptDescription,options,functionName,len(inputInts),c_inInts,len(inputFloats),c_inFloats,len(inputStrings),c_inStrings,len(inputBuffer),inputBufferV,ct.byref(intDataC),ct.byref(intDataP),ct.byref(floatDataC),ct.byref(floatDataP),ct.byref(stringDataC),ct.byref(stringDataP),ct.byref(bufferS),ct.byref(bufferP),operationMode)
    OSError: exception: access violation reading 0x0000028010E38000
    Training loss: 0.022923

Links:
    https://stackoverflow.com/questions/71316741/python-ctypes-x64-oserror-exception-access-violation-reading-0x000000000000000
    https://forum.coppeliarobotics.com/viewtopic.php?t=7858
    I found that if the function be called by client program through simxCallScriptFunction need long time to execute, the above errors will arise.
Reason:
    this is because you call a function that blocks. Currently there is a connection timeout of 1 second, which is too little. You can modify the timeout value in b0RemoteApi.py, around line 25.
fix:
    C:\Program Files\CoppeliaRobotics\CoppeliaSimEdu\programming\b0RemoteApiBindings\python
    b0RemoteApi.py
    self._serviceClient.set_option(3,timeout*25000) #read timeout #was 1000
