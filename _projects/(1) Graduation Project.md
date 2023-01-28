---
name: Project-1
tools: [SolidWorks, ArduinoIDE, Designing, Automation]
image: https://images.unsplash.com/photo-1595278069441-2cf29f8005a4?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=80
description: Recycling of Plastics Using Affordable Injection Moulding Machine
#external_url: https://www.google.com
---


## Recycling of Plastics Using Injection Moulding Machine

<!--
{% include elements/figure.html image="https://images.unsplash.com/photo-1611778030003-b681014615ff?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1955&q=30" caption="sample caption" %}
-->

In present scenario, production, use and disposal of plastics and its products are increasing, which is harmful to society. This improper disposing of plastics has been a major concern since a long time. They tend to disrupt the environment in a very serious way. They break into the soil creating smaller plastic pieces and release toxic chemicals, eventually eaten by the animals and therefore harming them. Plastic creates a whole lot of damage to the earth and people, some of which includes wildlife harm, clogged sewage systems.

We made an affordable Injection Moulding machine which will take recyclable plastics as raw material and will melt it depending upon the melting temperatur and will be injected out into a mould for the product to be made.

<p>&nbsp;</p>
#### Objectives

- The main aim of the project is to make an affordable injection moulding machine out of readily available and cheap parts. This will not only reduce the amount of waste plastic but will also create new products.
- This machine can be an excellent part in the “SWACCH BHARAT” initiative taken by out PMO India, as it will reduce the plastic waste generated

The project started by a quick handrawn sketch followed by a quick CAD model in SolidWorks.  
{% include elements/figure.html image="/assets/img/1-2.png" caption="First Sketch" %}
Now we didn't made the entire machine exactly as the cad file because The plan was to built  with scrap parts, therefore we headed towards local scarpyards for scouting junk that would be usefull to us.
We got most of the parts from there at reasonable cost, around Rs.55 per Kg. Rest of the materials were purchased new and some components like Band Heaters and Thermocouple were also purchased new.
{% include elements/figure.html image="/assets/img/1-2.png"  %}


Now the whole heating element was controlled by the Arduino microcontroller, basically a desired and known melting temperature of the material to be inserted was inputted in the code and with the help of relay arrangement, the heating of the barrel was controlled, We only controlled the exit nozzle temperature and this can be improved by controlling the melting temperature first and increasing temperature right before ejecting out the nozzle to maintain optimal fluidity, but it was left aside for another revision.
{% include elements/figure.html image="/assets/img/1-4.png" caption="Schematics"  %}
The exact code used for this project can be found [here.](https://github.com/yashraw/Codes/blob/main/Injection%20Moulding.ino) <br>

<p>&nbsp;</p>
#### Working

Initially the waste plastic is collected and are chopped down into finer pieces, this is called granules. This granule is then fed into the hopper, this acts as a storage area, this are then transferred into the heating barrel which melts the plastic. The barrel is provided with band heaters. The molten plastic is fed into a mould using pressure provided with a hand-lever mechanism. The molten plastic takes shape of the mould and it is allowed to be cooled down. <br>
The product formed may or may not require further machining depending upon the type of material used and/or the design of the mould. This post-process treatment can be anything from surface finishing, grinding, painting etc.



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
