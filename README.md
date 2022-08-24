# Meshing a City4CFD domain in CharLES

A simple section of downtown Seattle with 9 buildings was exported to STLs using [City4CFD](https://github.com/tudelft3d/City4CFD/). The program exports a planar STL for the 5 sides excluding the ground, a 3D ground STL, and an STL with the building geometries. For details on generating these files, refer to the folder [City4CFD_input](City4CFD_input/README.md).

I was not able to mesh the domain without generating errors in stitch. For all attempts, stitch invariably failed at generating the mesh in the step `NSMOOTH 0`. 

## Description of Geometry files
In addition to attempting to mesh the raw STLs as outputted by City4CFD, which can be found in [Geometry/Original](Geometry/Original), I attempted to mesh after the following changes to the STLs.

### Original STLs
Trying to mesh the original (except for rotation) STLs yields the following error in stitch:
```bash
processParam "NSMOOTH 0"
NSMOOTH set to 0
ensurePointsNearPlane: building ALL mesh points
calcBbminmax(): partVec.size(): 1
SurfaceShm::calcBbminmax: -381.60013:308.40082 11.397:200 -246.73526:213.26537
 [ -381.60013 11.397 -246.73526 ] [ -308.40082 -200 -213.26537 ]
 > HCP_X0 set to default: -36.599655 105.6985 -16.734947
 > HCP_DX0 set to default: 0.0001 0.0001 0.0001
 > applying 0 HCP_WINDOW(s)
 > setting surface_st_flag to FAZONE window index...
 > level_max: 0, nwi: 1
 setup time: 0.0022721291 [s], total time so far: 0.0022721291 [s]
 > full bbox for level: 0 -94.3016:94.3014 -230.00041:230.00021
 > delta for this level: 0.8660254
 bbox time: 0.0015649796 [s], total time so far: 0.0038371086 [s]
working on level 0...
 > cart ray array size: 217 531
 > nray cart: 115227 nray: 115227 ratio: 1
 > level 0 nray_final: 115228 avg/max per rank: 115228 115228
 > surface ray intersection time: 0.10369992 [s], total time so far: 0.10753703 [s]
 > in/out points found: 0, per rank avg/max: 0 0
 > in/out exchanges: 2
 > tris flagged at or above this level: 0 tris passing signed distance check: 0 fraction: -nan
 > bloated tri time: 0.066517115 [s], total time so far: 0.17405415 [s]
 > hcp point count: 6
 > hcp point distribution per level:
 > level: 0 count: 6 (100%)
writeHcpPointsToPartData: hcp point count: 6
setPoints()
prepareIsInside()
 > done prepareIsInside()
 > ncv_global: 6
MPI_Sync: rebuildVd.....................time since last sync:    0.1792202, cumulative:   0.19760799
StitchNS::rebuildVd(): nsmooth: 0
=====================================================
first Voronoi build: 
=====================================================
rebuildFlaggedVds(), b_surface: 1 final: 1
 > iter: 1
 > dumpRange: RC, 132:132
   > build vd stats: count=6 nnbr/vd (avg:max): 0 0
   > vd build timing: count: 6 avg: 0.00036569436 max: 0.0015821457
   > vd failed seed and no nbrs: 6 vd failed seed: 0 vd failed delta too small: 0 vd passed: 0 sum: 6
 > iter: 2
 > dumpRange: RC, 140:140
   > build vd stats: count=6 nnbr/vd (avg:max): 0.66666667 2
   > vd build timing: count: 6 avg: 0.00018616517 max: 0.00026702881
   > vd failed seed and no nbrs: 3 vd failed seed: 3 vd failed delta too small: 0 vd passed: 0 sum: 6
 > iter: 3
 > dumpRange: RC, 156:156
   > build vd stats: count=6 nnbr/vd (avg:max): 1 2
FAILED CuttableVoronoiData::check(): ned,nno problem: 0 0
FAILED cvd.buildSeed at x_vd: [ -239.59956 37.282593 -12.40472 ] rank: 0
writeTeclot: 0 x0: [ 0 0 0 ]
catch -1 on rank: 0: calling abort...
application called MPI_Abort(comm=0x84000000, -1) - process 0
[unset]: readline failed
```

### Translating down and intersecting the buildings in Surfer
The lines in the surfer .in file were added:
```bash
TRANSLATE DX 0 -0.01 0 ZONE_NAMES buildings  # translate buildings down so that it overlaps ground
REFINE_TRIS ZONE_NAMES ground DELTA 0.5  # refine intersection between building and ground
INTERSECT ZONE_NAMES ground buildings MODE SUBTRACT  # delete the part of the building below the domain - MAY be ADD instead of SUBTRACT
```
however surfer gives the error:
```bash
processParam "INTERSECT ZONE_NAMES ground buildings MODE SUBTRACT"
 > got mode: SUBTRACT
SimpleSurface::intersectFlaggedSubzones: mode: SUBTRACT
 > flagged tri count:
   > nst0: 8441494 bbox: [ -381.60013 11.397 -246.73526 ] : [ 308.40082 44.220001 213.26537 ]
   > nst1: 3268 bbox: [ -289.48465 28.664999 -119.77295 ] : [ -0.58840489 115.18 123.2514 ]
 > got nfa0,nfa1: 1960626 2463 facePairVec.size(): 4299541
========================= Working on =======================
p0: 1 0 e00 [ 0 1 1 ]
p0: 0 1 e01 [ 0 -1 -1 ]
p0: 1 1 e02 [ 0 0 0 ]
p1: 0 0 e10 [ 0 0 0 ]
p1: 0 -1 e11 [ 1 -1 -1 ]
p1: -1 0 e12 [ 1 -1 -1 ]
WARNING: value: 29798735 was not found: triTriIntVec.size(): 2
p0: 1 0 e00 [ 0 1 1 ]
p0: 0 1 e01 [ 0 -1 -1 ]
p0: 1 1 e02 [ 0 0 0 ]
p1: 0 0 e10 [ 0 0 0 ]
p1: 0 -1 e11 [ 1 -1 -1 ]
p1: -1 0 e12 [ 1 -1 -1 ]
triTriIntVec.size(): 2
 >  NODE_EDGE_INT 1 0 0.25
 >  NODE_EDGE_INT 1 0 0.25
GOT DUPLICATE triTriIntVec: 
 > ii:  NODE_EDGE_INT 1 0 0.25
 > jj:  NODE_EDGE_INT 1 0 0.25
triTriIntVec[ii].ddata[0]: 0.25 triTriIntVec[jj].ddata[0]: 0.25 diff: 2.7755576e-17
ONE INTERSECTION
CODE:
CODE: case 29798735:
CODE:   idata[0] = NODE_EDGE_INT;
CODE:   idata[1] = 1; // node0 index on tri0
CODE:   idata[2] = 0; // edge1 index on tri1
CODE:   idata[3] = 0; // unused
CODE:   idata[4] = 0; // unused
CODE:   ddata[0] = e0d[0][2]/(e0d[0][1]+e0d[0][2]); // edge1 wgt
CODE:   ddata[1] = 0.0; // unused
CODE:   ddata[2] = 0.0; // unused
CODE:   ddata[3] = 0.0; // unused
CODE:   return 1;
take a look!
 > writing ASCII tri file: tri0.000000.dat
 > writing ASCII tri file: tri1.000000.dat
Assertion failed: (0), function intersectFlaggedSubzones, file SimpleSurface_intersect.cpp, line 189.
[1]    41292 abort      nextgen-master/bin/surfer.exe -i City4CFD_CharLES_test_case/surfer.in
```

### Removing internal faces in buildings STL
I found that the buildings STL exported by City4CFD contains internal faces:
<img src="https://we-github-img.s3.us-east-2.amazonaws.com/int_faces.png" alt="showing internal faces in STL" width=400>

I removed these in Rhino and exported the new STL as [Geometry/Processed/buildings_intfacesremoved.stl](Geometry/Processed/buildings_intfacesremoved.stl). Running stitch with this STL and all the others kept the same yields the following error:
```bash
processParam "NSMOOTH 0"
NSMOOTH set to 0
ensurePointsNearPlane: building ALL mesh points
calcBbminmax(): partVec.size(): 1
SurfaceShm::calcBbminmax: -381.60013:308.40082 11.397:200 -246.73526:213.26537
 [ -381.60013 11.397 -246.73526 ] [ -308.40082 -200 -213.26537 ]
 > HCP_X0 set to default: -36.599655 105.6985 -16.734947
 > HCP_DX0 set to default: 0.0001 0.0001 0.0001
 > applying 0 HCP_WINDOW(s)
 > setting surface_st_flag to FAZONE window index...
 > level_max: 0, nwi: 1
 setup time: 0.0023708344 [s], total time so far: 0.0023708344 [s]
 > full bbox for level: 0 -94.3016:94.3014 -230.00041:230.00021
 > delta for this level: 0.8660254
 bbox time: 0.001502037 [s], total time so far: 0.0038728714 [s]
working on level 0...
 > cart ray array size: 217 531
 > nray cart: 115227 nray: 115227 ratio: 1
 > level 0 nray_final: 115228 avg/max per rank: 115228 115228
 > surface ray intersection time: 0.10563207 [s], total time so far: 0.10950494 [s]
 > in/out points found: 0, per rank avg/max: 0 0
 > in/out exchanges: 2
 > tris flagged at or above this level: 0 tris passing signed distance check: 0 fraction: -nan
 > bloated tri time: 0.068284988 [s], total time so far: 0.17778993 [s]
 > hcp point count: 6
 > hcp point distribution per level:
 > level: 0 count: 6 (100%)
writeHcpPointsToPartData: hcp point count: 6
setPoints()
prepareIsInside()
 > done prepareIsInside()
 > ncv_global: 6
MPI_Sync: rebuildVd.....................time since last sync:   0.18273807, cumulative:   0.20089412
StitchNS::rebuildVd(): nsmooth: 0
=====================================================
first Voronoi build: 
=====================================================
rebuildFlaggedVds(), b_surface: 1 final: 1
 > iter: 1
 > dumpRange: RC, 132:132
   > build vd stats: count=6 nnbr/vd (avg:max): 0 0
   > vd build timing: count: 6 avg: 0.00036001205 max: 0.0014870167
   > vd failed seed and no nbrs: 6 vd failed seed: 0 vd failed delta too small: 0 vd passed: 0 sum: 6
 > iter: 2
 > dumpRange: RC, 140:140
   > build vd stats: count=6 nnbr/vd (avg:max): 0.66666667 2
   > vd build timing: count: 6 avg: 0.0002036492 max: 0.00032997131
   > vd failed seed and no nbrs: 3 vd failed seed: 3 vd failed delta too small: 0 vd passed: 0 sum: 6
 > iter: 3
 > dumpRange: RC, 156:156
   > build vd stats: count=6 nnbr/vd (avg:max): 1 2
FAILED CuttableVoronoiData::check(): ned,nno problem: 0 0
FAILED cvd.buildSeed at x_vd: [ -239.59956 37.282593 -12.40472 ] rank: 0
writeTeclot: 0 x0: [ 0 0 0 ]
catch -1 on rank: 0: calling abort...
application called MPI_Abort(comm=0x84000000, -1) - process 0
[unset]: readline failed
```

### Making the ground a 3D object
Guessing that stitch may be struggling with an open STL, I closed the ground by adding a bottom and sidewalls:
<img src="https://we-github-img.s3.us-east-2.amazonaws.com/3d_ground.png" alt="showing thickened ground to make it a 3D object" width=700>

Running stitch with this yielded the following error:
```bash
processParam "NSMOOTH 0"
NSMOOTH set to 0
ensurePointsNearPlane: building ALL mesh points
calcBbminmax(): partVec.size(): 1
SurfaceShm::calcBbminmax: -381.60013:308.40082 0:200 -246.73526:213.26537
 [ -381.60013 0 -246.73526 ] [ -308.40082 -200 -213.26537 ]
 > HCP_X0 set to default: -36.599655 100 -16.734947
 > HCP_DX0 set to default: 0.0001 0.0001 0.0001
 > applying 0 HCP_WINDOW(s)
 > setting surface_st_flag to FAZONE window index...
 > level_max: 0, nwi: 1
 setup time: 0.0021560192 [s], total time so far: 0.0021560192 [s]
 > full bbox for level: 0 -100.0001:99.9999 -230.00041:230.00021
 > delta for this level: 0.8660254
 bbox time: 0.0015301704 [s], total time so far: 0.0036861897 [s]
working on level 0...
 > cart ray array size: 231 531
 > nray cart: 122661 nray: 122661 ratio: 1
 > level 0 nray_final: 122662 avg/max per rank: 122662 122662
 > surface ray intersection time: 0.11419702 [s], total time so far: 0.11788321 [s]
ERROR: mpi_rank: 0  debug_count: 18477
stitch.exe: HcpPointBuilder.hpp:3638: void HcpPointBuilder::ensurePoints(const double *, const double *): Assertion `iter->level > 0' failed.
Aborted
```

### Repairing the edited STLs in 3D Builder
Microsoft's 3D Builder has a STL repair tool. I tried using this to repair the aforementioned edited STLs, [ground_3D.stl](Geometry/Processed/ground_3D.stl) and [buildings_intfacesremoved.stl](Geometry/Processed/buildings_intfacesremoved.stl) so that mesh edges matched. The repaired files have the same names with the suffix `_repaired`. Trying to run stitch with these files yielded the error:
```bash
processParam "NSMOOTH 0"
NSMOOTH set to 0
ensurePointsNearPlane: building ALL mesh points
calcBbminmax(): partVec.size(): 1
SurfaceShm::calcBbminmax: -381.60785:308.40082 0:200 -246.74338:213.26926
 [ -381.60785 0 -246.74338 ] [ -308.40082 -200 -213.26926 ]
 > HCP_X0 set to default: -36.603516 100 -16.737061
 > HCP_DX0 set to default: 0.0001 0.0001 0.0001
 > applying 0 HCP_WINDOW(s)
 > setting surface_st_flag to FAZONE window index...
 > level_max: 0, nwi: 1
 setup time: 0.0031490326 [s], total time so far: 0.0031490326 [s]
 > full bbox for level: 0 -100.0001:99.9999 -230.00642:230.00622
 > delta for this level: 0.8660254
 bbox time: 0.0018188953 [s], total time so far: 0.0049679279 [s]
working on level 0...
 > cart ray array size: 231 531
 > nray cart: 122661 nray: 122661 ratio: 1
 > level 0 nray_final: 122662 avg/max per rank: 122662 122662
 > surface ray intersection time: 0.10640407 [s], total time so far: 0.11137199 [s]
ERROR: mpi_rank: 0  debug_count: 18477
stitch.exe: HcpPointBuilder.hpp:3638: void HcpPointBuilder::ensurePoints(const double *, const double *): Assertion `iter->level > 0' failed.
Aborted
``` 

