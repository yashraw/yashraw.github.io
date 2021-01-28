---
name: Graduation Project-1
tools: [SolidWorks, ArduinoIDE, Designing]
image: https://images.unsplash.com/photo-1595278069441-2cf29f8005a4?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1951&q=80
description: Recycling of Plastics Using Affordable Injection Moulding Machine
#external_url: https://www.google.com
permalink: "/gradproject1"
---


## Recycling of Plastics Using Injection Moulding Machine

<!--
{% include elements/figure.html image="https://images.unsplash.com/photo-1611778030003-b681014615ff?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1955&q=30" caption="sample caption" %}
-->

In present scenario, production, use and disposal of plastics and its products are increasing, which is harmful to society. This improper disposing of plastics has been a major concern since a long time. They tend to disrupt the environment in a very serious way. They break into the soil creating smaller plastic pieces and release toxic chemicals, eventually eaten by the animals and therefore harming them. Plastic creates a whole lot of damage to the earth and people, some of which includes wildlife harm, clogged sewage systems.

We made an affordable Injection Moulding machine which will take recyclable plastics as raw material and will melt it depending upon the melting temperatur and will be injected out into a mould for the product to be made.

#### Objectives
- The main aim of the project is to make an affordable injection moulding machine out of readily available and cheap parts. This will not only reduce the amount of waste plastic but will also create new products.
- This machine can be an excellent part in the “SWACCH BHARAT” initiative taken by out PMO India, as it will reduce the plastic waste generated

The project started by a quick handrawn sketch followed by a quick CAD model in SolidWorks.  
{% include elements/figure.html image="\assets\img\1-1.png" caption="First Sketch" width = "150" height = "150" %}
Now we didn't made the entire machine exactly as the cad file because The plan was to built  with scrap parts, therefore we headed towards local scarpyards for scouting junk that would be usefull to us.
We got most of the parts from there at reasonable cost, around Rs.55 per Kg. Rest of the materials were purchased new and some components like Band Heaters and Thermocouple were also purchased new.
{% include elements/figure.html image="\assets\img\1-2.png"  %}


Now the whole heating element was controlled by the Arduino microcontroller, basically a desired and known melting temperature of the material to be inserted was inputted in the code and with the help of relay arrangement, the heating of the barrel was controlled, We only controlled the exit nozzle temperature and this can be improved by controlling the melting temperature first and increasing temperature right before ejecting out the nozzle to maintain optimal fluidity, but it was left aside for another revision.
{% include elements/figure.html image="\assets\img\1-4.png" caption="Schematics"  %}
The exact code used for this project can be found [here.](https://github.com/yashraw/InjectionMould) <br>


Below is the result of the first trial of this machine
{% include elements/figure.html image="\assets\img\1-3.jpg" caption= "First Trial" width = "150" height = "150" %}

