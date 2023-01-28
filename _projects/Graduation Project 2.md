---
name: Project 2
tools: [ArduinoIDE, Automation]
image: https://images.unsplash.com/photo-1565451451500-4fa5c8c3f456?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=817&q=80
description: Remote Accessed Irrigation System
#external_url: https://www.google.com
---


<!--
https://youtu.be/ycKPtAkAFgk -> Embed this video!!
-->


## Remote Accessed Irrigation System

<!--
{% include elements/figure.html image="https://images.unsplash.com/photo-1611778030003-b681014615ff?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1955&q=30" caption="sample caption" %}
-->

This Project was inspired by the current trend in automation in every sector, one such ignored section is Irrigation, the aim was to create a solution that is very flexible in use. A person having few plants for his/her garden to a farmer having a field to irrigate, anyone can implement this.

<p>&nbsp;</p>
#### Objectives

- The main aim of the project is to create a baseline automated irrigation system which can be used to irrigate a single plant but can be scaled up for the usage of any number of plants.
- The parts used here are relatively inexpensive in terms of the value it provides, Total approximate cost we incur was less than ₹1500.

The schematics of the project is shown below.
{% include elements/figure.html image="/assets/img/2-1.png" caption="Schematics
" %}
The brain is the Arduino microcontroller, It takes the moisture data provided by Soil Moisture Sensor and will give instruction to the Pump to turn On or Off, this turning was controlled by Relay, and depending on the pump a person is going to use, this relay needs to be changed according to its amperage rating.

The exact code used for this project can be found [here.](https://github.com/yashraw/Codes/blob/main/Remote%20Irrigation.ino) <br>

<p>&nbsp;</p>
#### Working

The moisture sensor is placed in a field, in our case, inside a plot which constantly measures moisture level and sends that data to Arduino, it then compares to known dataset of moisture level required for a particular plant and decides if the pump should be turned On or not.

The future scop of this project would be to integrate with the Weather Forecast to predict if it's going to rain or not. Additionally, Force remote trigerring via Mobile application can be done that can also provide live telemetry to user and which can be used to analyse further.



<p>&nbsp;</p>
#### Result Analysis

1. The output of the code was binary, i.e. On and Off only, more fine tuning can be done.
2. There were so many ideas to integrate, but because of time constrained, it couldn't be done.
<br>

But overall proof of concept was achieved.


<p>&nbsp;</p>
#### Advantages and Applications

**1) Easily Scalable.** A hobbist gardener can also use this and a fully commercial farms can also implement this.

**2) Easy to understand and operate.** Very basic coding skill is required to achieve this project, and if anyone gets stuck somewhere, numerous online forums are ready to help as Arduino is widely used microcontroller.

**3) Design flexibility.**  A person that already has some kind of irrigation system in place can integrate this on top of that which basically means they don't have to start from ground zero in order to implement this.

<!--
{% include elements/video.html id="" %}
-->