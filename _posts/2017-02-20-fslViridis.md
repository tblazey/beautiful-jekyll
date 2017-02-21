---
layout: post
title: Matplotlib Colormaps for FSL
category: Visualization
tags: [FSL, Python, matplotlib]
---

I think that anyone that uses the amazingly useful [FSL](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki) package is fairly familar with the image viewer [fslview](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FslView). I use this viewer pretty much everyday, and overall I think it works quite well. It loads images quickly and has a number of [useful features](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FslView/UserGuide). However, one area that it is a bit lacking is in its colormap selection. By default it doesn't really have a great continuous colormap, and the ones hidden deep within its directory structure just aren't that [great]([http://ieeexplore.ieee.org/document/4118486/). 

There is an easy fix though. [Matplotlib](http://matplotlib.org/) has a number of great [colormaps](http://matplotlib.org/examples/color/colormaps_reference.html). In my opinion the best are their perceptually uniform colormaps viridis, plasma, magma, and inferno. It turns out that it is pretty easy to get these colormaps into a format that fslview can read. So easy, in fact, that I wrote a Python script to do it. You can find the script, as well as the four colormaps, [here](https://github.com/tblazey/fslViridis).

To use these colormaps you have two main options. The fastest option is to load it directly from the command line using the -l option:

{% highlight tcsh %}
fslview /usr/local/fsl/data/standard/MNI152_T1_2mm_brain -l plasma.lut
{% endhighlight %}

Alternatively, you can load the colormap inside FSLview by clicking on the little folder icon in the overlay information dialog. Either way, you get the following result:

![fslPlasma]({{ site.baseurl }}/img/fslPlasma.png "Plasma Colormap in FSL")

Pretty right? And if for some reason you don't like any of the colormaps I made, you can use the provided script to make a FSL lut out of any of the colormaps included in Matplotlib.  
