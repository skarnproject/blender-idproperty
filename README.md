# About

This Blender addon aids in addon development by providing `IDProperty`, a [Property](https://www.blender.org/api/blender_python_api_2_76_release/bpy.props.html) that is able to point to another object.  Some important features:

* Provides a unique id field on all Objects, accessed by the `.id` property
* Doesn't break when the object pointed to changes its name
* Includes a convenience function for adding the property to a Panel's layout
* Automatically avoids id collisions from [linked blend files](https://www.blender.org/manual/data_system/linked_libraries.html#append-and-link)

# Basic Usage

This code sets up two IDProperties on all objects.  The first property, `some_related_object` can point to any object.  The second property, `some_camera`, will only point to Cameras.  The Panel for these properties is displayed on the Object Properties tab of the information Area.

![http://i.imgur.com/oCbutrk.png](http://i.imgur.com/oCbutrk.png)

```python
import bpy
from bpy.utils import register_module, unregister_module
import idproperty

class SomePanel(bpy.types.Panel):
    bl_label = "Some Properties"
    bl_space_type = "PROPERTIES"
    bl_region_type = "WINDOW"
    bl_context = "object"
    
    def draw(self, ctx):
        ob = ctx.object
        
        layout = self.layout
        row = layout.row()
        idproperty.layout_id_prop(row, ob, "some_related_object")
        
        row = layout.row()
        idproperty.layout_id_prop(row, ob, "some_camera")
        
def is_camera(ob):
    return ob.type == "CAMERA"

def register():
    bpy.types.Object.some_related_object = idproperty.ObjectIDProperty(name="something related")
    bpy.types.Object.some_camera = idproperty.ObjectIDProperty(name="some camera", validator=is_camera)
    register_module(__name__)
    
def unregister():
    del bpy.types.Object.some_related_object
    del bpy.types.Object.some_camera
    unregister_module(__name__)
    
try:
    unregister()
except:
    pass
    
register()
```

# Installation

1. Click "Download Zip" on Github.
2. Save the zip somewhere.
3. From Blender, visit File -> User Preferences.
4. Go to Add-ons.
5. Click "Install from File..."
6. Select the zip file that you saved.

# Supported ID Properties

* ObjectIDProperty
* GroupIDProperty
* MaterialIDProperty
* LibraryIDProperty

Adding support for other ID types is trivial.

# How does it work?

The first step to enabling this functionality was the establishment of unique ids per object, and doing our best to enforce their uniqueness.  By having unique ids that don't change, it means that a Property can point to something other than the object's name, and is therefore resilient to the object's name changing.  Unique ids are implemented through a scene-level unique id counter.  An object initially starts out with no set `.id` field, but when it is fetched, we use the value from the scene unique id counter, then increment the counter.  In fact, these counters are incremented and kept in sync across *all* scenes, so that objects shared between scenes will never have an id collision.

The second step was making the `IDProperty` property point to unique ids, but display object names.  This is done by a getter and setter on `IDProperty`, as well as a simple lookup function that maps id to object.
