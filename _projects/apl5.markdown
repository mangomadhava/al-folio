---
layout: page
title: Terrain Relative Navigation
description: Analysis of flight software for NASA's dragonfly mission to Titan.
img: /assets/img/dragonfly-on-surface.jpeg
importance: 1
category: Applied Physics Lab
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/dragonfly-on-surface.jpeg' | relative_url }}" alt="" title="example image"/>
    </div>
</div>
<h5 id="skills-used-">The Mission:</h5>
<a href="https://dragonfly.jhuapl.edu/"> Dragonfly </a> is a NASA Mission that will put a drone on the largest moon of Saturn, Titan, in the 2030's. It is set for launch in 2027, and will investigate the atmosphere and geology of <a href="https://dragonfly.jhuapl.edu/Why-Titan/"> Titan</a>. Being millions of miles from Earth, it requires autonomous flight based on its camera and sensors.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/dragonfly-landing.png' | relative_url }}" alt="" title="example image"/>
    </div>
</div>

<h5 id="skills-used-">Terrain Relative Navigation</h5>
Dragonfly's terrain relative navigation team uses fast fourier transforms and other methods to match features between subsequent images to accurately determine position and velocity. I analyzed the flight algorithm for errors that could occur due to camera and sensor miscalibration, which could occur due to the harsh conditions during liftoff and while on Titan. I designed a study to test for possible miscalibration that could occur and delivered results that helped guide the development for the final version of their algorithm to accurately navigate the drone on Titan.


I also helped compare the current flight algorithm being developed to other state of the art feature detection algorithms. This information is needed to explain to NASA the reasons for choosing a specific algorithm.

<h5 id="skills-used-">Skills used:</h5>
<li>Matlab Software Development</li>
<li>Analysis and comparison of errors</li>
<li>Source and version control on an active repository</li>
<li>Gained knowledge of camera parameters, ex. camera principle distance, focal distance, intrinsic/extrinsic matrix, and distortion correction</li>


Summer 2021.
