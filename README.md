# PostMesh

<!--![almond](doc/almond.png)-->
<!--![mech](doc/mech2d.png)-->
<img src="docs/almond.png" width="380">
<img src="docs/mech2d.png" width="350">
<img src="docs/wing2d.png" width="320">
<img src="docs/torus.png" width="350">

PostMesh is a solid mechanics based a posteriori high order curvilinear mesh generator based on OpenCascade with C++, Cython and Python APIs. Its main goal is to serve as a bridge between CAD models and high order finite elements. Hence, it can be used as a plugin with various compiled and interpreted code-bases.

## Philosophy
PostMesh is an a posteriori approach, in that it requires a linear mesh in advance. Higher order nodes are then placed on the linear mesh and the projection of these nodes to the exact boundary is computed with the CAD library and subsequently fed as the Dirichlet boundary condition to either a linear, a linearised or a non-linear solid mechanics problem.

## Build Requirements
PostMesh depends on the following third party libraries:

- **C++11 compatible compiler**
- **[GNU make]**        - build process
- **[OpenCascade]**     - CAD processing
- **[Eigen]**           - Matrix operations and SIMD vectorisation
- **[Cython]**          - Cython bindings
- **[NumPy]**           - Python interface


[GNU make]:     http://www.gnu.org/software/make
[OpenCascade]:  http://www.opencascade.com
[Eigen]:        http://eigen.tuxfamily.org
[Cython]:       http://www.cython.org
[NumPy]:        http://www.numpy.org

Installing these dependencies on a linux distribution is straight-forward. For building OpenCascade on Debian based systems, [here](https://github.com/jmwright/pythonocc_oce_setup) is an automated shell script.

## Installation
To build PostMesh shared library for C++ API, perform the following

    git clone https://github.com/romeric/PostMesh
    cd PostMesh
    make
    [sudo] make install
    
To further build the C++ examples, after cloning the repository and entering PostMesh directory, do

    cd examples
    make
    
To build Cython/Python bindings

    git clone https://github.com/romeric/PostMesh
    [sudo] python setup.py install
    
    
### Usage
PostMesh provides a very intuitive objected oriented API. The wrappers are designed such that the C++ and Python codes look and feel the same. Have a look at the examples directory for getting started with PostMesh. For conveninece, here are two complete examples.

#### A complete C++ example: [3D] surface projections for high order tetrahedral elements
````c++
    // MAKE AN INSTANCE OF PostMeshSurface
    auto curvilinear_mesh = PostMeshSurface();
    // PASS MESH DATA TO PostMesh - PostMesh takes raw pointers as input arguments
    curvilinear_mesh.SetMeshElements(elements, elements_rows, elements_cols);
    curvilinear_mesh.SetMeshPoints(points,points_rows, points_cols);
    curvilinear_mesh.SetMeshEdges(edges, edges_rows, edges_cols);
    curvilinear_mesh.SetMeshFaces(faces,  faces_rows,  faces_cols);
    curvilinear_mesh.SetScale(scale);
    curvilinear_mesh.SetCondition(condition);
    curvilinear_mesh.SetProjectionPrecision(precision);
    curvilinear_mesh.ComputeProjectionCriteria();
    curvilinear_mesh.ScaleMesh();
    curvilinear_mesh.InferInterpolationPolynomialDegree();
    curvilinear_mesh.SetNodalSpacing(nodal_spacing, nodal_spacing_rows, nodal_spacing_cols);
    // READ THE GEOMETRY FROM THE IGES FILE
    curvilinear_mesh.ReadIGES(iges_filename);
    // EXTRACT GEOMETRY INFORMATION FROM THE IGES FILE
    curvilinear_mesh.GetGeomVertices();
    // EXTRACT TRUE BOUNDARY FACES FROM CAD FILE
    curvilinear_mesh.GetGeomFaces();
    curvilinear_mesh.GetGeomPointsOnCorrespondingFaces();
    // FIRST IDENTIFY WHICH CURVES CONTAIN WHICH EDGES
    curvilinear_mesh.IdentifySurfacesContainingFaces();
    // PROJECT ALL BOUNDARY POINTS FROM THE MESH TO THE SURFACE
    curvilinear_mesh.ProjectMeshOnSurface();
    // PERFORM POINT INVERSION FOR THE INTERIOR POINTS (ORTHOGONAL PROJECTION)
    curvilinear_mesh.MeshPointInversionSurface();
    // GET DIRICHLET DATA - (THE DISPLACMENT OF BOUNDARY NODES)
    DirichletData Dirichlet_data = curvilinear_mesh.GetDirichletData();

````

#### A complete Python example: [2D] curve projections for high order triangular elements
````python
    curvilinear_mesh = PostMeshCurve("tri",2)
    curvilinear_mesh.SetMeshElements(elements)
    curvilinear_mesh.SetMeshPoints(points)
    curvilinear_mesh.SetMeshEdges(edges)
    curvilinear_mesh.SetMeshFaces(np.zeros((1,4),dtype=np.uint64))
    curvilinear_mesh.SetScale(scale)
    curvilinear_mesh.SetCondition(condition)
    curvilinear_mesh.SetProjectionPrecision(1.0e-04)
    curvilinear_mesh.ComputeProjectionCriteria()
    curvilinear_mesh.ScaleMesh()
    curvilinear_mesh.SetNodalSpacing(nodal_spacing.reshape(9,1))
    curvilinear_mesh.GetBoundaryPointsOrder()
    # READ THE GEOMETRY FROM THE IGES FILE
    curvilinear_mesh.ReadIGES(iges_filename)
    # EXTRACT GEOMETRY INFORMATION FROM THE IGES FILE
    geometry_points = curvilinear_mesh.GetGeomVertices()
    curvilinear_mesh.GetGeomEdges()
    curvilinear_mesh.GetGeomFaces()
    curvilinear_mesh.GetGeomPointsOnCorrespondingEdges()
    # FIRST IDENTIFY WHICH CURVES CONTAIN WHICH EDGES
    curvilinear_mesh.IdentifyCurvesContainingEdges()
    # PROJECT ALL BOUNDARY POINTS FROM THE MESH TO THE CURVE
    curvilinear_mesh.ProjectMeshOnCurve()
    # FIX IMAGES AND ANTI IMAGES IN PERIODIC CURVES/SURFACES
    curvilinear_mesh.RepairDualProjectedParameters()
    # PERFORM POINT INVERSION FOR THE INTERIOR POINTS
    curvilinear_mesh.MeshPointInversionCurveArcLength()
    # OBTAIN MODIFIED MESH POINTS - THIS IS NECESSARY TO ENSURE LINEAR MESH IS ALSO CORRECT
    curvilinear_mesh.ReturnModifiedMeshPoints(points)
    # GET DIRICHLET DATA - (THE DISPLACMENT OF BOUNDARY NODES)
    nodesDBC, Dirichlet = curvilinear_mesh.GetDirichletData() 
````
