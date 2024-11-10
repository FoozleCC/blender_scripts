import bpy
import os
import math

def render8directions_selected_objects(path):
    # path fixing
    path = os.path.abspath(path)

    # get list of selected objects
    selected_list = bpy.context.selected_objects

    # deselect all in scene
    bpy.ops.object.select_all(action='TOGGLE')

    s=bpy.context.scene

    s.render.resolution_x = 256 # set to whatever you want!
    s.render.resolution_y = 256
    
    # I left this in as in some of my models, I needed to translate the "root" object but
    # the animations were on the armature which I selected.
    # 
    # obRoot = bpy.context.scene.objects["root"]
    
    # The below is if you also want to render a normal to use in your engine for lighting.
    # For this to work, you need to:
    #    1. Enable "Normal" under View Layer - > Passes - > Data - > Normal in the properties panel.
    #    2. In the compositing node tree, you need to add an: Output - > File Output node.
    #    3. Connect the Normal node from the Rendered Layers Node to the File Output node.
    #    4. Select the File Output node and change the label in it's properties to 'Normal'. We will use this to dynamically find the node and dynamically change the output path.
    
    # Access the File Output node in the compositor for the normal map
    node_tree = s.node_tree
    normal_output = None
    for node in node_tree.nodes:
        if node.type == 'OUTPUT_FILE' and 'Normal' in node.label:  # Ensure it's the right node by label
            normal_output = node
            node.base_path = ""  # Clear the base path to prevent concatenation
            break

    if normal_output is None:
        print("Normal output node not found.")
        return
    # Assuming we have the normal_output node from the above, we can then use for each frame to update the output path while rendering lower in the code.
    
    # loop all initial selected objects (which will likely just be one obect.. I haven't tried setting up multiple yet)
    for o in selected_list:
        
        # select the object
        bpy.context.scene.objects[o.name].select_set(True)

        scn = bpy.context.scene
        
        # loop through the actions
        for a in bpy.data.actions:
            #assign the action
            bpy.context.active_object.animation_data.action = bpy.data.actions.get(a.name)
            
            #dynamically set the last frame to render based on action
            scn.frame_end = int(bpy.context.active_object.animation_data.action.frame_range[1])
            
            #set which actions you want to render.  Make sure to use the exact name of the action!
            if (
#                 a.name == "idle"
                 a.name == "run"
#                 or a.name == "dying"
                ):
                #create folder for animation
                action_folder = os.path.join(path, a.name)
                if not os.path.exists(action_folder):
                    os.makedirs(action_folder)
                
                #loop through all 8 directions
                for angle in range(0, 360, 45):
                    if angle == 0:
                        angleDir = "E"
                    if angle == 45:
                        angleDir = "NE"
                    if angle == 90:
                        angleDir = "N"
                    if angle == 135:
                        angleDir = "NW"
                    if angle == 180:
                        angleDir = "W"
                    if angle == 225:
                        angleDir = "SW"
                    if angle == 270:
                        angleDir = "S"
                    if angle == 315:
                        angleDir = "SE"
                        
                    #set which angles we want to render.
                    if (
                        angle == 0
                        or angle == 45
                        or angle == 90
                        or angle == 135
                        or angle == 180
                        or angle == 225
                        or angle == 270
                        or angle == 315
                        ):
                        
                        # Create folder for specific angle renders and normals
                        animation_folder = os.path.join(action_folder, angleDir)
                        if not os.path.exists(animation_folder):
                            os.makedirs(animation_folder)
                            
                        # Feel free to disable below if not using normals.
                        normal_folder = os.path.join(action_folder, f"{angleDir}_Normals")
                        if not os.path.exists(normal_folder):
                            os.makedirs(normal_folder)
                        
                        #rotate the model for the new angle
                        bpy.context.active_object.rotation_euler[2] = math.radians(angle)
                        
                        # the below is for the use case where the root needed to be translated.
#                        obRoot.rotation_euler[2] = math.radians(angle)
                        
                        
                        #loop through and render frames.  Can set how "often" it renders.
                        #Every frame is likely not needed.  Currently set to 2 (every other).
                        for i in range(s.frame_start,s.frame_end,1):
                            s.frame_current = i

                            s.render.filepath = (
                                                animation_folder
                                                + "\\"
                                                + str(a.name)
                                                + "_"
                                                + str(angle)
                                                + "_"
                                                + str(s.frame_current).zfill(3)
                                                )
                                                
                            # Set the full path for the normal map in the angle-specific normals folder
                            normal_output.file_slots[0].path = os.path.join(
                                normal_folder,
                                f"{a.name}_{angle}_{str(s.frame_current).zfill(3)}_normal"
                            )
                            
                            bpy.ops.render.render( #{'dict': "override"},
                                                  #'INVOKE_DEFAULT',  
                                                  False,            # undo support
                                                  animation=False, 
                                                  write_still=True
                                                 ) 
                                                 
        
render8directions_selected_objects('C:\\tmp\\Mutant')
