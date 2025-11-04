---
name: Recyling of Plastics 
tools: [SolidWorks, ArduinoIDE, Designing, Automation]
image: https://www.rts.com/wp-content/uploads/2020/10/shutterstock_1345281257_opt.jpg
description: Recycling of Plastics Using Affordable Injection Moulding Machine Made in-house.
#external_url: https://www.google.com
---


## Recycling of Plastics Using Injection Moulding Machine

<!--
{% include elements/figure.html image="https://images.unsplash.com/photo-1611778030003-b681014615ff?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1955&q=30" caption="sample caption" %}
-->

In present scenario, production, use and disposal of plastics and its products are increasing, which is harmful to society. This improper disposing of plastics has been a major concern since a long time. They tend to disrupt the environment in a very serious way. They break into the soil creating smaller plastic pieces and release toxic chemicals, eventually eaten by the animals and therefore harming them. Plastic creates a whole lot of damage to the earth and people, some of which includes wildlife harm, clogged sewage systems.

We made an affordable Injection Moulding machine which will take recyclable plastics as raw material and will melt it depending upon the melting temperature and will be injected out into a mould for the product to be made.

<p>&nbsp;</p>
#### Objectives

- The main aim of the project is to make an affordable injection moulding machine out of readily available and cheap parts. This will not only reduce the amount of waste plastic but will also create new products.
- This machine can be an excellent part in the “SWACCH BHARAT” initiative taken by out PMO India, as it will reduce the plastic waste generated

The project started by a quick handrawn sketch followed by a quick CAD model in SolidWorks.  
Below is what we designed in *SolidWorks.*
<div class="sketchfab-embed-wrapper" style="position: relative; width: 100%; padding-top: 56.25%;">
  <iframe
    title="Injection Moulding Machine"
    frameborder="0"
    allowfullscreen
    mozallowfullscreen="true"
    webkitallowfullscreen="true"
    allow="autoplay; fullscreen; xr-spatial-tracking"
    src="https://sketchfab.com/models/4ddd548cdbcc4ff1989b931a893f9152/embed?autospin=1&autostart=1&preload=1&ui_hint=2"
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none;">
  </iframe>
</div>


Now we didn't make the entire machine exactly as the cad file because The plan was to built  with scrap parts, therefore we headed towards local scarpyards for scouting junk that would be usefull to us.
We got most of the parts from there at reasonable cost, around Rs.55($~1CAD) per Kg. Rest of the materials were purchased new and some components like Band Heaters and Thermocouple were also purchased new.
{% include elements/figure.html image="/assets/img/1-2.png"  %}


The heating system was controlled using an Arduino-based microcontroller platform. A predefined target melting temperature for the thermoplastic material was programmed into the firmware, and temperature regulation was achieved via a relay-driven on/off control loop. A thermocouple (or RTD sensor) interfaced with the microcontroller provided real-time temperature feedback.

In the current configuration, only the nozzle (hot-end) temperature was actively monitored and regulated. The barrel or melt zone temperature was not independently controlled, resulting in suboptimal thermal consistency and potential premature solidification or excessive viscosity variation during extrusion.

A more robust thermal management strategy would involve implementing a multi-zone heating system. This would include closed-loop control of the primary melting zone using PID (Proportional-Integral-Derivative) control algorithms to maintain the material at its optimal melting temperature, followed by a secondary controlled temperature increase at the nozzle to ensure proper melt fluidity and reduce backpressure during extrusion. This improvement was identified but deferred for future system revisions.
{% include elements/figure.html image="/assets/img/1-4.png" caption="Schematics"  %}
The exact code used for this project can be found [here.](https://github.com/yashraw/Codes/blob/main/Injection%20Moulding.ino) <br>

<p>&nbsp;</p>
#### Working Principle (Technical Description)

The process begins with the collection of waste thermoplastic materials, which are mechanically shredded using a granulator into uniform particles known as plastic granules or feedstock. These granules are then introduced into a hopper that serves as the material storage and feeding unit. Gravity or a screw feeder mechanism directs the granules from the hopper into the heating barrel.

The barrel is equipped with electric band heaters arranged in multiple heating zones to ensure gradual and uniform melting of the polymer. A thermocouple or RTD sensor continuously monitors the barrel temperature, which is regulated using an Arduino-based control system, typically employing PID (Proportional-Integral-Derivative) or relay-based on/off control. The plastic transitions from a solid to a viscoelastic molten state as it moves through the heated barrel. In advanced systems, a rotating screw would provide shear heating and material conveyance, but in this setup, the plastic melt is manually transferred.

Once sufficient melt is accumulated, pressure is manually applied using a hand-operated lever mechanism, functioning similarly to a plunger-type injection system. This pressure forces the molten plastic into the mold cavity through the nozzle and sprue channel. The mold, typically made of aluminum or mild steel, defines the final geometry of the part and may be fitted with cooling channels or fins to facilitate controlled solidification.

After the molten plastic fills the mold, it is allowed to cool and solidify under ambient or forced cooling conditions. The molded component is then ejected.

Depending on the material properties, mold surface quality, and dimensional tolerances required, the part may undergo post-processing. This may include flash removal, CNC machining, drilling, surface finishing (sanding or grinding), painting, or coating to enhance aesthetics and functionality.


Below is the result of the first mould created and the outcome during the initial trial of this machine.
{% include elements/figure.html image="/assets/img/1-3.jpg" caption= "Tile - Trial 1" %}

<p>&nbsp;</p>
#### Result Analysis

There were numerous difficulty faced and things didn’t go as planned.
1. The heating was not stable, temperature oscillated between 10-25C from desired.
2. Plastic used to stick to the edges of barrel and plunger.
3. The mould we made was not totally precisely manufactured.<br>

But overall the product came out was satisfactory.

{% include elements/figure.html image="/assets/img/1-6.jpg" caption= "TILES"%}

In the above picture, we used LDPE plastic which has melting temperature of 160C, this was collected
from a used water tank and chopped into granules. The surface finish we got was quite good, but
could be improved further by using various surface finishing methods and using a very precisely
manufactured mould.

<p>&nbsp;</p>
#### Advantages and Applications

**1) Fast production and highly efficient.** Injection moulding can produce an incredible amount of parts per hour. Speed depends on the complexity and size of the mould. 

**2) Low labour and operation costs.** As it is build with affordable materials, anyone can built this machine utilising ours as a template. It doesn't requires much technical knowledge except knowing melting points of various plastics.

**3) Design flexibility.**  The moulds themselves are subjected to extremely high pressure. As a result, the plastic within the moulds is pressed harder and allows for a large amount of detail to be imprinted onto the part and for complex or intricate shapes to be manufactured. Also did I forget that these are interchangeable? 


**4) Large material choice.**  Depending on the plastics collected, the material choice is abundant and cheap.

This machine being **very affordable** gives a chance to small business owners as well as established workshop to implement the idea of recycling plastics and reduce the wastage. This will not only **reduce the wastage** caused by improper disposing of plastic materials and helps in **reducing the pollution**. It's **low maintenance** and is easy to repair and replace if anything goes wrong. The product itself is made out of  household and easily available parts so it’s not hard to build your own machine.

The 3D MODEL(.iges) file can be found [here.](https://drive.google.com/file/d/1gruXpl3QTFOZgCFDQEDkuUstFQxakams/view?usp=sharing)
