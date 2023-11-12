
.. highlight:: none
.. _sec-basics-waveplot:

*************************
First steps with Waveplot
*************************

[Input: `recipes/basics/waveplot/`]

Waveplot is a tool for generating grid-based volumetric data for charge
distributions or wave-functions in your system. By visualising those
distributions with appropriate graphical tools you can obtain a deeper
understanding of the physics and chemistry of your quantum mechanical system.


Making a DFTB+ calculation
==========================

# 为了绘制电荷分布和轨道，必须打开WriteDetailedXML和WriteEigenvectors选项

In order to plot the charge distribution or the orbitals in a certain system,
you have to execute a DFTB+ calculation for this system first. The calculation
must be executed as usual, you just have to make sure, that the options
``WriteDetailedXML`` and ``WriteEigenvectors`` are turned on.

Below you see the input for the H2O molecule, where the geometry is
optimised by DFTB+::

  Geometry = GenFormat {
  3  C
   O H
       1    1    0.00000000000E+00  -0.10000000000E+01   0.00000000000E+00
       2    2    0.00000000000E+00   0.00000000000E+00   0.78306400000E+00
       3    2    0.00000000000E+00   0.00000000000E+00  -0.78306400000E+00
  }

  Driver = ConjugateGradient {
    MovedAtoms = 1:-1
    MaxForceComponent = 1.0e-4
    MaxSteps = 100
    OutputPrefix = "geom.out"
  }

  Hamiltonian = DFTB {
    Scc = Yes
    SccTolerance = 1.0e-5
    SlaterKosterFiles = Type2FileNames {
      Prefix = "../../slakos/mio-ext/"
      Separator = "-"
      Suffix = ".skf"
    }
    MaxAngularMomentum = {
      O = "p"
      H = "s"
    }
  }

  Options {
    WriteDetailedXml = Yes
  }

  Analysis {
    WriteEigenvectors = Yes
  }

  ParserOptions {
    ParserVersion = 12
  }

Running DFTB+ for this input, you should obtain the usual results, and
additionaly the files ``detailed.xml`` and ``eigenvec.bin``. Former contains
some information about the calculated system, latter contains the obtained
eigenvectors in binary format. Both files are needed by waveplot.

# 得到Detailed.xml和eigve.bin，前者包含计算信息，后者包含所获得的二进制格式的特征向量。WavePlot需要这两个文件。

Running Waveplot
================

# 绘制水分子的总电荷分布、最高占据分子轨道 (HOMO) 的电荷分布（波函数平方）、HOMO 的波函数以及总电荷差

Now, you have to decide, what kind of charge distributions, wavefunctions etc.
to plot. In the current example, we will plot the total charge distribution of
the water molecule, the charge distribution (wavefunction squared) for the
highest occupied molecular orbital (HOMO), the wave function for the HOMO, and
the total charge difference, which tells us, how the chemical bonding between
the atoms modified the total charge distribution compared to the superpositions
of neutral atomic densities.

Input
-----
# 输入文件waveplot_in.hsd

The appropriate waveplot input (``waveplot_in.hsd``) could look like the
following::

  # General options

  Options {
    TotalChargeDensity = Yes           # Total density be plotted?
    TotalChargeDifference = Yes        # Total density difference plotted?
    ChargeDensity = Yes                # Charge density for each state?
    RealComponent = Yes                # Plot real component of the wavefunction
    PlottedSpins = 1 -1
    PlottedLevels = 4                  # Levels to plot
    PlottedRegion =  OptimalCuboid {}  # Region to plot

    NrOfPoints = 50 50 50              # Number of grid points in each direction
    NrOfCachedGrids = -1               # Nr of cached grids (speeds up things)
    Verbose = Yes                      # Wanna see a lot of messages?
  }

  DetailedXml = "detailed.xml"         # File containing the detailed xml output
                                       # of DFTB+
  EigenvecBin = "eigenvec.bin"         # File cointaining the binary eigenvecs


  # Definition of the basis
  Basis {
    Resolution = 0.01
    # Including mio-1-1.hsd. (If you use a set, which depends on other sets,
    # the wfc.*.hsd files for each required set must be included in a similar
    # way.)
    <<+ "../../slakos/wfc/wfc.mio-1-1.hsd"
  }


Some notes to the input:

* Option ``TotalChargeDensity`` controls the plotting of the total charge
  density. If turned on, the file ``wp-abs2.cube`` is created. # 选项 TotalChargeDensity 控制总电荷密度的绘图。如果打开，则会创建文件 wp-abs2.cube。

* Option ``TotalChargeDifference`` instructs Waveplot to plot the difference
  between the actual total charge density and the density you would obtain by
  summing up the densities of the neutral atoms. # 选项 TotalChargeDifference 指示 Waveplot 绘制实际总电荷密度与通过中性原子密度求和获得的密度之间的差异。

* Option ``ChargeDensity`` tells the code, that the charge distribution for some
  orbitals (specified later) should be plotted. Similarly, ``RealComponent``
  instructs Waveplot to create cube files for the real part of the one-electron
  wavefunctions for the specified orbitals. (For non-periodic systems the
  wavefunctions are real.)  # 选项 ChargeDensity 告诉代码，应该绘制某些轨道（稍后指定）的电荷分布。类似地，RealComponent 指示 Waveplot 为指定轨道的单电子波函数的实部创建立方体文件。 （对于非周期系统，波函数是实数。）

* Options ``PlottedSpins``, ``PlottedLevels`` (for periodic systems also
  ``PlottedKPoints``) controls the levels (orbitals) to plot.  In the current
  example we are plotting level 4 (is the HOMO of the water molecule) for all
  available spins. Since the DFTB+ calculation was spin unpolarised, we obtain
  only one plot for the HOMO in file ``wp-1-1-4-abs2.cube`` (1-1-4 in the file
  name indicates first K-point, first spin, 4th level).  #选项 PlottedSpins、PlottedLevels（对于周期系统还有 PlottedKPoints）控制要绘制的能级（轨道）。由于 DFTB+ 计算是自旋非极化的，因此我们仅在文件 wp-1-1-4-abs2.cube 中获得 HOMO 的一个图（文件名中的 1-1-4 表示第一个 K 点、第一个自旋、第 4 级） ）。对水分子而言，轨道4是HOMO轨道。

* The region to plot is selected with the option ``PlottedRegion``. Instead of
  specifying the box origin and box dimensions by hand, Waveplot can be
  instructed by using the ``OptimalCuboid`` method to take the smallest cuboid,
  which contains all the atoms and enough space around them, so that the
  wavefunctions are not leaking out of it. (For details and other options for
  ``PlottedRegion`` please consult the manual.)  The selected region in the
  example is sampled by a mesh of 50 by 50 by
  50.  (``NrOfPoints``)  # 使用 PlottedRegion 选项选择要绘制的区域。 OptimalCuboid 方法指示 Waveplot 获取最小的长方体，该长方体包含所有原子及其周围足够的空间，以便波函数不会从中泄漏。 示例中的选定区域是通过 50 x 50 x 50 的网格进行采样的。(NrOfPoints)

* The basis defintion (``Basis``) is made by including the file containing the
  appropriate wave function coefficient definitions.  You must make sure that
  you use the file for the same set, which you used during your DFTB+
  calculation. Here, the ``mio-1-1`` set was used for calculating the H2O
  molecule, and therefore the file ``wfc.mio-1-1.hsd`` is included.  # Basis需要指定包含适当的波函数系数定义的文件。波函数系数通常可以从与 Slater-Koster 文件相同的位置下载。

  The wavefuntion coefficients can be usually downloaded from the same place as
  the Slater-Koster files.


Output
------

::

  ================================================================================
       WAVEPLOT  0.2
  ================================================================================

  Interpreting input file 'waveplot_in.hsd'
  --------------------------------------------------------------------------------
  WARNING!
  -> The following 3 node(s) had been ignored by the parser:
  (1)
  Path: waveplot/Basis/C
  Line: 1-33 (File: wfc.mio-0-1.hsd)
  (2)
  Path: waveplot/Basis/N
  Line: 52-84 (File: wfc.mio-0-1.hsd)
  (3)
  Path: waveplot/Basis/S
  Line: 120-170 (File: wfc.mio-0-1.hsd)

  Processed input written as HSD to 'waveplot_pin.hsd'
  Processed input written as XML to 'waveplot_pin.xml'
  --------------------------------------------------------------------------------

  Doing initialisation

  Starting main program

  Origin
    -5.00000 -6.35306 -6.47114
  Box
    10.00000 0.00000 0.00000
    0.00000 11.08472 0.00000
    0.00000 0.00000 12.94228
  Spatial resolution [1/Bohr]:
    5.00000 4.51071 3.86331

  Total charge of atomic densities:    7.981973


   Spin KPoint  State  Action        Norm   W. Occup.
      1      1      1    read
      1      1      2    read
      1      1      3    read
      1      1      4    read

  Calculating grid

      1      1      1    calc    0.996855    2.000000
      1      1      2    calc    1.003895    2.000000
      1      1      3    calc    0.998346    2.000000
      1      1      4    calc    1.000053    2.000000
  File 'wp-1-1-4-abs2.cube' written
  File 'wp-1-1-4-real.cube' written
  File 'wp-abs2.cube' written

  Total charge:    7.998297

  File 'wp-abs2diff.cube' written

  ================================================================================

Some notes on the output:

* The warnings about unprocessed nodes appears, because the included file
  ``wfc.mio-0-1.hsd`` also contained wave function coefficients for elements (C,
  N, S), which are not present in the calculated system. Hence these extra
  definitions in the file were ignored. # 出现有关未处理元素的警告，因为包含的文件 wfc.mio-0-1.hsd 还包含元素 (C、N、S) 的波函数系数，而水分子中不存在这些系数。因此，这些不存在的元素波函数系数被忽略。

* The ``Total charge of atomic densities`` tells you the amount of charge found
  in the selected region, if atomic densities are superposed. This number should
  be approximately equal to the number of electrons in your system (here 8).
  There could be two reasons for a substantial deviation. Either the grid is not
  dense enough (option ``NrOfPoints``) or the box for the plotted region is too
  small or misplaced (``PlottedRegion``). # Total charge of atomic densities应该等于分子的总电子数，水分子总电子数为8，出现偏差的原因可能有两个。网格不够密集（选项 NrOfPoints）或绘制区域的框太小或位置错误（PlottedRegion）。

* The output files for the individual levels (charge density, real part,
  imaginary part) follow the naming convention `wp-KPOINT-SPIN-LEVEL-TYPE.cube`.  # 各个能级（电荷密度、实部、虚部）的输出文件遵循命名约定 wp-KPOINT-SPIN-LEVEL-TYPE.cube。

  The total charge and the total charge difference are stored in the files
  `wp-abs2.cube` and `wp-abs2diff.cube`, respectively.  # 总电荷和总电荷差分别存储在文件 wp-abs2.cube 和 wp-abs2diff.cube 中。


Visualising the results 
=======================

The volumetric data generated by Waveplot is in the Gaussian cube format and can
be visualized with several graphical tools (VMD, JMol, ParaView, ...). Below we
show the necessary steps to visualize it using VMD. (It refers to VMD version
1.8.6 and may differ in newer versions.) # cube数据可以使用VMD、JMol、ParaView 等图形工具可视化，这里使用VMD 1.8.6


Total charge distribution 总电荷分布
-------------------------
# 包含总电荷分布 wp-abs2.cube 的立方体文件可以通过使用 File|New Molecule 菜单读取。 成功加载后，VMD屏幕显示分子的骨架。

The cube file containing the total charge distribution ``wp-abs2.cube`` can be
read by using the ``File|New Molecule`` menu. VMD should automatically
recognise, that the file has the Gaussian cube format. After successful loading,
the VMD screen shows the skeleton of the molecule.

In order to visualise the charge distribution, the graphical representation of
the molecule has to be changed. This can be achieved by using the
``Graphics|Representations...`` submenu. The skeleton representation can be
turned to a CPK represenation (using balls and sticks) by selecting CPK for the
``Drawing method`` in the ``Graphical Representations`` dialog box. Then you
should create an additional representation (``Create Rep``) and change the
drawing method for it to be ``Isosurface``. The type of isosurface (``Draw``)
should be changed from ``Points`` to ``Solid Surface`` and instead of
``Box+Isosurface`` only ``Isosurface`` should be selected.  Then, by tuning the
``Isovalue`` one can select the isosurface to be plotted.
:numref:`fig_waveplot_h2odensity` was created using 0.100. (Display background
color had been set to white using the ``Graphics|Colors`` menu.)

  .. _fig_waveplot_h2odensity:
  .. figure:: /_figures/waveplot/h2o-density.png
     :align: center
     :alt: H2O density

     Total charge density for the H2O molecule, created by Waveplot, visualised
     by VMD.



Charge distribution difference 电荷分布差异
------------------------------

The charge distribution difference can be plotted in a similar way as the total
charge. One has to load the file ``wp-abs2diff.cube``. One should then, however,
make not one, but two additional graphical representations of the type
``Isosurface``. One of them should have positive isovalue, the other one a
negative one. The different isosurfaces can be colored in a different way by
using ``ColorID`` as coloring method and choosing different color values for the
different representations.

:numref:`fig_waveplot_h2odensdiff` demonstrates this for the water
molecule. Negative net populations were colored red, positive net populations
blue. One can clearly see, that there is a significant electron transfer from
the hydrogens to the oxygen (lone pair on the oxygen).

 .. _fig_waveplot_h2odensdiff:
 .. figure:: /_figures/waveplot/h2o-densitydiff.png
     :align: center
     :alt: H2O density difference

     Charge density difference (total density minus sum of atomic densities) for
     the H2O molecule, as created by Waveplot and visualised by VMD.


Molecular orbitals 分子轨道
------------------
# 绘制轨道的电荷密度(概率分布)，则数据只包含正值，因此只需要一个等值面表示(就像电荷分布一样)。而要绘制波函数的实部(或对于周期系统也是虚部)，则需要两个等值面表示法，一个用于正值，另一个用于负值(如电荷差)。

The plotting of molecular orbitals can be, depending which property is plotted,
done in the same way as the total charge distribution or the total charge
difference. If the charge density (probability distribution) of an orbital is
plotted, the data contains only positive values, therefore only one isosurface
representation is necessary (like for the charge distribution). If the real (or
for periodic systems also the imaginary) part of the wavefunction is to be
plotted, two isosurface representations are needed, one for the positive and one
for the negative values (like for the charge difference).

:numref:`fig_waveplot_h2ohomoabs2` shows the distribution of the electron
(wavefunction squared) for the HOMO, while :numref:`fig_waveplot_h2ohomoreal`
shows the HOMO wavefunction itself (blue - positive, red - negative). You can
easily recognise the p-type of the HOMO, positive on one side, negative on the
other side, a node plane in the middle.

  .. _fig_waveplot_h2ohomoabs2:
  .. figure:: /_figures/waveplot/h2o-homo-abs2.png
     :align: center
     :alt: H2O homo density

     Highest occupied molecular orbital of a water molecule (wavefunction
     square)

  .. _fig_waveplot_h2ohomoreal:
  .. figure:: /_figures/waveplot/h2o-homo-real.png
     :align: center
     :alt: H2O homo real

     Highest occupied molecular orbital of a water molecule (real part of the
     wavefunction).
