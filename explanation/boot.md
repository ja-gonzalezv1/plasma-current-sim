# Bootstrap Current
Now that we have found the current that is induced into the plasma by the central solenoid, we can find the current that originates due to the diamagnetic drift with the poloidal magnetic field. This field is generated due to the plasma current, and in this way, there cannot be any bootstrap current without first giving the plasma enough current, whether through induction or by any other methods.  
Because the expression that gives the bootstrap current actually gives a current density, the best way to analyze this quantity will be through a cross section of the plasma, in which we can calculate the current density at each point. To calculate the CS-induced current, we had simplified the plasma to a simple loop with no thickness. Now we will consider a plasma with a minor radius and an elliptical cross section.  
The cross section of the plasma will be elliptical as well, and the main axis of the plasma will be aligned with the main axis of the vessel. In our case, the main axis of the ellipse (the long radius) will be on the ${ y }$ axis, and the ellipse will have and aspect ratio (main axis over minor axis), which we will name ${ m }$. Then if we define a vessel radius ${ r_{ v }  }$, which will correspond to the minor axis, we will be able to give the mayor axis as ${ m \ r_{ v }  }$. The same will go for the plasma.  

For our simulation, we will have a vessel of (minor) radius of 2 m, a plasma of radius 1 m, and they will have an aspect ratio of 1.5. Then, when plotting a cross section of the vessel and the plasma inside of it, and centering the major axis with the origin, it will look like:
![Diagram of vessel and plasma cross section](bootstrap/attachments/v-plasma-cross.png) 

## Space initialization
The first thing that needs to be done before we can do any calculations is to create the space in which they will be made. So first we will create a rectangular mesh-grid, making sure that the number of points per side is odd. This is so that we can end up with a center point (a point with coordinates ${ \left( 0, 0 \right)  }$), which will be needed for later.  
In a similar way in which we found the points that were inside of the plasma loop to calculate the field flux, we will find the points that lie inside of the plasma cross section, to calculate the bootstrap current in them. There will be a small difference however, as back then the boundaries were circular, and now they are elliptical. This will make it a little more difficult to determine whether a point lies inside the boundary or not, as now the distance from the center to the boundary will be different depending on the polar angle $\theta$ that is being considered.  

For a given point in the mesh-grid, we can find said angle with the `arctan2` function in `numpy`. Normally, one would use `np.arctan2(y, x)`, but because we are using an ellipse in which the `y` coordinate is `m` times bigger than `x`, then we have to correct this by using `np.arctan2(y / m, x)`. Doing otherwise gives incorrect results.  
Once we find the angle, labeled as `tp` in the code, then we can find the corresponding distance to the boundary by using the Pythagorean theorem, where the coordinates of the boundary point can be found as ${ \left( b \cos \left( \theta  \right), a \sin \left( \theta  \right)  \right)  }$. Then we compare the distance from the center to a point with this distance to the boundary, and so we can determine if a point lies inside of the ellipse or not.  

After this, we can follow the same procedure of going through the mesh-grid, appending the points that are inside into a list. While we are doing this, we can also create more list for the distance of the point to the center, the distance of the corresponding boundary point to the center, and the angle `tp` of each point. All of this information will be useful for later.  

Once we have done this, we may end up with an array of points that looks like:
![Diagram of points inside plasma cross section](bootstrap/attachments/p-cross-points.png) 

## Bootstrap current calculation
Once we have created the space to make the calculations, we can start finding the necessary components to calculate the bootstrap current. Once of these components is the plasma pressure gradient ${ \vec{ \nabla } p }$. But instead of working directly with the plasma pressure, we can start with the particle number density ${ n }$, which is more commonly used as a system parameter.  

### Particle density gradient
Then, we are going to first find the particle density gradient ${ \vec{ \nabla } n }$. The way we are going to create this gradient is by setting it equal to a certain value ${ n_{ 0 } }$ at the center (coordinates ${ \left( 0,0 \right)  }$) point, and make it linearly decrease all the way to 0 at the boundary.  
The way we find what the center value ${ n_{ 0 } }$ must be is by using the fact that the sum of the particle densities, at all the different points inside the plasma ellipse, must be equal to the total particle density ${ n }$.  

With ${ n_{ 0 }  }$, named `n0` in the code, we find the linear gradient magnitude by dividing it over the distance to the boundary (for a given direction). This is because the particle density will go from this ${ n_{ 0 }  }$ value at the center, all the way to 0 at the boundary.  
We will also need the direction of the gradient, which will be the equivalent of a radial direction in the ellipse. To get it, we divide the coordinates of the point (equivalent of vector ${ \vec{ r } }$) over the distance from that point to the center (equivalent to the vector norm ${ r }$). This would give the unitary direction from the center of the ellipse to the point.  
Then, to each direction we multiply its corresponding magnitude, and the result would be the list of particle density gradient vectors at each point inside of the plasma ellipse.  

A precision can be made here about the dimensions of the particle density and our current simulation. Particle densities are given as a number over unit of volume. However, in our simulation we are using a plasma cross section, meaning we would be working with area, instead of volume. Then, to be completely formal and precise, we should work with an area particle density, instead of a volumetric particle density.  
However, this would yield results that apply to the plasma cross section, not the whole plasma. We would then have to scale the results back to make them applicable to the whole system. Instead, we are just going to assume that the plasma is completely symmetrical across its main axis, meaning that while we work and plot only a plasma cross section, this cross section will be the same for all other cross sections in the plasma. Then, while we only look at the plasma cross section, the results are true for the whole toroidal plasma.  

Another precision must be made, this time about the code. When finding the direction of the gradients, we divide the list of points over their corresponding distances to the center. However, for the center point, this division will yield a `nan` value. This may bring problems later on, so to fix it, we can just replace the obtained `nan` vector with a zero vector. This will give the sensible result of no particle gradient right at the center of the plasma. 

### Pressure gradient
Now that we have the particle density gradient, we can early find the pressure gradient like ${ \vec{ \nabla } p = k_{ B } T \vec{ \nabla } n }$. But, as we working with adimensional quantities, then the pressure gradient will be the same as the particle density gradient.  

Then, once we have obtained the list of gradients for each point, we can plot it (normalized and without dimension for better viewing) to get something like:
![Diagram with plasma pressure gradients](bootstrap/attachments/p-gradient.png)  
As can be seen, at each point is a vector that is pointing towards the center, which is what we should expect seeing how the number of particles increases in this direction, and a greater number of particles will generate a greater plasma pressure.

### Current density 
The next step would be to calculate the plasma's poloidal magnetic that is required to calculate the bootstrap current. For that, we are going to use Ampère's law, and for that we will need the value of the current density at each point inside of the plasma.  
This values will be found in the same way as the particle densities were found at each point. If we know the value of the density at the center (which is maximum there), we can find the value at a point by subtracting the corresponding change in the density, which will be the gradient times the distance from the center to the point.  
Doing this will give us a value of particle density at a point, which without dimension will actually be a fraction of the total plasma particle density. So then, we can use these same fractions to find the current density that belongs to each point, by multiplying the array of fractions by the value of the current density, at a given moment in time.  
The result will be one array that gives the current density at each point inside the plasma, per moment of time.  

As an example, we can plot a current density heatmap for the first instance in time, which corresponds to the current that would start being induced by the central solenoid. This would produce a graph like:
![Heatmap of current density at t = 0](bootstrap/attachments/p-curr-first.png)  
As can be seen, the current (density) is greater near the center of the plasma, as there is a greater concentration of particles and charges near that area. 

### Poloidal magnetic field
With the plasma pressure gradient, we would only be missing the plasma's poloidal magnetic field to calculate the bootstrap current at a given point. This field can be found using Ampère's law. For our plasma's elliptical geometry, the field would be given like:
$$
\begin{align*}
    B_{ p }  \left( 2 \pi \sqrt{\dfrac{  a_{ p } ^{ 2 } + b_{ p } ^{ 2 }   }{ 2 }}  \right) &= \mu_{ 0 } \left( \pi a_{ p } b_{ p } \right) \int \vec{ J } \cdot \text{d} \vec{ \hat{ A } } \\
    \\
    \vec{ B } _{ p } &= \dfrac{ \sqrt{ 2 } \mu_{ 0 } a_{ p } b_{ p }  }{ 2 \sqrt{ a_{ p } ^{ 2 } + b_{ p } ^{ 2 }  }  } \hat{ l }_{ B }  \int \vec{ J } \cdot \text{d} \vec{ A }
\end{align*}
$$
Where ${ \hat{ l }_{ B }   }$ is the (unitary) tangent vector to the surface of the ellipse, given like:
$$
\begin{align*}
    \hat{ l }_{ B }  &=  \dfrac{ - b \sin \left( \theta  \right) \hat{ x }  + a \cos \left( \theta  \right) \hat{ y } }{ \sqrt{ a^{ 2 } + b^{ 2 }  }  } 
\end{align*}
$$
And ${ \text{d} \hat{ A }  }$ is again a unitary area vector.  
${ a_{ p }  }$ and ${ b_{ p }  }$ would be the axes of a smaller ellipse (the amperian loop than encloses the current density), inside of the ellipse that defines the plasma boundary. These axes would have the same ratio between them as the plasma boundary ones.  
![Diagram of ellipses](bootstrap/attachments/eli-path.png)  
The quantities ${ \pi \sqrt{ a_{ p } ^{ 2 } + b_{ p } ^{ 2 }  }  }$ and ${ \pi a_{ b } b_{ p }  }$ would then be the perimeter and area of that inner ellipse.  
For a point ${ \left( x_{ p } , y_{ p }  \right)  }$ inside of the plasma, these parameters would be found like:
$$
\begin{align*}
    \theta _{ p } &=  \arctan2 \left( y_{ p } /m , x_{ p } \right) \\
    \\
    x^{ 2 } + y^{ 2 } &= b_{ p } ^{ 2 } \cos^{ 2 } \left( \theta _{ p }  \right) + a_{ p } ^{ 2 } \sin^{ 2 }  \left( \theta _{ p }  \right) \\
    &= b_{ p } ^{ 2 } \left( \cos^{ 2 } \left( \theta _{ p }  \right)  + m^{ 2 } \sin^{ 2 }  \left( \theta _{ p }  \right)  \right) \\
    \\
    b_{ p } &= \left( \dfrac{ x^{ 2 } + y^{ 2 }  }{ \cos^{ 2 } \left( \theta _{ p }  \right)  + m^{ 2 } \sin^{ 2 }  \left( \theta _{ p }  \right) } \right) ^{ 1/2 } \\
    a_{ p } &= m b_{ p } 
\end{align*}
$$

### Enclosed current
The only thing we would be missing to get the poloidal magnetic field would be the current that is enclosed by the amperian path. This is done through a similar integral to the one we used to calculate the flux of the CS field through the plasma loop. However, we will have to do some extra work to determine which points are being enclosed.  

Using the formulas shown above, we can find the axes ${ a_{ p }  }$ and ${ b_{ p }  }$, in the code named as `ae` and `be`, for the ellipse that goes through all the points inside of the plasma. With these, we can also find the perimeters and areas of the ellipses, as well as the tangent vectors at each point.  
Then, in a similar way in which we found which points were inside of the plasma elliptical boundary in the beginning, we can use `ae` and `be`, along with the distance of each point to the center (`d_points` in the code), to determine which points fall inside each inner ellipse.  
After we do this for every point inside of the plasma, we would get an array of arrays (with boolean values) that tells us which points are inside the amperian path inner ellipse, for every point inside the plasma.  

For example, for a point inside the plasma we can get the following inner ellipse, which would in turn give the following list of points that are inside it:
![Diagram with points inside inner ellipse](bootstrap/attachments/p-points-ins.png)  
The title of the plot also gives the area and perimeter we calculated for that same amperian path.  

Looking at the code, one may notice that a lot of quantities in this part are rounded to seven figures. This is because normally calculations are given with around 16 figures, where in some quantities, the last two differ for results that should be the same. This very small difference in quantities like the inner ellipse axes and the point distances makes it so, when finding which points are inside the inner ellipse, ellipses that should be the same and get the same list of points do not do so.  
This in turn makes it so that two points have a different enclosed current when it should be the same, which will do the same for the magnetic field and in turn the bootstrap current. This would be specially noticeable for the points near the center, which already only grab a few points inside their ellipses. The obtained results would be asymmetrical without any physical reason, so the rounding is there to fix this problem.  

Now we would have everything necessary to calculate the poloidal magnetic field. We can make this calculation through a function like:
```python
def b_poloidal (i, j):
    # get perimeter of corresponding elipse
    p = p_ellipse[i]
    # get the area
    a = a_ellipse[i]
    # get direction of field (tangent)
    d = e_tang[i]
    # get list of points inside of ellipse
    ins = e_inside[i]
    # calculate current with points that are inside
    I = 0
    for k in range(len(j)):
    if ins[k]:
        I += j[k]
    field = ((I * a) / p) * d
    return field
```
This function takes in the index `i` of the current point that is being evaluated, and the array or current densities `j` of the current moment in time. With the index it gets the corresponding perimeter and area of the amaperian loop, the tangent vector and the list of points that is inside said loop (the inner ellipse). Then it goes through every point in the current array, and it adds all the current densities of the points that are inside, ignoring the ones that are outside.  
The addition of these densities, multiplied by the area of the ellipse, would give the total enclosed current for Ampère's law. Then we divide over the perimeter and multiply times the tangent vector, and the result would be the (dimensionless) poloidal magnetic field vector that the plasma produces at a given point (inside of the plasma itself).

### Bootstrap current density
With the magnetic field now we would have everything necessary to calculate the bootstrap current density. We can also do this through a function:
```python
def bootstrap (bfield, pgrad):
    b2 = bfield[0]**2 + bfield[1]**2
    curr = np.cross(bfield, pgrad) / b2
    return curr
```
This function is basically the implementation of the corresponding mathematical formula, which is a function of magnetic field an pressure gradient. First we calculate the squared magnitude of the magnetic field, and we use it to divide the cross product between the magnetic field and the pressure gradient. The result is a single number (because the magnetic field and pressure gradient lie in the same plane, so the resulting vector from the cross product has only one component) which gives the corresponding bootstrap current density for a point.  

The only thing that would be left would be to calculate this bootstrap current density at each point inside the plasma, for each moment in time. The way we are going to do this is that we will go through each moment in time, getting the corresponding current density array, and then going through each point inside of the plasma, getting the pressure gradient, calculating the poloidal magnetic field and then calculating the bootstrap current density. Then, we will add this obtained current density to this same point for all moments in time after the current one. In this way, the state of the plasma will be constantly evolving, and we will dynamically calculate the changes in the system.  
To be able to do this, we will need a copy of the arrays of current density that are induced by the central solenoid (which will be called `jt`), in order to add the bootstrap current on top of it. To make this addition, we will also need to multiply the calculated bootstrap current by a factor (`Db` in the code), so that it is added to the dimensionless current densities in the correct proportion. We would also need a list with all the indexes in the time array (named `jti`), so that we can add the bootstrap current density for all points with an index greater than the current one (all moments in time after the current one).  

This calculations can be done using `for` loops, like:
```python
# go from first to penultimate moment in time
for i in range(len(jt) - 1):
    # for all points inside the plasma
    for k in range(len(p_inside)):
        # skip the center point
        if x[k] == 0 and y[k] == 0:
            continue
        # get corresponding current map
        j = jt[i]
        # get pressure gradient
        pg = pgrad[k]
        # get magnetic field
        bf = b_poloidal(k, j)
        # get bootstrap current 
        jb = bootstrap(bf, pg)
        # add the bootstrap current to the same point
        # to all moments after right now
        jt[jti > i, k] += Db * jb
```
First, we go through all the moments in time, minus the last one, so that we have a place to add the penultimate set of bootstrap currents. Then, we go through the list of all points inside of the plasma. We will skip the center point, as it will enclose no current, and there will be no magnetic field there to produce a diamagnetic drift (which is what produces the bootstrap current).  
Then, we get the current density array `j` for the actual moment in time and calculate the magnetic field `bf` at the current point inside of the plasma. We also get the pressure gradient `pg` at this same point, and with it we calculate the bootstrap current `jb` that it produces with the magnetic field, at this particular point. Having found this value, we will add it for all moments in time with an index that is greater than the current time index `i`, at the same point as the current one with index `k`.  

Once all of this is done, we will have an array of arrays, which gives the total current density (the one that is induced by the CS plus the one that is generated as bootstrap) at each point in the plasma, for each moment in time for the simulation.
