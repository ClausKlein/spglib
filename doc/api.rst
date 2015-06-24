API
====

``spg_get_symmetry``
^^^^^^^^^^^^^^^^^^^^

::

  int spg_get_symmetry(int rotation[][3][3],
  		       double translation[][3],
  		       const int max_size,
		       const double lattice[3][3],
  		       const double position[][3],
		       const int types[],
  		       const int num_atom,
		       const double symprec);

Find symmetry operations. The operations are stored in ``rotatiion``
and ``translation``. The number of operations is return as the return
value. Rotations and translations are given in fractional coordinates,
and ``rotation[i]`` and ``translation[i]`` with same index give a
symmetry oprations, i.e., these have to be used togather.

``spg_get_international``
^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  int spg_get_international(char symbol[11],
                            const double lattice[3][3],
                            const double position[][3],
                            const int types[],
			    const int num_atom,
                            const double symprec);

Space group is found in international table symbol (``symbol``) and
as number (return value). 0 is returned when it fails.

``spg_get_schoenflies``
^^^^^^^^^^^^^^^^^^^^^^^^

::

  int spg_get_schoenflies(char symbol[10], const double lattice[3][3],
                          const double position[][3],
                          const int types[], const int num_atom,
                          const double symprec);

Space group is found in schoenflies (``symbol``) and as number (return
value).  0 is returned when it fails.


``spg_find_primitive``
^^^^^^^^^^^^^^^^^^^^^^^

::
  
  int spg_find_primitive(double lattice[3][3],
                         double position[][3],
                         int types[],
			 const int num_atom,
			 const double symprec);

A primitive cell is found from an input cell. Be careful that 
``lattice``, ``position``, and ``types`` are overwritten. ``num_atom``
is returned as return value.

``spg_refine_cell``
^^^^^^^^^^^^^^^^^^^^^

Symmetrized and standarized crystal structure is obtained from a
non-standard crystal structure which may be slightly distorted within
a symmetry recognition tolerance, or whose primitive vectors are differently
chosen, etc.

::

  int spg_refine_cell(double lattice[3][3],
		      double position[][3],
		      int types[],
		      const int num_atom,
 		      const double symprec);

Returned Bravais lattice and symmetrized atomic positions are
overwritten. The number of atoms in the Bravais lattice is returned as
the return value. The memory space for ``position`` and ``types`` must
be prepared four times more than those required for the input
structures. This is because, when the crystal has the face centering,
four times more atoms than those in the primitive cell are
generated. To do the same for the non-standard choices of origin,
axis, or cell, it is necessary to use
``spg_get_dataset_with_hall_number`` to extract the crystal structure.

.. _api_spg_get_dataset:

``spg_get_dataset``, ``spg_get_dataset_with_hall_number``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For an input crystal structure, the space group operations of crystal
are searched. Then they are compared with the crsytallographic
database and the space group type is determined. The result is
returned as the ``SpglibDataset`` structure as a dataset.

Usage
------

Dataset corresponding to the space group type in the **standard
setting** is obtained by ``spg_get_dataset``. If this symmetry search
fails, ``spacegroup_number`` in the ``SpglibDataset`` structure is
set 0. In this function, the other crystallographic setting is not
obtained.

::

   SpglibDataset * spg_get_dataset(const double lattice[3][3],
                                   const double position[][3],
                                   const int types[],
                                   const int num_atom,
                                   const double symprec);

To specify the other crystallographic setting (origin, axis, or cell
choice), ``spg_get_dataset_with_hall_number`` is used. 
				   
:: 
				  
   SpglibDataset * spg_get_dataset_with_hall_number(SPGCONST double lattice[3][3],
						    SPGCONST double position[][3],
						    const int types[],
						    const int num_atom,
						    const int hall_number,
						    const double symprec)

where ``hall_number`` is used to specify the setting. The possible
choices and those serial numbers are found at `list of space groups
(Seto's web site)
<http://pmsl.planet.sci.kobe-u.ac.jp/~seto/?page_id=37&lang=en>`_.
The crystal structure has to possess the space-group type of the Hall
symbol. If the symmetry search fails or the specified ``hall_number``
is not in the list of Hall symbols for the space group type of the
crystal structure, ``spacegroup_number`` in the ``SpglibDataset``
structure is set 0.

Finally, its allocated memory space must be freed by calling ``spg_free_dataset``.


Dataset
--------
				  
The dataset is
accessible through the C-structure given by

::

   typedef struct {
     int spacegroup_number;
     int hall_number;
     char international_symbol[11];
     char hall_symbol[17];
     char setting[6];
     double transformation_matrix[3][3];
     double origin_shift[3];
     int n_operations;
     int (*rotations)[3][3];
     double (*translations)[3];
     int n_atoms;
     int *wyckoffs;
     int *equivalent_atoms;
     double brv_lattice[3][3];
     int *brv_types;
     double (*brv_positions)[3];
   } SpglibDataset;

.. _api_spg_get_dataset_spacegroup_type:

Space group type
"""""""""""""""""

``spacegroup_number`` is the space group type number defined in
International Tables for Crystallography (ITA). ``hall_number`` is the
serial number between 1 and 530 which are found at `list of space
groups (Seto's web site)
<http://pmsl.planet.sci.kobe-u.ac.jp/~seto/?page_id=37&lang=en>`_.
The (full) Hermann–Mauguin notation of space group type is given by
``international_symbol``. The Hall symbol is stored in
``hall_symbol``. The information on unique axis,
setting or cell choices is found in ``setting``.
   
Space group operations
"""""""""""""""""""""""
   
The symmetry operations of the input cell are stored in ``rotations``
and ``translations``. A space group operation :math:`(R|\tau)` is made
from a set of rotation :math:`R` and translation :math:`\tau` with the
same index. Number of space group operations is found in
``n_operations``. 

Site symmetry
""""""""""""""

``n_atoms`` is the number of
atoms of the input cell. ``wyckoffs`` gives Wyckoff letters that are
assigned to atomic positions of the input cell. The numbers of 0, 1,
2, :math:`\ldots`, correspond to the a, b, c, :math:`\ldots`,
respectively. Number of elements in ``wyckoffs`` is same as
``n_atoms``. ``equivalent_atoms`` is a list of atomic indices that map
to indices of symmetrically independent atoms, where the list index
corresponds to atomic index of the input crystal structure.

Origin shift and lattice transformation
""""""""""""""""""""""""""""""""""""""""

``transformation_matrix`` and ``origin_shift`` are obtained as a
result of space-group-type matching under a set of unique axis,
setting and cell choices. In this matching, lattice and atomic
positions have to be standardized to compare with the database of
space-group operations. The lattice is transformed to a Bravais
lattice. Atomic positions are shifted in order that symmetry
operations have a standard origin.  ``transformation_matrix``
(:math:`\mathrm{M}`) is the matrix to transform the input lattice to a
Bravais lattice with unique axis, setting and cell choices defined as

.. math::

   ( \mathbf{a}_\mathrm{B} \; \mathbf{b}_\mathrm{B} \; \mathbf{c}_\mathrm{B} )
   =  ( \mathbf{a} \; \mathbf{b} \; \mathbf{c} ) \mathrm{M}

where :math:`\mathbf{a}_\mathrm{B}`, :math:`\mathbf{b}_\mathrm{B}`,
and :math:`\mathbf{c}_\mathrm{B}` are the column vectors of a Bravais
lattice, and :math:`\mathbf{a}`, :math:`\mathbf{b}`, and
:math:`\mathbf{c}` are the column vectors of the input lattice. The
``origin_shift`` (:math:`\mathbf{o}`) is the atomic position shift in
terms of the Bravais lattice. The atomic position shift is measured
from the standardized cell (conventional unit cell) to the original
cell in terms of the Bravais lattice. An atomic position in the
original cell :math:`\mathbf{x}` (input data) is mapped to that in
Bravais lattice :math:`\mathbf{x}_\mathrm{B}` by

.. math::

   \mathbf{x}_\mathrm{B} = \mathrm{M}\mathbf{x} - \mathbf{o} \;\;(\mathrm{mod}\;
   \mathbf{1}).

Standardized crystal structure
"""""""""""""""""""""""""""""""
   
The standardized crystal structure corresponding to a Hall symbol is
stored in ``n_brv_atoms``, ``brv_lattice``, ``brv_types``, and ``brv_positions``.


``spg_free_dataset``
^^^^^^^^^^^^^^^^^^^^^

Allocated memoery space of the C-structure of ``SpglibDataset`` is
freed by calling ``spg_free_dataset``.

:: 

  void spg_free_dataset(SpglibDataset *dataset);
  

``spg_get_spacegroup_type``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This function allows to directly access to the space-group-type
database in spglib (spg_database.c). To specify the space group type
with a specific setting, ``hall_number`` is used. The definition of
``hall_number`` is found at
:ref:`api_spg_get_dataset_spacegroup_type`.


::

   SpglibSpacegroupType spg_get_spacegroup_type(const int hall_number)

``SpglibSpacegroupType`` structure is as follows:

::
   
   typedef struct {
     int number;
     char schoenflies[7];
     char hall_symbol[17];
     char international[32];
     char international_full[20];
     char international_short[11];
   } SpglibSpacegroupType;


``spg_get_symmetry_from_database``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This function allows to directly access to the space group operations
in the spglib database (spg_database.c). To specify the space group
type with a specific setting, ``hall_number`` is used. The definition
of ``hall_number`` is found at
:ref:`api_spg_get_dataset_spacegroup_type`.

::

   int spg_get_symmetry_from_database(int rotations[192][3][3],
				      double translations[192][3],
				      const int hall_number);

The returned value is the number of space group operations. The space
group operations are stored in ``rotations`` and ``translations``.
  
``spg_get_smallest_lattice``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  int spg_get_smallest_lattice(double smallest_lattice[3][3],
  			       const double lattice[3][3],
			       const double symprec)

Considering periodicity of crystal, one of the possible smallest lattice is
searched. The lattice is stored in ``smallest_lattice``.

``spg_get_multiplicity``
^^^^^^^^^^^^^^^^^^^^^^^^^
  
::

  int spg_get_multiplicity(const double lattice[3][3],
  			   const double position[][3],
  			   const int types[],
			   const int num_atom,
  			   const double symprec);

Return exact number of symmetry operations. This function may be used
in advance to allocate memoery space for symmetry operations.

``spg_get_symmetry_with_collinear_spin``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  int spg_get_symmetry_with_collinear_spin(int rotation[][3][3],
                                           double translation[][3],
                                           const int max_size,
                                           SPGCONST double lattice[3][3],
                                           SPGCONST double position[][3],
                                           const int types[],
                                           const double spins[],
                                           const int num_atom,
                                           const double symprec);

Find symmetry operations with collinear spins on atoms. Except for the
argument of ``const double spins[]``, the usage is same as
``spg_get_symmetry``.

``spg_get_ir_reciprocal_mesh``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

   int spg_get_ir_reciprocal_mesh(int grid_address[][3],
                                  int map[],
                                  const int mesh[3],
                                  const int is_shift[3],
                                  const int is_time_reversal,
                                  const double lattice[3][3],
                                  const double position[][3],
                                  const int types[],
                                  const int num_atom,
                                  const double symprec)

Irreducible reciprocal grid points are searched from uniform mesh grid
points specified by ``mesh`` and ``is_shift``.  ``mesh`` stores three
integers. Reciprocal primitive vectors are divided by the number
stored in ``mesh`` with (0,0,0) point centering. The center of grid
mesh is shifted +1/2 of a grid spacing along corresponding reciprocal
axis by setting 1 to a ``is_shift`` element. No grid mesh shift is
made if 0 is set for ``is_shift``.

The reducible uniform grid points are returned in reduced coordinates
as ``grid_address``. A map between reducible and irreducible points are
returned as ``map`` as in the indices of ``grid_address``. The number of
the irreducible k-points are returned as the return value.  The time
reversal symmetry is imposed by setting ``is_time_reversal`` 1.

Grid points are stored in the order that runs left most element
first, e.g. (4x4x4 mesh).::

   [[ 0  0  0]   
    [ 1  0  0]   
    [ 2  0  0]   
    [-1  0  0]   
    [ 0  1  0]   
    [ 1  1  0]   
    [ 2  1  0]   
    [-1  1  0]   
    ....      ]  

where the first index runs first.  k-qpoints are calculated by
``(grid_address + is_shift / 2) / mesh``. A grid point index is
recovered from ``grid_address`` by ``numpy.dot(grid_address % mesh,
[1, mesh[0], mesh[0] * mesh[1]])`` in Python-numpy notation, where
``%`` always returns non-negative integers. The order of
``grid_address`` can be changed so that the last index runs first by
setting the macro ``GRID_ORDER_XYZ`` in ``kpoint.c``. In this case the
grid point index is recovered by ``numpy.dot(grid_address % mesh,
[mesh[2] * mesh[1], mesh[2], 1])``.

``spg_get_stabilized_reciprocal_mesh``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Change in version 1.4**

::

   int spg_get_stabilized_reciprocal_mesh(int grid_address[][3],
                                          int map[],
                                          const int mesh[3],
                                          const int is_shift[3],
                                          const int is_time_reversal,
                                          const int num_rot,
                                          const int rotations[][3][3],
                                          const int num_q,
                                          const double qpoints[][3])

The irreducible k-points are searched from unique k-point mesh grids
from real space lattice vectors and rotation matrices of symmetry
operations in real space with stabilizers. The stabilizers are written
in reduced coordinates. Number of the stabilizers are given by
``num_q``. Reduced k-points are stored in ``map`` as indices of
``grid_address``. The number of the reduced k-points with stabilizers
are returned as the return value.

Mesh grid points without symmetrization can be obtained by setting
``num_rot = 1``, ``rotations = {{1, 0, 0}, {0, 1, 0}, {0, 0, 1}}``,
``num_q = 1``, and ``qpoints = {0, 0, 0}``.

.. |sflogo| image:: http://sflogo.sourceforge.net/sflogo.php?group_id=161614&type=1
            :target: http://sourceforge.net

|sflogo|
