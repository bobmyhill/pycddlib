.. testsetup::

   import cdd

.. currentmodule:: cdd

Working With Polyhedron Representations
=======================================

.. class:: Polyhedron(mat)

    A class for converting between representations of a polyhedron.

    Bases: :class:`~cdd.NumberTypeable`

    :param mat: The matrix to load the polyhedron from.
    :type mat: :class:`~cdd.Matrix`

Methods and Attributes
----------------------

.. method:: Polyhedron.get_inequalities()

        Get all inequalities.

        :returns: H-representation.
        :rtype: :class:`~cdd.Matrix`

        For a polyhedron described as `P = {x | A x <= b}`, the
        H-representation is the matrix `[b -A]`.

.. method:: Polyhedron.get_generators()

        Get all generators.

        :returns: V-representation.
        :rtype: :class:`~cdd.Matrix`

        For a polyhedron described as `P = conv(v_1, ..., v_n) + nonneg(r_1,
        ..., r_s)`, the V-representation matrix is `[t V]` where `t` is the
        column vector with `n` ones followed by `s` zeroes, and `V` is the
        stacked matrix of `n` vertex row vectors on top of `s` ray row vectors.

.. method:: Polyhedron.get_adjacency()

        Get the adjacencies.

        :returns: Adjacency list.
        :rtype: :class:`tuple`

        H-representation: For each vertex, list adjacent vertices.
        V-representation: For each face, list adjacent faces.

.. method:: Polyhedron.get_input_adjacency()

        Get the input adjacencies.

        :returns: Input adjacency list.
        :rtype: :class:`tuple`

        H-representation: For each face, list adjacent faces.
        V-representation: For each vertex, list adjacent vertices.

.. method:: Polyhedron.get_incidence()

        Get the incidences.

        :returns: Incidence list.
        :rtype: :class:`tuple`

        H-representation: For each vertex, list adjacent faces.
        V-representation: For each face, list adjacent vertices.

.. method:: Polyhedron.get_input_incidence()

        Get the input incidences.

        :returns: Input incidence list.
        :rtype: :class:`tuple`

        H-representation: For each face, list adjacent vertices.
        V-representation: For each vertex, list adjacent faces.

.. attribute:: Polyhedron.rep_type

        Representation (see :class:`~cdd.RepType`).

.. note::

    The H-representation and/or V-representation are not guaranteed to
    be minimal, that is, they can still contain redundancy.

Examples
--------

This is the sampleh1.ine example that comes with cddlib.

>>> import cdd
>>> mat = cdd.Matrix([[2,-1,-1,0],[0,1,0,0],[0,0,1,0]], number_type='fraction')
>>> mat.rep_type = cdd.RepType.INEQUALITY
>>> poly = cdd.Polyhedron(mat)
>>> print(poly)
begin
 3 4 rational
 2 -1 -1 0
 0 1 0 0
 0 0 1 0
end
>>> ext = poly.get_generators()
>>> print(ext)
V-representation
linearity 1  4
begin
 4 4 rational
 1 0 0 0
 1 2 0 0
 1 0 2 0
 0 0 0 1
end
>>> print(list(ext.lin_set)) # note: first row is 0, so fourth row is 3
[3]


The following example illustrates how to get adjacencies and incidences.

>>> import cdd
>>> # We start with the H-representation for a square
>>> # 0 <= 1 + x1 (face 0)
>>> # 0 <= 1 + x2 (face 1)
>>> # 0 <= 1 - x1 (face 2)
>>> # 0 <= 1 - x2 (face 3)
>>> mat = cdd.Matrix([[1, 1, 0], [1, 0, 1], [1, -1, 0], [1, 0, -1]])
>>> mat.rep_type = cdd.RepType.INEQUALITY
>>> poly = cdd.Polyhedron(mat)
>>> # The V-representation can be printed in the usual way:
>>> gen = poly.get_generators()
>>> print(gen)
V-representation
begin
 4 3 rational
 1 1 -1
 1 1 1
 1 -1 1
 1 -1 -1
end
>>> # graphical depiction of vertices and faces:
>>> #
>>> #   2---(3)---1
>>> #   |         |
>>> #   |         |
>>> #  (0)       (2)
>>> #   |         |
>>> #   |         |
>>> #   3---(1)---0
>>> #
>>> # vertex 0 is adjacent to vertices 1 and 3
>>> # vertex 1 is adjacent to vertices 0 and 2
>>> # vertex 2 is adjacent to vertices 1 and 3
>>> # vertex 3 is adjacent to vertices 0 and 2
>>> print([list(x) for x in poly.get_adjacency()])
[[1, 3], [0, 2], [1, 3], [0, 2]]
>>> # vertex 0 is the intersection of faces (1) and (2)
>>> # vertex 1 is the intersection of faces (2) and (3)
>>> # vertex 2 is the intersection of faces (0) and (3)
>>> # vertex 3 is the intersection of faces (0) and (1)
>>> print([list(x) for x in poly.get_incidence()])
[[1, 2], [2, 3], [0, 3], [0, 1]]
>>> # face (0) is adjacent to faces (1) and (3)
>>> # face (1) is adjacent to faces (0) and (2)
>>> # face (2) is adjacent to faces (1) and (3)
>>> # face (3) is adjacent to faces (0) and (2)
>>> print([list(x) for x in poly.get_input_adjacency()])
[[1, 3], [0, 2], [1, 3], [0, 2], []]
>>> # face (0) intersects with vertices 2 and 3
>>> # face (1) intersects with vertices 0 and 3
>>> # face (2) intersects with vertices 0 and 1
>>> # face (3) intersects with vertices 1 and 2
>>> print([list(x) for x in poly.get_input_incidence()])
[[2, 3], [0, 3], [0, 1], [1, 2], []]
>>> # add a vertex, and construct new polyhedron
>>> gen.extend([[1, 0, 2]])
>>> vpoly = cdd.Polyhedron(gen)
>>> print(vpoly.get_inequalities())
H-representation
begin
 5 3 rational
 1 0 1
 2 1 -1
 1 1 0
 2 -1 -1
 1 -1 0
end
>>> # so now we have:
>>> # 0 <= 1 + x2
>>> # 0 <= 2 + x1 - x2
>>> # 0 <= 1 + x1
>>> # 0 <= 2 - x1 - x2
>>> # 0 <= 1 - x1
>>> #
>>> # graphical depiction of vertices and faces:
>>> #
>>> #        4
>>> #       / \
>>> #      /   \
>>> #    (1)   (3)
>>> #    /       \
>>> #   2         1
>>> #   |         |
>>> #   |         |
>>> #  (2)       (4)
>>> #   |         |
>>> #   |         |
>>> #   3---(0)---0
>>> #
>>> # for each face, list adjacent faces
>>> print([list(x) for x in vpoly.get_adjacency()])
[[2, 4], [2, 3], [0, 1], [1, 4], [0, 3]]
>>> # for each face, list adjacent vertices
>>> print([list(x) for x in vpoly.get_incidence()])
[[0, 3], [2, 4], [2, 3], [1, 4], [0, 1]]
>>> # for each vertex, list adjacent vertices
>>> print([list(x) for x in vpoly.get_input_adjacency()])
[[1, 3], [0, 4], [3, 4], [0, 2], [1, 2]]
>>> # for each vertex, list adjacent faces
>>> print([list(x) for x in vpoly.get_input_incidence()])
[[0, 4], [3, 4], [1, 2], [0, 2], [1, 3]]
