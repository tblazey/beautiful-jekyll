---
layout: post
title: Viewing FreeSurfer Surfaces in R with RGL
category: R
tags: [R, FreeSurfer,rgl]
---

As part of a recent project, I needed to view [FreeSurfer](https://surfer.nmr.mgh.harvard.edu/) cortical surfaces within R.  I tried various packages, but ended up using the [rgl](https://cran.r-project.org/web/packages/rgl/index.html) package. The whole thing is fairly simple, but there are few little tricks that are needed to get everything working.

The first thing you need to do is to get your FreeSurfer surface into a format that R can read. I chose to convert the FreeSurfer surface into ASCII format using FreeSurfer's [mris_convert](https://surfer.nmr.mgh.harvard.edu/fswiki/mris_convert):

{% highlight Tcsh %}
mris_convert lh.pial lh.pial.asc
{% endhighlight %}

This commands converts the left hemisphere pial surface into ASCII. For this example, I used the lh.pial file of FreeSurfer's average surface, fsaverage. The first few lines of the file look like this

{% highlight Plain Text %}
#!ascii version of lh.pial
163842 327680
-38.834549  -19.019356  66.908409  0
-16.554127  -69.255852  60.977936  0
{% endhighlight %}

The second line states the number of vertices and then the number of faces. Each vertex and each face gets is own line, with all the vertices coming before the first face. So for this example, the vertices are contained in lines 3 to 163,844, and the faces from lines 163,845 to 491,524. Each vertex row is simply a list of x,y,z coordinates whearas each face row contains the vertex indices that form that face (first index is zero). To load in the data into R, I wrote the following little function:

{% highlight R %}
loadSurf = function(surfPath){
  
  #Get the number of vertices and faces
  surfInfo = read.table(surfPath,skip=1,nrows=1,col.names=c("nVertex","nFace"))
  
  #Get the vertex data
  surfVertex = read.table(surfPath,skip=2,nrows=surfInfo$nVertex,
                          colClasses = c(rep("numeric", 3),"NULL"))
  colnames(surfVertex) = c("X","Y","Z")
  
  #Get the face data
  surfFace = read.table(surfPath,skip=2+surfInfo$nVertex,nrows=surfInfo$nFace,
                        colClasses = c(rep("integer", 3),"NULL")) + 1
  colnames(surfFace) = c("Vertex.1","Vertex.2","Vertex.3")
  
  #Return vertices and faces
  return(list(vertex=t(surfVertex),face=t(surfFace)))

}
{% endhighlight %}

This function returns a list with an element for the faces and a element for the vertices. With this we have what we need to load the surface using RGL. 

{% highlight R %}
#Load in surface
surfPath = "./lh.pial.asc"
surfData = loadSurf(surfPath)

#Make surface mesh
library(rgl)
surfaceMesh = tmesh3d(surfData$vertex,surfData$face,homogeneous=FALSE)

#Show the surface
shade3d(surfaceMesh,col="#7E7E7E")
{% endhighlight %}

The resulting surface image is pretty basic, and doesn't really display any information:

![surface]({{ site.baseurl }}/img/surfaceOne_cropped.png "Basic RGL Surface")

We can, however, easily add overlays. Lets test this out with the average thickness map (lh.thickness). First, we need to convert the thickness file and load it into R. I chose to convert to ASCII, but you could use another format:

{% highlight Tcsh %}
mris_convert -c lh.thickness lh.pial lh.thickiness.asc
{% endhighlight %}

This creates a four-column file where the columsn are index, vertex x-coordinate, vertex y-coordinate, vertex z-coordinate, and thickness value. As the file contains the same number of vertices in the same order as lh.pial, we can just ignore everything but the fifth column:

{% highlight R %}
#Load in thickness data
thickPath = './lh.thickness.asc'
thickData = read.table(thickPath,skip=0,
                       colClasses = c(rep("NULL", 4),"numeric"))$V5
{% endhighlight %}

Now we need to map the thickness values to a colormap. A nice option is the [viridis colormap](https://cran.r-project.org/web/packages/viridis/vignettes/intro-to-viridis.html):

{% highlight R %}
#Get a viridis colormap with 255 colors
library(viridis)
nColors = 255
colors = viridis(nColors)

#Set the limits of the colormap to 1 and 3. 
#Data above three will be set to the last color and below 1 to gray
colorIdx = approx(seq(from=1,to=3,length.out=nColors),1:nColors,
                  thickData,rule=2,method="constant")$y
thickColors = colors[colorIdx]
thickColors[thickData<1] = "#7E7E7E"

#Get a color for all the vertices using face indicies
thickColors = thickColors[as.vector(surfData$face)]

#Show colored surface
shade3d(surfaceMesh,col=thickColors)
{% endhighlight %}

![thickness]({{ site.baseurl }}/images/surfaceTwo_cropped.png "RGL Surface with Thickness Overlay")

The same process can be used to load any standard surface overlay. While this might not be a fast as [other](https://surfer.nmr.mgh.harvard.edu/fswiki/TkSurfer) [options](https://surfer.nmr.mgh.harvard.edu/fswiki/FreeviewGuide/FreeviewIntroduction), it is pretty flexible and works right within R. Questions? Feel free to [ask](https://tblazey.github.io/about).
