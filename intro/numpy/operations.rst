
..  For doctests
    
    >>> import numpy as np
    >>> np.random.seed(0)
    >>> # For doctest on headless environments
    >>> import matplotlib.pyplot as plt
    >>> plt.switch_backend("Agg")

.. currentmodule:: numpy


数组的数值运算
==============================

.. contents:: Section contents
    :local:
    :depth: 1



元素智能(elementwise)运算
----------------------

基本运算
................

（数组）与标量：

.. sourcecode:: pycon

    >>> a = np.array([1, 2, 3, 4])
    >>> a + 1
    array([2, 3, 4, 5])
    >>> 2**a
    array([ 2,  4,  8, 16])

所有运算符都是元素智能的：

.. sourcecode:: pycon

    >>> b = np.ones(4) + 1
    >>> a - b
    array([-1.,  0.,  1.,  2.])
    >>> a * b
    array([ 2.,  4.,  6.,  8.])

    >>> j = np.arange(5)
    >>> 2**(j + 1) - j
    array([ 2,  3,  6, 13, 28])


并且NumPy比纯Python的计算速度快多了：

.. sourcecode:: pycon

   >>> a = np.arange(10000)
   >>> %timeit a + 1  # doctest: +SKIP
   10000 loops, best of 3: 24.3 us per loop
   >>> l = range(10000)
   >>> %timeit [i+1 for i in l] # doctest: +SKIP
   1000 loops, best of 3: 861 us per loop


.. warning:: **数组乘法并非矩阵乘法：**

    .. sourcecode:: pycon

        >>> c = np.ones((3, 3))
        >>> c * c                   # 不是矩阵乘法！
        array([[ 1.,  1.,  1.],
               [ 1.,  1.,  1.],
               [ 1.,  1.,  1.]])

.. note:: **矩阵乘法的实现：**

    .. sourcecode:: pycon

        >>> c.dot(c)
        array([[ 3.,  3.,  3.],
               [ 3.,  3.,  3.],
               [ 3.,  3.,  3.]])

.. topic:: **练习：元素智能运算**
   :class: green

    * 试试简单的元素智能运算：将偶数元素与奇数元素相加
    * 用 ``%timeit`` 指令比较NumPy与纯Python下的运算速度
    * 运行如下语句：

      * ``[2**0, 2**1, 2**2, 2**3, 2**4]``
      * ``a_j = 2^(3*j) - j``


其他运算
................

**比较：**

.. sourcecode:: pycon

    >>> a = np.array([1, 2, 3, 4])
    >>> b = np.array([4, 2, 2, 4])
    >>> a == b
    array([False,  True, False,  True], dtype=bool)
    >>> a > b
    array([False, False,  True, False], dtype=bool)

.. 建议::

   基于数组元素的比较：

   .. sourcecode:: pycon

    >>> a = np.array([1, 2, 3, 4])
    >>> b = np.array([4, 2, 2, 4])
    >>> c = np.array([1, 2, 3, 4])
    >>> np.array_equal(a, b)
    False
    >>> np.array_equal(a, c)
    True


**逻辑运算：**

.. sourcecode:: pycon

    >>> a = np.array([1, 1, 0, 0], dtype=bool)
    >>> b = np.array([1, 0, 1, 0], dtype=bool)
    >>> np.logical_or(a, b)
    array([ True,  True,  True, False], dtype=bool)
    >>> np.logical_and(a, b)
    array([ True, False, False, False], dtype=bool)

**超越函数：**

.. sourcecode:: pycon

    >>> a = np.arange(5)
    >>> np.sin(a)
    array([ 0.        ,  0.84147098,  0.90929743,  0.14112001, -0.7568025 ])
    >>> np.log(a)
    array([       -inf,  0.        ,  0.69314718,  1.09861229,  1.38629436])
    >>> np.exp(a)
    array([  1.        ,   2.71828183,   7.3890561 ,  20.08553692,  54.59815003])


**数组大小不匹配：**

.. sourcecode:: pycon

    >>> a = np.arange(4)
    >>> a + np.array([1, 2])  # doctest: +SKIP
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ValueError: operands could not be broadcast together with shapes (4) (2)

*Broadcasting?* We'll return to that :ref:`later <broadcasting>`.

**转置：**

.. sourcecode:: pycon

    >>> a = np.triu(np.ones((3, 3)), 1)   # see help(np.triu)
    >>> a
    array([[ 0.,  1.,  1.],
           [ 0.,  0.,  1.],
           [ 0.,  0.,  0.]])
    >>> a.T
    array([[ 0.,  0.,  0.],
           [ 1.,  0.,  0.],
           [ 1.,  1.,  0.]])


.. warning:: **转置操作只产生数组的view**

    因此下列代码 **是错的**， **不会返回对称矩阵**::

        >>> a += a.T

    对于小矩阵而言或许可以（由于缓冲区），但对大矩阵，上述代码会返回不可预料的内容。

.. note:: **线性代数**

    子模块 :mod:`numpy.linalg` 是线性代数运算的补充，例如解线性方程组，分解奇异值等等。然而它不能保证最佳效率，我们推荐使用 :mod:`scipy.linalg` ，这在 :ref:`scipy_linalg` 介绍。

.. topic:: 练习其他运算
   :class: green

    * 查阅 ``np.allclose`` 的帮助文档，它有啥用？
    * 查阅 ``np.triu`` 和 ``np.tril`` 的帮助文档。


基本简化运算
----------------

求和
..............

.. sourcecode:: pycon

    >>> x = np.array([1, 2, 3, 4])
    >>> np.sum(x)
    10
    >>> x.sum()
    10

.. image:: images/reductions.png
    :align: right

按行、列分别求和：

.. sourcecode:: pycon

    >>> x = np.array([[1, 1], [2, 2]])
    >>> x
    array([[1, 1],
           [2, 2]])
    >>> x.sum(axis=0)   # 按列求和（第一维）
    array([3, 3])
    >>> x[:, 0].sum(), x[:, 1].sum()
    (3, 3)
    >>> x.sum(axis=1)   # 按行求和（第二维）
    array([2, 4])
    >>> x[0, :].sum(), x[1, :].sum()
    (2, 4)

.. tip::

  高维数组求和的方法类似：

  .. sourcecode:: pycon

    >>> x = np.random.rand(2, 2, 2)
    >>> x.sum(axis=2)[0, 1]     # doctest: +ELLIPSIS
    1.14764...
    >>> x[0, 1, :].sum()     # doctest: +ELLIPSIS
    1.14764...

其它简化运算
................

--- 格式与求和运算类似（比如 ``axis``）

**极值：**

.. sourcecode:: pycon

  >>> x = np.array([1, 3, 2])
  >>> x.min()
  1
  >>> x.max()
  3

  >>> x.argmin()  # 返回最小值的索引
  0
  >>> x.argmax()  # 返回最大值的索引
  1

**逻辑运算：**

.. sourcecode:: pycon

  >>> np.all([True, True, False])
  False
  >>> np.any([True, True, False])
  True

.. note::

   上述逻辑运算可用于数组间比较：

   .. sourcecode:: pycon

      >>> a = np.zeros((100, 100))
      >>> np.any(a != 0)
      False
      >>> np.all(a == a)
      True

      >>> a = np.array([1, 2, 3, 2])
      >>> b = np.array([2, 2, 3, 2])
      >>> c = np.array([6, 4, 4, 5])
      >>> ((a <= b) & (b <= c)).all()
      True

**统计：**

.. sourcecode:: pycon

  >>> x = np.array([1, 2, 3, 1])
  >>> y = np.array([[1, 2, 3], [5, 6, 1]])
  >>> x.mean()
  1.75
  >>> np.median(x)
  1.5
  >>> np.median(y, axis=-1) # last axis
  array([ 2.,  5.])

  >>> x.std()          # 整体标准差
  0.82915619758884995


... 其它运算可在实践中查阅、使用。

.. topic:: **练习：简化运算**
   :class: green

    * 你能想到哪些有关 ``sum`` 函数的其他函数？
    * ``sum`` 与 ``cumsum`` 之间有什么差别？ 

.. topic:: 实例：数据统计
   :class: green

   数据见 :download:`populations.txt <../../data/populations.txt>` 
   
   数据内容是加拿大北部20年的野兔、山猫以及胡萝卜数。


   可以在文本编辑器查看这些数据，或者在IPython的shell或notebook里查看:

   .. sourcecode:: ipython

     In [1]: !cat data/populations.txt

   首先将这些数据导入为NumPy数组:

   .. sourcecode:: pycon

     >>> data = np.loadtxt('data/populations.txt')
     >>> year, hares, lynxes, carrots = data.T  # trick: columns to variables

   然后绘图:

   .. sourcecode:: pycon

     >>> from matplotlib import pyplot as plt
     >>> plt.axes([0.2, 0.1, 0.5, 0.8]) # doctest: +SKIP
     >>> plt.plot(year, hares, year, lynxes, year, carrots) # doctest: +SKIP
     >>> plt.legend(('Hare', 'Lynx', 'Carrot'), loc=(1.05, 0.5)) # doctest: +SKIP

   .. plot:: pyplots/numpy_intro_4.py

   计算平均数量:

   .. sourcecode:: pycon

     >>> populations = data[:, 1:]
     >>> populations.mean(axis=0)
     array([ 34080.95238095,  20166.66666667,  42400.        ])

   样本标准差:

   .. sourcecode:: pycon

     >>> populations.std(axis=0)
     array([ 20897.90645809,  16254.59153691,   3322.50622558])

   每一年数量最多的物种是？

   .. sourcecode:: pycon

     >>> np.argmax(populations, axis=1)
     array([2, 2, 0, 0, 1, 1, 2, 2, 2, 2, 2, 2, 0, 0, 0, 1, 2, 2, 2, 2, 2])

.. topic:: 实例：用随机行走算法模拟的扩散行为

  .. image:: random_walk.png
     :align: center

  .. tip::

    考虑简单的一维随机行走过程：
    Let us consider a simple 1D random walk process: at each time step a
    walker jumps right or left with equal probability.

    We are interested in finding the typical distance from the origin of a
    random walker after ``t`` left or right jumps? We are going to
    simulate many "walkers" to find this law, and we are going to do so
    using array computing tricks: we are going to create a 2D array with
    the "stories" (each walker has a story) in one direction, and the
    time in the other:

  .. only:: latex

    .. image:: random_walk_schema.png
        :align: center

  .. only:: html

    .. image:: random_walk_schema.png
        :align: center
        :width: 100%

  .. sourcecode:: pycon

   >>> n_stories = 1000 # number of walkers
   >>> t_max = 200      # time during which we follow the walker

  We randomly choose all the steps 1 or -1 of the walk:

  .. sourcecode:: pycon

   >>> t = np.arange(t_max)
   >>> steps = 2 * np.random.random_integers(0, 1, (n_stories, t_max)) - 1
   >>> np.unique(steps) # Verification: all steps are 1 or -1
   array([-1,  1])

  We build the walks by summing steps along the time:

  .. sourcecode:: pycon

   >>> positions = np.cumsum(steps, axis=1) # axis = 1: dimension of time
   >>> sq_distance = positions**2

  We get the mean in the axis of the stories:

  .. sourcecode:: pycon

   >>> mean_sq_distance = np.mean(sq_distance, axis=0)

  Plot the results:

  .. sourcecode:: pycon

   >>> plt.figure(figsize=(4, 3)) # doctest: +ELLIPSIS
   <matplotlib.figure.Figure object at ...>
   >>> plt.plot(t, np.sqrt(mean_sq_distance), 'g.', t, np.sqrt(t), 'y-') # doctest: +ELLIPSIS
   [<matplotlib.lines.Line2D object at ...>, <matplotlib.lines.Line2D object at ...>]
   >>> plt.xlabel(r"$t$") # doctest: +ELLIPSIS
   <matplotlib.text.Text object at ...>
   >>> plt.ylabel(r"$\sqrt{\langle (\delta x)^2 \rangle}$") # doctest: +ELLIPSIS
   <matplotlib.text.Text object at ...>

  .. plot:: pyplots/numpy_intro_5.py

  We find a well-known result in physics: the RMS distance grows as the
  square root of the time!


.. arithmetic: sum/prod/mean/std

.. extrema: min/max

.. logical: all/any

.. the axis argument

.. EXE: verify if all elements in an array are equal to 1
.. EXE: verify if any elements in an array are equal to 1
.. EXE: load data with loadtxt from a file, and compute its basic statistics

.. CHA: implement mean and std using only sum()

.. _broadcasting:

Broadcasting
------------

* Basic operations on ``numpy`` arrays (addition, etc.) are elementwise

* This works on arrays of the same size.

    | **Nevertheless**, It's also possible to do operations on arrays of different
    | sizes if *Numpy* can transform these arrays so that they all have
    | the same size: this conversion is called **broadcasting**.

The image below gives an example of broadcasting:

.. only:: latex

    .. image:: images/numpy_broadcasting.png
        :align: center

.. only:: html

    .. image:: images/numpy_broadcasting.png
        :align: center
        :width: 100%

Let's verify:

.. sourcecode:: pycon

    >>> a = np.tile(np.arange(0, 40, 10), (3, 1)).T
    >>> a
    array([[ 0,  0,  0],
           [10, 10, 10],
           [20, 20, 20],
           [30, 30, 30]])
    >>> b = np.array([0, 1, 2])
    >>> a + b
    array([[ 0,  1,  2],
           [10, 11, 12],
           [20, 21, 22],
           [30, 31, 32]])

We have already used broadcasting without knowing it!:

.. sourcecode:: pycon

    >>> a = np.ones((4, 5))
    >>> a[0] = 2  # we assign an array of dimension 0 to an array of dimension 1
    >>> a
    array([[ 2.,  2.,  2.,  2.,  2.],
           [ 1.,  1.,  1.,  1.,  1.],
           [ 1.,  1.,  1.,  1.,  1.],
           [ 1.,  1.,  1.,  1.,  1.]])

An useful trick:

.. sourcecode:: pycon

    >>> a = np.arange(0, 40, 10)
    >>> a.shape
    (4,)
    >>> a = a[:, np.newaxis]  # adds a new axis -> 2D array
    >>> a.shape
    (4, 1)
    >>> a
    array([[ 0],
           [10],
           [20],
           [30]])
    >>> a + b
    array([[ 0,  1,  2],
           [10, 11, 12],
           [20, 21, 22],
           [30, 31, 32]])


.. tip::

    Broadcasting seems a bit magical, but it is actually quite natural to
    use it when we want to solve a problem whose output data is an array
    with more dimensions than input data.

.. topic:: Worked Example: Broadcasting
   :class: green

   Let's construct an array of distances (in miles) between cities of
   Route 66: Chicago, Springfield, Saint-Louis, Tulsa, Oklahoma City,
   Amarillo, Santa Fe, Albuquerque, Flagstaff and Los Angeles.

   .. sourcecode:: pycon

       >>> mileposts = np.array([0, 198, 303, 736, 871, 1175, 1475, 1544,
       ...        1913, 2448])
       >>> distance_array = np.abs(mileposts - mileposts[:, np.newaxis])
       >>> distance_array
       array([[   0,  198,  303,  736,  871, 1175, 1475, 1544, 1913, 2448],
              [ 198,    0,  105,  538,  673,  977, 1277, 1346, 1715, 2250],
              [ 303,  105,    0,  433,  568,  872, 1172, 1241, 1610, 2145],
              [ 736,  538,  433,    0,  135,  439,  739,  808, 1177, 1712],
              [ 871,  673,  568,  135,    0,  304,  604,  673, 1042, 1577],
              [1175,  977,  872,  439,  304,    0,  300,  369,  738, 1273],
              [1475, 1277, 1172,  739,  604,  300,    0,   69,  438,  973],
              [1544, 1346, 1241,  808,  673,  369,   69,    0,  369,  904],
              [1913, 1715, 1610, 1177, 1042,  738,  438,  369,    0,  535],
              [2448, 2250, 2145, 1712, 1577, 1273,  973,  904,  535,    0]])


   .. image:: images/route66.png
      :align: center
      :scale: 60

A lot of grid-based or network-based problems can also use
broadcasting. For instance, if we want to compute the distance from
the origin of points on a 10x10 grid, we can do

.. sourcecode:: pycon

    >>> x, y = np.arange(5), np.arange(5)[:, np.newaxis]
    >>> distance = np.sqrt(x ** 2 + y ** 2)
    >>> distance
    array([[ 0.        ,  1.        ,  2.        ,  3.        ,  4.        ],
           [ 1.        ,  1.41421356,  2.23606798,  3.16227766,  4.12310563],
           [ 2.        ,  2.23606798,  2.82842712,  3.60555128,  4.47213595],
           [ 3.        ,  3.16227766,  3.60555128,  4.24264069,  5.        ],
           [ 4.        ,  4.12310563,  4.47213595,  5.        ,  5.65685425]])

Or in color:

.. sourcecode:: pycon

    >>> plt.pcolor(distance)    # doctest: +SKIP
    >>> plt.colorbar()    # doctest: +SKIP

.. plot:: pyplots/numpy_intro_6.py


**Remark** : the ``numpy.ogrid`` function allows to directly create vectors x
and y of the previous example, with two "significant dimensions":

.. sourcecode:: pycon

    >>> x, y = np.ogrid[0:5, 0:5]
    >>> x, y
    (array([[0],
           [1],
           [2],
           [3],
           [4]]), array([[0, 1, 2, 3, 4]]))
    >>> x.shape, y.shape
    ((5, 1), (1, 5))
    >>> distance = np.sqrt(x ** 2 + y ** 2)

.. tip::

  So, ``np.ogrid`` is very useful as soon as we have to handle
  computations on a grid. On the other hand, ``np.mgrid`` directly
  provides matrices full of indices for cases where we can't (or don't
  want to) benefit from broadcasting:

  .. sourcecode:: pycon

    >>> x, y = np.mgrid[0:4, 0:4]
    >>> x
    array([[0, 0, 0, 0],
           [1, 1, 1, 1],
           [2, 2, 2, 2],
           [3, 3, 3, 3]])
    >>> y
    array([[0, 1, 2, 3],
           [0, 1, 2, 3],
           [0, 1, 2, 3],
           [0, 1, 2, 3]])

.. rules

.. some usage examples: scalars, 1-d matrix products

.. newaxis

.. EXE: add 1-d array to a scalar
.. EXE: add 1-d array to a 2-d array
.. EXE: multiply matrix from the right with a diagonal array
.. CHA: constructing grids -- meshgrid using only newaxis



Array shape manipulation
------------------------

Flattening
..........

.. sourcecode:: pycon

    >>> a = np.array([[1, 2, 3], [4, 5, 6]])
    >>> a.ravel()
    array([1, 2, 3, 4, 5, 6])
    >>> a.T
    array([[1, 4],
           [2, 5],
           [3, 6]])
    >>> a.T.ravel()
    array([1, 4, 2, 5, 3, 6])

Higher dimensions: last dimensions ravel out "first".

Reshaping
.........

The inverse operation to flattening:

.. sourcecode:: pycon

    >>> a.shape
    (2, 3)
    >>> b = a.ravel()
    >>> b = b.reshape((2, 3))
    >>> b
    array([[1, 2, 3],
           [4, 5, 6]])

Or,

.. sourcecode:: pycon

    >>> a.reshape((2, -1))    # unspecified (-1) value is inferred
    array([[1, 2, 3],
           [4, 5, 6]])

.. warning::

   ``ndarray.reshape`` **may** return a view (cf ``help(np.reshape)``)), 
   or copy

.. tip::

   .. sourcecode:: pycon

     >>> b[0, 0] = 99
     >>> a
     array([[99,  2,  3],
            [ 4,  5,  6]])

   Beware: reshape may also return a copy!:

   .. sourcecode:: pycon

     >>> a = np.zeros((3, 2))
     >>> b = a.T.reshape(3*2)
     >>> b[0] = 9
     >>> a
     array([[ 0.,  0.],
            [ 0.,  0.],
            [ 0.,  0.]])

   To understand this you need to learn more about the memory layout of a numpy array.

Adding a dimension
..................

Indexing with the ``np.newaxis`` object allows us to add an axis to an array
(you have seen this already above in the broadcasting section):

.. sourcecode:: pycon

    >>> z = np.array([1, 2, 3])
    >>> z
    array([1, 2, 3])

    >>> z[:, np.newaxis]
    array([[1],
           [2],
           [3]])

    >>> z[np.newaxis, :]
    array([[1, 2, 3]])



Dimension shuffling
...................

.. sourcecode:: pycon

    >>> a = np.arange(4*3*2).reshape(4, 3, 2)
    >>> a.shape
    (4, 3, 2)
    >>> a[0, 2, 1]
    5
    >>> b = a.transpose(1, 2, 0)
    >>> b.shape
    (3, 2, 4)
    >>> b[2, 1, 0]
    5

Also creates a view:

.. sourcecode:: pycon

    >>> b[2, 1, 0] = -1
    >>> a[0, 2, 1]
    -1

Resizing
........

Size of an array can be changed with ``ndarray.resize``:

.. sourcecode:: pycon

    >>> a = np.arange(4)
    >>> a.resize((8,))
    >>> a
    array([0, 1, 2, 3, 0, 0, 0, 0])

However, it must not be referred to somewhere else:

.. sourcecode:: pycon

    >>> b = a
    >>> a.resize((4,))   # doctest: +SKIP
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ValueError: cannot resize an array that has been referenced or is
    referencing another array in this way.  Use the resize function

.. seealso: ``help(np.tensordot)``

.. resizing: how to do it, and *when* is it possible (not always!)

.. reshaping (demo using an image?)

.. dimension shuffling

.. when to use: some pre-made algorithm (e.g. in Fortran) accepts only
   1-D data, but you'd like to vectorize it

.. EXE: load data incrementally from a file, by appending to a resizing array
.. EXE: vectorize a pre-made routine that only accepts 1-D data
.. EXE: manipulating matrix direct product spaces back and forth (give an example from physics -- spin index and orbital indices)
.. EXE: shuffling dimensions when writing a general vectorized function
.. CHA: the mathematical 'vec' operation

.. topic:: **Exercise: Shape manipulations**
   :class: green

   * Look at the docstring for ``reshape``, especially the notes section which
     has some more information about copies and views.
   * Use ``flatten`` as an alternative to ``ravel``. What is the difference?
     (Hint: check which one returns a view and which a copy)
   * Experiment with ``transpose`` for dimension shuffling.

Sorting data
------------

Sorting along an axis:

.. sourcecode:: pycon

    >>> a = np.array([[4, 3, 5], [1, 2, 1]])
    >>> b = np.sort(a, axis=1)
    >>> b
    array([[3, 4, 5],
           [1, 1, 2]])

.. note:: Sorts each row separately!

In-place sort:

.. sourcecode:: pycon

    >>> a.sort(axis=1)
    >>> a
    array([[3, 4, 5],
           [1, 1, 2]])

Sorting with fancy indexing:

.. sourcecode:: pycon

    >>> a = np.array([4, 3, 1, 2])
    >>> j = np.argsort(a)
    >>> j
    array([2, 3, 1, 0])
    >>> a[j]
    array([1, 2, 3, 4])

Finding minima and maxima:

.. sourcecode:: pycon

    >>> a = np.array([4, 3, 1, 2])
    >>> j_max = np.argmax(a)
    >>> j_min = np.argmin(a)
    >>> j_max, j_min
    (0, 2)


.. XXX: need a frame for summaries

    * Arithmetic etc. are elementwise operations
    * Basic linear algebra, ``.dot()``
    * Reductions: ``sum(axis=1)``, ``std()``, ``all()``, ``any()``
    * Broadcasting: ``a = np.arange(4); a[:,np.newaxis] + a[np.newaxis,:]``
    * Shape manipulation: ``a.ravel()``, ``a.reshape(2, 2)``
    * Fancy indexing: ``a[a > 3]``, ``a[[2, 3]]``
    * Sorting data: ``.sort()``, ``np.sort``, ``np.argsort``, ``np.argmax``

.. topic:: **Exercise: Sorting**
   :class: green

    * Try both in-place and out-of-place sorting.
    * Try creating arrays with different dtypes and sorting them.
    * Use ``all`` or ``array_equal`` to check the results.
    * Look at ``np.random.shuffle`` for a way to create sortable input quicker.
    * Combine ``ravel``, ``sort`` and ``reshape``.
    * Look at the ``axis`` keyword for ``sort`` and rewrite the previous
      exercise.

Summary
-------

**What do you need to know to get started?**

* Know how to create arrays : ``array``, ``arange``, ``ones``,
  ``zeros``.

* Know the shape of the array with ``array.shape``, then use slicing
  to obtain different views of the array: ``array[::2]``,
  etc. Adjust the shape of the array using ``reshape`` or flatten it
  with ``ravel``.

* Obtain a subset of the elements of an array and/or modify their values
  with masks

  .. sourcecode:: pycon

     >>> a[a < 0] = 0

* Know miscellaneous operations on arrays, such as finding the mean or max
  (``array.max()``, ``array.mean()``). No need to retain everything, but
  have the reflex to search in the documentation (online docs,
  ``help()``, ``lookfor()``)!!

* For advanced use: master the indexing with arrays of integers, as well as
  broadcasting. Know more Numpy functions to handle various array
  operations.

.. topic:: **Quick read**

   If you want to do a first quick pass through the Scipy lectures to
   learn the ecosystem, you can directly skip to the next chapter:
   :ref:`matplotlib`.

   The remainder of this chapter is not necessary to follow the rest of
   the intro part. But be sure to come back and finish this chapter, as
   well as to do some more :ref:`exercices <numpy_exercises>`.
