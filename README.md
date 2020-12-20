# Fantasy Map Generator (Work in Progress)

This program is an implementation of a fantasy map generator based on the methods described in Martin O'Leary's "Generating fantasy map" notes (https://mewo2.com/notes/terrain/).

# Progress

## Generating Irregular Grids

A Poisson Disc Sampler generates a set of random points with the property that no two points are within a set radius of eachother.
![alt tag](http://rlguy.com/map_generation/images/uniform_vs_poisson_sampling.jpg)

The set of points are triangulated in a Delaunay triangulation. The triangulation is stored in a doubly connected edge list (DCEL) data structure.
![alt tag](http://rlguy.com/map_generation/images/uniform_vs_poisson_delaunay.jpg)

The dual of the Delaunay triangulation is computed to produce a Voronoi diagram, which is also stored as a DCEL.
![alt tag](http://rlguy.com/map_generation/images/uniform_vs_poisson_voronoi.jpg)

Each vertex in the Delaunay triangulation becomes a face in the Voronoi diagram, and each triangle in the Delaunay triangulation becomes a vertex in the Voronoi diagram. A triangle is transformed into a vertex by fitting a circle to the three triangle vertices and setting the circle's center as the position of a Voronoi vertex. The following image displays the relationship between a Delaunay triangulation and a Voronoi diagram.

![alt tag](http://rlguy.com/map_generation/images/voronoi_delaunay_overlay.jpg)

The vertices in the Voronoi diagram will be used as the nodes in an irregular grid. Note that each node has exactly three neighbours.

## Generating Terrain

An initial height map is generated using a set of primitives:
- addHill - Create a rounded hill where height falls off smoothly
- addCone - Create a cone where height falls off linearly
- addSlope - Create a slope gradient that runs parallel to a line

and a set of operations:
- normalize - Normalize the height map values to [0,1]
- round - Round height map features by normalizing and taking the square root of the height values
- relax - Replace height values with the average of their neighbours
- setSeaLevel - Translate the height map so that the new sea level is at zero

![alt tag](http://rlguy.com/map_generation/images/heightmap_primitives.jpg)

Contour lines are generated from the Voronoi edges. If a countour line is generated for some elevation h, a Voronoi edge will be included in the countour if one of its adjacent faces has a height less than h while the other has a height greater than or equal to h.

![alt tag](http://rlguy.com/map_generation/images/heightmap_contour.jpg)