## Running City4CFD
The files here can be used to run City4CFD once installed. It can be run by calling `path-to-executable/City4CFD test_case_config.json`

## Preparing geometry for surfer
After running City4CFD, the outputted STLs must be rotated so that +Y points up and +X points along the domain. This can be done with two rotations: Z axis rotation of -70° and then +X axis rotation of -90°.