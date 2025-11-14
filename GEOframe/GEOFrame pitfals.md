---
owner: "Daniele Andreis"
created: 2025-11-14 18:43
updated: 2025-11-14
framework: GEOframe
tags: [geoframe]
---

# OMS console related problem

- file not found
- class not found
- Not well formatting csv. If you open the file in Excel and then saved it as csv paid attention to the format.
    
    ![[photo_5841539484502572807_y.jpg]]
    
    from Sohaib (telegram group)
    

# Kriging related problem

- singular matrix check your shapefile, maybe you have two station with same location.

## EVOTRANSPIRATION

- not good output check if subbasins centroid lies into the polygons, if the centroid is outside or you have to change the raster input (the poiny should be indide the raste) or you have to move the centroid. An example of chcek script are available: [[https://gist.github.com/bubbobne/5a20c1273e671f12dbcd099112093888]]  it move the centroid on the middle of the network.