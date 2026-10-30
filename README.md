# Plasma Current Simulation

This is a simulation for the current of a plasma in a toroidal machine. The types of current that are simulated are the induced current by a central solenoid, and the bootstrap current, modeled as a diamagnetic current. For the induced current, the magnetic field flux of a central solenoid is calculated. For the bootstrap current, the pressure gradient of the plasma is used, and the poloidal magnetic field its current produces is calculated. The relevant calculations are made for the plasma's cross section.  
The physics that are simulated in the code are explained, as well as the thought process behind their implementation. The simulation is made in Python, and presented in a Jupyter notebook. 

<!-- TODO: add some text saying:
           to see the physics of the plasma behind the diamagnetic drift, link to two-fluid pdf
           to see an implementation in c++, link the repo
           an improved fluid simulation was also done, link repo --> 

## Table of Contents:

1. [Physical Explanation](~/explanation/physics.md):
    1.1 [Overview](~/explanation/physics.md##overview) 
    1.2 [Central Solenoid Induction](bootstrap/explanation/physics.md##central-solenoid-induction) 
        1.2.1 [Central Solenoid Magnetic Field](bootstrap/explanation/physics.md###cs-magnetic-field) 
        1.2.2 [Field Flux](bootstrap/explanation/physics.md###field-flux) 
        1.2.3 [Induced Electromotive Force](bootstrap/explanation/physics.md###induced-emf) 
        1.2.4 [Plasma Resistivity](bootstrap/explanation/physics.md###plasma-resistivity) 
        1.2.5 [Induced Current](bootstrap/explanation/physics.md###induced-current) 
    1.3 [Bootstrap Current](bootstrap/explanation/physics.md##bootstrap-current) 
        1.3.1 [Particle Density and its Gradient](bootstrap/explanation/physics.md###particle-density-and-gradient) 
        1.3.2 [Poloidal Magnetic Field](bootstrap/explanation/physics.md###poloidal-magnetic-field) 
        1.3.3 [Plasma Current Density](bootstrap/explanation/physics.md###current-density) 
        1.3.4 [Total Plasma Current](bootstrap/explanation/physics.md###total-plasma-current) 

2. [Central Solenoid Current Induction](bootstrap/explanation/central-solenoid.md):
    2.1 [Current Profile](bootstrap/explanation/central-solenoid.md##current-profile) 
    2.2 [Magnetic Field](bootstrap/explanation/central-solenoid.md##magnetic-field) 
        2.2.1 [Definition of Central Solenoid](bootstrap/explanation/central-solenoid.md###cs-definition) 
        2.2.2 [Biot-Savart Law](bootstrap/explanation/central-solenoid.md###biot-savart-law) 
    2.3 [Induced EMF](bootstrap/explanation/central-solenoid.md##induced-emf) 
        2.3.1 [Magnetic Field Flux](bootstrap/explanation/central-solenoid.md###field-flux) 
        2.3.2 [Faraday's Law](bootstrap/explanation/central-solenoid.md###faraday's-law) 
    2.4 [Induced Current](bootstrap/explanation/central-solenoid.md##induced-current) 
        2.4.1 [Current Density](bootstrap/explanation/central-solenoid.md###current-density) 
        2.4.2 [Plasma Resistance](bootstrap/explanation/central-solenoid.md###plasma-resistance) 
        2.4.3 [Induced Plasma Current](bootstrap/explanation/central-solenoid.md###cs-induced-plasma-current) 

3. [Bootstrap Current](bootstrap/explanation/boot.md):
    3.1 [Initialization of Space](bootstrap/explanation/boot.md##space-initialization) 
    3.2 [Calculation of Bootstrap Current](bootstrap/explanation/boot.md##bootstrap-current-calculation) 
        3.2.1 [Particle Density Gradient](bootstrap/explanation/boot.md###particle-density-gradient) 
        3.2.2 [Pressure Gradient](bootstrap/explanation/boot.md###pressure-gradient) 
        3.2.3 [Current Density](bootstrap/explanation/boot.md###current-density) 
        3.2.4 [Poloidal Magnetic Field](bootstrap/explanation/boot.md###poloidal-magnetic-field) 
        3.2.5 [Enclosed Current](bootstrap/explanation/boot.md###enclosed-current) 
        3.2.6 [Bootstrap Current Density](bootstrap/explanation/boot.md###bootstrap-current-density) 

## Summary
The simulation first creates a current profile for the central solenoid, and then it calculates the magnetic field it would produce. Next, the flux this field creates through a plasma is calculated, as well as its change through time. With this, we find the EMF that is induced into the plasma, and the corresponding current.  
Once with the current profile the solenoid induced into the plasma, the bootstrap current is calculated. The plasma is given a pressure gradient, and the poloidal magnetic field is calculated with the plasma's current (density). The resulting current density is added to the previously found induced current profile, and so a total current profile is found for the simulated plasma.

## Results

### Current density in plasma cross section
![Current density in plasma cross section, as time advances](bootstrap/attachments/gif.mp4) 

### Induced and bootstrap current
![Induced current through time, along with the bootstrap current](bootstrap/attachments/b-curr-vs-cs.png) 

- The central solenoid induces a current of 3 MA (mega Amperes), and the bootstrap current can increase the total plasma current up to 100 MA.
- The 98% of the total plasma current is due to the bootstrap current.
- Bootstrap current can increase the total plasma current without the need of any external systems or energy input. 
