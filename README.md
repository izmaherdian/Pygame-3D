# Pygame-3D

![image](https://github.com/Anthony-Gambale/Pygame-3D/blob/main/images/1_screenshot.png)

Pygame 3D is a 3D renderer for Pygame. It uses Pygame's 2D rendering functions to display 3D models.

This project is an experimental demo of a mathematical technique that I came up with. Using this technique, the renderer can skip the "view space" phase of traditional 3D rendering, saving a significant chunk of computation time.

### Install and Run
```
$ git clone https://github.com/Anthony-Gambale/Pygame-3D.git
$ python -m pip install pygame
$ python Pygame-3D/src/main.py
```

## Traditional renderer "view space" computation
In a traditional 3D renderer, whenever some transformation R would be applied to the camera, the inverse of that transformation is applied to every model in the scene instead.  

This creates the illusion that R is being applied to the camera, without having to move the screen plane.

This computation is referred to as transforming into "view space." This is very expensive, especially for intricate models, and scales with the detail in the scene.

![image](https://github.com/Anthony-Gambale/Pygame-3D/blob/main/images/2_traditional_rotate.png | width=600)  
*Figure 1: Traditional method for rotating camera. Computation scales with complexity of 3D models.*  

## The difference: robust plane modelling
In my method, the screen plane is modelled in a more robust way.  

In a traditional 3D renderer, the screen plane must remain parallel to the xy plane at all times. However, if the plane is modelled more robustly, it is possible to perform matrix transformations on its components, as shown in Figure 2. This saves a large chunk of computation time.

![image](https://github.com/Anthony-Gambale/Pygame-3D/blob/main/images/3.0_my_rotate.png | width=600)  
*Figure 2: My method for rotating camera. Computation required is constant, and will never scale.*

A screen plane is defined by local basis vectors and a normal vector, as shown in Figure 3. These vectors are all orthonormal to each other.

![image](https://github.com/Anthony-Gambale/Pygame-3D/blob/main/images/3.1_plane_definition.png | width=600)  
*Figure 3: Robust definition of a screen. Plane with orthonormal basis.*  

Figure 4 has a camera point and a screen plane together. The camera point is offset from s by a factor k of the normal vector.

![image](https://github.com/Anthony-Gambale/Pygame-3D/blob/main/images/3.2_plane_definition.png | width=600)  
*Figure 4: Screen defined robustly, and paired with a camera point.*

Whenever a transformation is made to the camera (i.e. translating the camera, rotating the camera) these transformations are applied to the camera vectors and values, rather than to the world around the camera. This is where computation is saved.  

## Projecting onto robustly modelled plane
When using the robust plane definition, it is more difficult to project 3D models. My technique for this is as follows.  

 - Define ray as parametric line
 - Substitute ray point into plane equation, and solve for t
 - Substitute t back into line equation to get intersection point x

Note that I left out the algebra steps I took to get to the final result.  

![image](https://github.com/Anthony-Gambale/Pygame-3D/blob/main/images/4_intersections.png | width=600)  
*Figure 5: Calculating the point of intersection of a ray and screen plane.*  

## Transforming screen plane to xy plane
Once the intersection point is found, it must be mapped onto the xy plane, giving pixel coordinates for rasterization.

 - Subtract intersection point x from centerpoint s, to get relative distance d
 - Project d onto local basis vectors to get x and y magnitudes  

![image](https://github.com/Anthony-Gambale/Pygame-3D/blob/main/images/5_xy_transform.png | width=600)  
*Figure 6: Projecting relative distance onto local basis vectors.*  

## The implications of this technique
This technique has the potential to be much faster than current 3D renderering techniques, provided that there are no major issues with implementing it into a real 3D rendering API.
