.. _dem_mosaic:

dem_mosaic
----------

The program ``dem_mosaic`` takes as input a list of DEM files and
creates a mosaic. By default, it seamlessly blends the DEMs where they
overlap.  It can also process the inputs to be combined in other ways,
as described below.

The input DEMs can either be set on the command line, or, if too many,
they can be listed in a text file (one per line) and that file can be
passed to the tool.

Usage
~~~~~
::

     dem_mosaic [options] <dem files> -o output_file_prefix

or::

     dem_mosaic [options] -l dem_files_list.txt -o output_file_prefix

Overview
~~~~~~~~

The output mosaic is written as non-overlapping tiles with desired tile
size, with the size set either in pixels or in georeferenced (projected)
units. The default tile size is large enough that normally the entire
mosaic is saved as one tile, named as::

    output_file_prefix-tile-0.tif

Alternatively, one can pass to the ``-o`` option an output file, such
as ``output.tif``. Then the mosaic will be written with this exact
name. (This will fail if the tool decides there is a need for more
than one tile.)

Individual tiles can be saved via the ``--tile-index`` option (the tool
displays the total number of tiles when it is being run). As such,
separate processes can be invoked for individual tiles for increased
robustness and perhaps speed.

By the default, the output mosaicked DEM will use the same grid size and
projection as the first input DEM. These can be changed via the ``--tr``
and ``--t_srs`` options. Also note the ``--tap`` option.

The default behavior is to blend the DEMs everywhere. If the option
``--priority-blending-length integer`` is invoked, the blending behavior
will be different. At any location, the pixel value of the DEM earliest
in the list present at this location will be kept, unless closer to the
boundary of that DEM than this blending length (measured in input DEM
pixels), only in the latter case blending will happen. This mode is
useful when blending several high-resolution "foreground" DEMs covering
small regions with larger "background" DEMs covering a larger extent.
Then, the pixels from the high-resolution DEMs are more desirable, yet
at their boundary these DEMs should blend into the background.

To obtain smoother blending when the input DEMs are quite different at
the boundary, one can increase ``--weights-blur-sigma`` and
``--weights-exponent``. The latter will result in weights growing slower
earlier and faster later. Some experimentation may be necessary, helped
for example by examining the weights used in blending; they can be
written out with ``--save-dem-weight integer``.

Instead of blending, ``dem_mosaic`` can compute the image of first,
last, minimum, maximum, mean, standard deviation, median, and count of
all encountered valid DEM heights at output grid points. For the
"first" and "last" operations, the order in which DEMs were passed in
is used. With any of these options, the tile names will be adjusted
accordingly. It is important to note that with these options blending
will not happen, since it is explicitly requested that particular
values of the input DEMs be used.

If the number of input DEMs is very large, the tool can fail as the
operating system may refuse to load all DEMs. In that case, it is
suggested to use the parameter ``--tile-size`` to break up the output
DEM into several large tiles, and to invoke the tool for each of the
output tiles with the option ``--tile-index``. Later, ``dem_mosaic`` can
be invoked again to merge these tiles into a single DEM.

If the DEMs have reasonably regular boundaries and no holes, smoother
blending may be obtained by using ``--use-centerline-weights``.

This tool can also apply hole-filling, smoothing, and pixel erosion at
boundary.

Examples
~~~~~~~~

Example 1. Blend DEMs::

     dem_mosaic dem1.tif dem2.tif -o blended.tif

Example 2. Read the DEMs from a list, and apply priority blending::

     echo dem1.tif dem2.tif > image_list.txt
     dem_mosaic -l image_list.txt --priority-blending-length 14 \
       -o priority_blended

Since an extension for the output was not specified, it will be saved
as ``priority_blended/tile-0.tif`` (there may be more than one tile if
the ``--tile-size`` parameter is set).

Example 3. Find the DEM of mean heights. No blending is used::

     dem_mosaic -l image_list.txt --mean -o mosaic

Example 4: Erode 3 pixels at the boundary::

     dem_mosaic --erode-length 3 input.tif -o output.tif

Example 5: Enforce that the grid is at integer multiples of grid size
(like the GDAL ``gdalwarp`` tool, :numref:`gdal_tools`)::

    dem_mosaic --tr 0.10 --tap input.tif -o output.tif

If the bounds of the output DEM from above are examined with
``gdalinfo`` (:numref:`gdal_tools`), they will be multiples of 0.05,
because each grid point is centered at an integer multiple of 0.10,
and extends for half a grid vertically and horizontally.

(Note that ``point2dem`` (:numref:`point2dem`) and ``mapproject``
(:numref:`mapproject`) create their outputs by default that way, and
if ``dem_mosaic`` is invoked on such datasets, it will respect the
input grid even without ``--tap`` being explicitly set.)

Command-line options
~~~~~~~~~~~~~~~~~~~~

-l, --dem-list-file <string>
    Text file listing the DEM files to mosaic, one per line.

-o, --output-prefix <string>
    Specify the output prefix. One or more tiles will be written
    with this prefix. Alternatively, an exact output file can be
    specified, with a .tif extension.

--tile-size <integer (default: 1000000)>
    The maximum size of output DEM tile files to write, in pixels.

--tile-index <integer>
    The index of the tile to save (starting from zero). When this
    program is invoked, it will print out how many tiles are there.
    Default: save all tiles.

--tile-list <string>
    List of tile indices (in quotes) to save. A tile index starts
    from 0.

--erode-length <integer (default: 0)>
    Erode the DEM by this many pixels at boundary.

--priority-blending-length <integer (default: 0)>
    If positive, keep unmodified values from the earliest available
    DEM except a band this wide measured in pixels inward of its
    boundary where blending with subsequent DEMs will happen.

--hole-fill-length <integer (default: 0)>
    Maximum dimensions of a hole in the DEM to fill in, in
    pixels.

--tr <double>
    Output grid size, that is, the DEM resolution in target
    georeferenced units per pixel. Default: use the same resolution as
    the first DEM to be mosaicked.

--t_srs <string>
    Specify the output projection (PROJ.4 string). Default: use the
    one from the first DEM to be mosaicked.

--t_projwin <double double double double>
    Limit the mosaic to this region, with the corners given in
    georeferenced coordinates (xmin ymin xmax ymax). Max is exclusive.
    See the ``--tap`` option if desired to apply addition adjustments
    to this extent.

--tap
    Let the output grid be at integer multiples of the grid size (like
    the default behavior of ``point2dem`` and ``mapproject``, and
    ``gdalwarp`` when invoked with ``-tap``, though the latter does
    not have the half-a-pixel extra extent this tool has). If this
    option is not set, the input grids determine the output grid.

--first
    Keep the first encountered DEM value (in the input order).

--last
    Keep the last encountered DEM value (in the input order).

--min
    Keep the smallest encountered DEM value.

--max
    Keep the largest encountered DEM value.

--mean
    Find the mean DEM value.

--stddev
    Find the standard deviation of DEM values.

--median
    Find the median DEM value (this can be memory-intensive, fewer threads are suggested).

--nmad
    Find the normalized median absolute deviation DEM value (this
    can be memory-intensive, fewer threads are suggested).

--count
    Each pixel is set to the number of valid DEM heights at that pixel.

--georef-tile-size <double>
    Set the tile size in georeferenced (projected) units (e.g.,
    degrees or meters).

--output-nodata-value <double>
    No-data value to use on output.  Default: use the one from the
    first DEM to be mosaicked.

--ot <string (default: Float32)>
    Output data type. Supported types: Byte, UInt16, Int16, UInt32,
    Int32, Float32. If the output type is a kind of integer, values
    are rounded and then clamped to the limits of that type.

--weights-blur-sigma <double (default: 5.0)>
    The standard deviation of the Gaussian used to blur the weights.
    Higher value results in smoother weights and blending.  Set to
    0 to not use blurring.

--weights-exponent <float (default: 2.0)>
    The weights used to blend the DEMs should increase away from
    the boundary as a power with this exponent. Higher values will
    result in smoother but faster-growing weights.

--use-centerline-weights
    Compute weights based on a DEM centerline algorithm. Produces
    smoother weights if the input DEMs don't have holes or complicated
    boundary.

--dem-blur-sigma <double (default: 0.0)>
    Blur the DEM using a Gaussian with this value of sigma.
    A larger value will blur more. Default: No blur.

--extra-crop-length <integer (default: 200)>
    Crop the DEMs this far from the current tile (measured in pixels)
    before blending them (a small value may result in artifacts).

--nodata-threshold <float>
    Values no larger than this number will be interpreted as no-data.

--force-projwin
    Make the output mosaic fill precisely the specified projwin,
    by padding it if necessary and aligning the output grid to the
    region.

--save-dem-weight <integer>
    Save the weight image that tracks how much the input DEM with
    given index contributed to the output mosaic at each pixel
    (smallest index is 0).

--save-index-map
    For each output pixel, save the index of the input DEM it came
    from (applicable only for ``--first``, ``--last``, ``--min``,
    ``--max``, ``--median``, and ``--nmad``). A text file with the
    index assigned to each input DEM is saved as well.

--threads <integer (default: 0)>
    Select the number of threads to use for each process. If 0, use
    the value in ~/.vwrc.

--tif-tile-size <integer (default: 256 256)>
    The dimensions of each block in the output image.

--cache-size-mb <integer (default = 1024)>
    Set the system cache size, in MB.

--no-bigtiff
    Tell GDAL to not create bigtiffs.

--tif-compress <None|LZW|Deflate|Packbits (default: LZW)>
    TIFF compression method.

-v, --version
    Display the version of software.

-h, --help
    Display this help message.
