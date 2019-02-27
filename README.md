# OpenFOAM_SALOME_Propeller_example
An example for using Salome unv file meshing with OpenFOAM.
****************************************************************
First watch:

SALOME & OpenFOAM Tutorial: Propeller - Preparing The Geometry
https://www.youtube.com/watch?v=rqrCqzuvxPY

SALOME & OpenFOAM Tutorial: Propeller - Preparing The Mesh
https://www.youtube.com/watch?v=z8J6euEVCvg

SALOME & OpenFOAM Tutorial: Propeller - Folder Case Setup And Run
https://www.youtube.com/watch?v=F64G75nrMhE

SALOME & OpenFOAM Tutorial: Propeller - Postprocessing and Animation
https://www.youtube.com/watch?v=m81uFAHJP9Y

****************************************************************

This was tested with an AMD64 processor (32 gigs RAM)
on Xubuntu 18.04 LTS with OpenFOAM v18.06.
The following steps use the mesh already contained in this
repo and starts with exporting the unv files.

Open "rotor and stator.hdf" (found in CAD folder) in Salome.
Export stator mesh as stator.unv into prop/stator
Export rotor mesh as rotor.unv into prop/rotor

start command line in prop/

follow the steps

	$ cd rotor 
	/rotor$ ideasUnvToFoam rotor.unv
	$ cd ../stator 
	/stator$ ideasUnvToFoam stator.unv
	/stator$ cd ../ 
	$ mergeMeshes -overwrite stator rotor
	$ cd rotor 
	/rotor$ checkMesh
	$ cd ../stator 
	/stator$ checkMesh
	/stator$ setSet
	Command>cellZoneSet rotor new setToCellZone region1
	Command>quit
	/stator$ topoSet

In your stator/constant/polyMesh/boundary file make sure AMI patches look like

	AMI2
    {
        type            cyclicAMI;//This needs to be changed.
		neighbourPatch  AMI1;//This needs to be added.
        nFaces          9774;
        startFace       4088405;
    }
    AMI1
    {
        type            cyclicAMI;//This needs to be changed.
		neighbourPatch  AMI2;//This needs to be added.
        nFaces          10160;
        startFace       4098179;
    }

	/stator$ decomposePar
	/stator$ mpiexec -np 2 renumberMesh -overwrite -parallel

In your parallel processor files ( e.g. processor1/constant/polyMesh/boundary)
the polyMesh boundary files need to look like this for patches Side and propeller.

Side
{
	type            wall;//this will cause an error if it's not wall
	nFaces          21884;
	startFace       2003703;
}


propeller
{
	type            wall;//this will cause an error if it's not wall
	nFaces          2308;
	startFace       2041234;
}
	
	
	/stator$ mpiexec -np 2 pimpleFoam -parallel | tee log .pimpleFoam

The simulation should now run....
