.. index::
   single: Image
   single: Components; Image

The Image Component
===================

    The Image component provides tools to manipulate images.

Installation
------------

You can install the component in many different ways:

* :doc:`Install it via Composer </components/using_components>` (``symfony/image`` on `Packagist`_);
* Use the official Git repository (https://github.com/symfony/image).

.. include:: /components/require_autoload.rst.inc

Usage
-----

.. code-block:: php

    <?php

    $loader = new Symfony\Component\Image\Gd\Loader();
    // or
    $loader = new Symfony\Component\Image\Imagick\Loader();
    // or
    $loader = new Symfony\Component\Image\Gmagick\Loader();

    $size    = new Symfony\Component\Image\Image\Box(40, 40);

    $mode    = Symfony\Component\Image\Image\ImageInterface::THUMBNAIL_INSET;
    // or
    $mode    = Symfony\Component\Image\Image\ImageInterface::THUMBNAIL_OUTBOUND;

    $loader->open('/path/to/large_image.jpg')
        ->thumbnail($size, $mode)
        ->save('/path/to/thumbnail.png')
    ;


Introduction
------------

LoaderInterface_ (``Symfony\Component\Image\Image\LoaderInterface``) and its implementations is the main
entry point into Symfony Image.
You may think of it as a factory for ``Symfony\Component\Image\Image\ImageInterface`` as it is responsible for creating and opening
instances of it and also for instantiating ``Symfony\Component\Image\Image\FontInterface`` object.

The main piece of image processing functionality is concentrated in the ``ImageInterface`` implementations
(one per driver - e.g. ``Symfony\Component\Image\Imagick\Image``)

The main idea of Symfony Image is to avoid driver specific methods spill outside of this class and couple of other internal
interfaces (``Draw\DrawerInterface``), so that the filters and any other image manipulations can operate
on ``ImageInterface`` through its public API.

Open Existing Images
++++++++++++++++++++

To open an existing image, all you need is to instantiate an image loader and invoke ``LoaderInterface::open()``
with ``$path`` to image as the argument

.. code-block:: php

   <?php

   $loader = new Symfony\Component\Image\Gd\Loader();
   // or
   $loader = new Symfony\Component\Image\Imagick\Loader();

   $image = $loader->open('/path/to/image.jpg');

The ``LoaderInterface::open()`` method may throw one of the following exceptions:

* ``Symfony\Component\Image\Exception\InvalidArgumentException``
* ``Symfony\Component\Image\Exception\RuntimeException``

.. TIP::
   All exceptions thrown by Symfony Image implement ``Symfony\Component\Image\Exception\ExceptionInterface``

Now that you've opened an image, you can perform manipulations on it:

.. code-block:: php

   <?php

   use Symfony\Component\Image\Image\Box;
   use Symfony\Component\Image\Image\Point;

   $image->resize(new Box(15, 25))
      ->rotate(45)
      ->crop(new Point(0, 0), new Box(45, 45))
      ->save('/path/to/new/image.jpg');

Resize Images
+++++++++++++

Resize an image is very easy, just pass the box size you want as argument:

.. code-block:: php

   <?php

   use Symfony\Component\Image\Image\Box;
   use Symfony\Component\Image\Image\Point;

   $image->resize(new Box(15, 25))

You can also specify the filter you want as second argument:

.. code-block:: php

   <?php

   use Symfony\Component\Image\Image\Box;
   use Symfony\Component\Image\Image\Point;
   use Symfony\Component\Image\Image\ImageInterface;

   // resize with lanczos filter
   $image->resize(new Box(15, 25), ImageInterface::FILTER_LANCZOS);

Available filters are ``ImageInterface::FILTER_*`` constants.

.. NOTE::
   GD only supports ``ImageInterface::RESIZE_UNDEFINED`` filter.

Create New Images
+++++++++++++++++

Symfony Image also lets you create new, empty images. The following example creates an empty image of width
400px and height 300px:

.. code-block:: php

   <?php

   $size  = new Symfony\Component\Image\Image\Box(400, 300);
   $image = $loader->create($size);

You can optionally specify the fill color for the new image, which defaults to opaque white.
The following example creates a new image with a fully-transparent black background:

.. code-block:: php

   <?php

   $palette = new Symfony\Component\Image\Image\Palette\RGB();
   $size  = new Symfony\Component\Image\Image\Box(400, 300);
   $color = $palette->color('#000', 100);
   $image = $loader->create($size, $color);

Save Images
+++++++++++

Images are saved given a path and optionally options.

The following example opens a Jpg image and saves it as Png format :

.. code-block:: php

   <?php

   $loader = new Symfony\Component\Image\Imagick\Loader();

   $loader->open('/path/to/image.jpg')
      ->save('/path/to/image.png');

Three options groups are currently supported : quality, resolution and flatten.

.. TIP::
   Default values are 75 for Jpeg quality, 7 for Png compression level and 72 dpi for x/y-resolution.

.. NOTE::
   GD does not support resolution options group

The following example demonstrates the basic quality settings.

.. code-block:: php

   <?php

   $loader = new Symfony\Component\Image\Imagick\Loader();

   $loader->open('/path/to/image.jpg')
      ->save('/path/to/image.jpg', array('jpeg_quality' => 50)) // from 0 to 100
      ->save('/path/to/image.png', array('png_compression_level' => 9)); // from 0 to 9

The following example opens a Jpg image and saves it with it with 150 dpi horizontal resolution
and 120 dpi vertical resolution.

.. code-block:: php

   <?php

   use Symfony\Component\Image\Image\ImageInterface;

   $loader = new Symfony\Component\Image\Imagick\Loader();

   $options = array(
       'resolution-units' => ImageInterface::RESOLUTION_PIXELSPERINCH,
       'resolution-x' => 150,
       'resolution-y' => 120,
       'resampling-filter' => ImageInterface::FILTER_LANCZOS,
   );

   $loader->open('/path/to/image.jpg')->save('/path/to/image.jpg', $options);

.. NOTE::
   You **MUST** provide a unit system when setting resolution values.
   There are two available unit systems for resolution :
   ``ImageInterface::RESOLUTION_PIXELSPERINCH`` and ``ImageInterface::RESOLUTION_PIXELSPERCENTIMETER``.

The flatten option is used when dealing with multi-layers images (see the
layers section for information). Image are saved flatten by default,
you can avoid this by explicitly set this option to ``false`` when saving :

.. code-block:: php

   <?php

   use Symfony\Component\Image\Image\Box;
   use Symfony\Component\Image\Image\ImageInterface;
   use Symfony\Component\Image\Imagick\Loader;

   $loader = new Loader();

   $loader->open('/path/to/animated.gif')
           ->resize(new Box(320, 240))
           ->save('/path/to/animated-resized.gif', array('flatten' => false));

.. TIP::
   You **SHOULD** not flatten image only for animated gif and png images.

Of course, you can combine options :

.. code-block:: php

   <?php

   use Symfony\Component\Image\Image\ImageInterface;

   $loader = new Symfony\Component\Image\Imagick\Loader();

   $options = array(
       'resolution-units' => ImageInterface::RESOLUTION_PIXELSPERINCH,
       'resolution-x' => 300,
       'resolution-y' => 300,
       'jpeg_quality' => 100,
   );

   $loader->open('/path/to/image.jpg')->save('/path/to/image.jpg', $options);

Show Images
+++++++++++

Images are shown (i.e. outputs the image content) given a format and optionally options.

The following example shows a Jpg image:

.. code-block:: php

   <?php

   $loader = new Symfony\Component\Image\Imagick\Loader();

   $loader->open('/path/to/image.jpg')
      ->show('jpg');

.. NOTE::
   This will send a "Content-type" header.

It supports the same options groups as for the save method.

For example:

.. code-block:: php

   <?php

   $loader = new Symfony\Component\Image\Imagick\Loader();

   $options = array(
      'resolution-units' => ImageInterface::RESOLUTION_PIXELSPERINCH,
      'resolution-x' => 300,
      'resolution-y' => 300,
      'jpeg_quality' => 100,
   );

   $loader->open('/path/to/image.jpg')
      ->show('jpg', $options);

Image Watermarking
++++++++++++++++++

Here is a simple way to add a watermark to an image :

.. code-block:: php

    <?php

    $watermark = $loader->open('/my/watermark.png');
    $image     = $loader->open('/path/to/image.jpg');
    $size      = $image->getSize();
    $wSize     = $watermark->getSize();

    $bottomRight = new Symfony\Component\Image\Image\Point($size->getWidth() - $wSize->getWidth(), $size->getHeight() - $wSize->getHeight());

    $image->paste($watermark, $bottomRight);

An Image Collage
++++++++++++++++

Assume we were given the not-so-easy task of creating a four-by-four collage of 16 student portraits for a school
yearbook.  Each photo is 30x40 px and we need four rows and columns in our collage, so the final product will be 120x160 px.

Here is how we would approach this problem with Symfony Image.

.. code-block:: php

   <?php

   use Symfony\Component\Image;

   // make an empty image (canvas) 120x160px
   $collage = $loader->create(new Image\Image\Box(120, 160));

   // starting coordinates (in pixels) for inserting the first image
   $x = 0;
   $y = 0;

   foreach (glob('/path/to/people/photos/*.jpg') as $path) {
      // open photo
      $photo = $loader->open($path);

      // paste photo at current position
      $collage->paste($photo, new Image\Image\Point($x, $y));

      // move position by 30px to the right
      $x += 30;

      if ($x >= 120) {
         // we reached the right border of our collage, so advance to the
         // next row and reset our column to the left.
         $y += 40;
         $x = 0;
      }

      if ($y >= 160) {
         break; // done
      }
   }

   $collage->save('/path/to/collage.jpg');

Image Reflection Filter
+++++++++++++++++++++++

.. code-block:: php

   <?php

   class ReflectionFilter implements Symfony\Component\Image\Filter\FilterInterface
   {
       private $loader;

       public function __construct(Symfony\Component\Image\Image\LoaderInterface $loader)
       {
           $this->loader = $loader;
       }

       public function apply(Symfony\Component\Image\Image\ImageInterface $image)
       {
           $size       = $image->getSize();
           $canvas     = new Symfony\Component\Image\Image\Box($size->getWidth(), $size->getHeight() * 2);
           $reflection = $image->copy()
               ->flipVertically()
               ->applyMask($this->getTransparencyMask($image->palette(), $size))
           ;

           return $this->loader->create($canvas, $image->palette()->color('fff', 100))
               ->paste($image, new Symfony\Component\Image\Image\Point(0, 0))
               ->paste($reflection, new Symfony\Component\Image\Image\Point(0, $size->getHeight()));
       }

       private function getTransparencyMask(Symfony\Component\Image\Image\Palette\PaletteInterface $palette, Symfony\Component\Image\Image\BoxInterface $size)
       {
           $white = $palette->color('fff');
           $fill  = new Symfony\Component\Image\Image\Fill\Gradient\Vertical(
               $size->getHeight(),
               $white->darken(127),
               $white
           );

           return $this->loader->create($size)
               ->fill($fill)
           ;
       }
   }

   $loader = new Symfony\Component\Image\Gd\Loader();
   $filter  = new ReflectionFilter($loader);

   $filter->apply($loader->open('/path/to/image/to/reflect.png'))
      ->save('/path/to/processed/image.png')
   ;

The architecture is very flexible, as the filters don't need any processing logic other than calculating the variables
based on some settings and invoking the corresponding method, or sequence of methods, on the ``ImageInterface`` implementation.

The ``Transformation`` object is an example of a composite filter, representing a stack or queue of filters, that get
applied to an Image upon application of the ``Transformation`` itself.

Metadata
--------

Metadata are read with a ``Symfony\Component\Image\Image\Metadata\MetadataReaderInterface`` and
accessible through ``Symfony\Component\Image\Image\ImageInterface::metadata`` method that returns
a ``Symfony\Component\Image\Image\Metadata\MetadataBag`` object:

.. code-block:: php

    <?php

    use Symfony\Component\Image\Image\Metadata\ExifMetadataReader;

    $image = $loader->open('/path/to/image.jpg');
    $metadata = $image->metadata();

    // prints '/path/to/image.jpg'
    print($metadata['filename']);


Change the metadata reader
++++++++++++++++++++++++++

Symfony Image comes bundled with two metadata readers: ``Symfony\Component\Image\Image\Metadata\DefaultMetadataReader``
and ``Symfony\Component\Image\Image\Metadata\ExifMetadataReader``.

Symfony Image uses the default metadata reader by default. You can easily switch to
the one you want using ``Symfony\Component\Image\Image\LoaderInterface::setMetadataReader``
method.

.. code-block:: php

    <?php

    use Symfony\Component\Image\Image\Metadata\ExifMetadataReader;

    $loader->setMetadataReader(new ExifMetadataReader());

Default Metadata Reader
+++++++++++++++++++++++

The default metadata reader is a basic reader that stores original information
about the resource.

.. code-block:: php

    <?php

    use Symfony\Component\Image\Image\Metadata\DefaultMetadataReader;

    $image = $loader
        ->setMetadataReader(new DefaultMetadataReader())
        ->open('chenille.jpg');
    $metadata = $image->metadata();

    var_dump($metadata->toArray());

The previous code might produce such output:

.. code-block:: none

    array(2) {
      'filepath' =>
      string(60) "/Users/romainneutron/Documents/workspace/symfony-image/chenille.jpg"
      'uri' =>
      string(12) "chenille.jpg"
    }

Exif Metadata Reader
++++++++++++++++++++

Exif Metadata Reader gives the same base information as the default metadata reader
and adds exif data provided by the Exif extension.

.. note::

    Using the exif metadata reader adds a significant overhead to image processing.

.. code-block:: php

    <?php

    use Symfony\Component\Image\Image\Metadata\ExifMetadataReader;

    $image = $loader
        ->setMetadataReader(new ExifMetadataReader())
        ->open('chenille.jpg');
    $metadata = $image->metadata();

    var_dump($metadata->toArray());

The previous code should produce this output:

.. code-block:: none

    array(37) {
      'filepath' =>
      string(60) "/Users/romainneutron/Documents/workspace/symfony-image/chenille.jpg"
      'uri' =>
      string(12) "chenille.jpg"
      'exif.ExposureTime' =>
      string(5) "1/120"
      'exif.FNumber' =>
      string(4) "11/5"
      'exif.ExposureProgram' =>
      int(2)
      'exif.ISOSpeedRatings' =>
      int(40)
      'exif.ExifVersion' =>
      string(4) "0221"
      'exif.DateTimeOriginal' =>
      string(19) "2014:04:06 16:11:59"
      'exif.DateTimeDigitized' =>
      string(19) "2014:04:06 16:11:59"
      'exif.ComponentsConfiguration' =>
      string(4) "\000"
      'exif.ShutterSpeedValue' =>
      string(9) "9488/1373"
      'exif.ApertureValue' =>
      string(9) "7801/3429"
      'exif.BrightnessValue' =>
      string(8) "4457/710"
      'exif.MeteringMode' =>
      int(3)
      'exif.Flash' =>
      int(16)
      'exif.FocalLength' =>
      string(6) "103/25"
      'exif.ColorSpace' =>
      int(1)
      'exif.ExifImageWidth' =>
      int(2048)
      'exif.ExifImageLength' =>
      int(1536)
      'exif.SensingMethod' =>
      int(2)
      'exif.ExposureMode' =>
      int(0)
      'exif.WhiteBalance' =>
      int(0)
      'exif.FocalLengthIn35mmFilm' =>
      int(30)
      'exif.SceneCaptureType' =>
      int(0)
      'exif.UndefinedTag:0xA433' =>
      string(5) "Apple"
      'exif.UndefinedTag:0xA434' =>
      string(34) "iPhone 5s back camera 4.12mm f/2.2"
      'ifd0.Make' =>
      string(5) "Apple"
      'ifd0.Model' =>
      string(9) "iPhone 5s"
      'ifd0.XResolution' =>
      string(4) "72/1"
      'ifd0.YResolution' =>
      string(4) "72/1"
      'ifd0.ResolutionUnit' =>
      int(2)
      'ifd0.Software' =>
      string(5) "7.0.4"
      'ifd0.DateTime' =>
      string(19) "2014:04:06 16:11:59"
      'ifd0.YCbCrPositioning' =>
      int(1)
      'ifd0.Exif_IFD_Pointer' =>
      int(192)
      'ifd0.GPS_IFD_Pointer' =>
      int(1486)
    }

Create your own metadata reader
+++++++++++++++++++++++++++++++

Any metadata reader must implement ``Symfony\Component\Image\Image\Metadata\MetadataReaderInterface``.
However it's easier to extend ``Symfony\Component\Image\Image\Metadata\AbstractMetadataReader``
to avoid missing things and focus on the purpose of the reader.

Here's an example of a metadata reader that retrieves posix access information from
a file:

.. code-block:: php

    <?php

    use Symfony\Component\Image\Image\Metadata\AbstractMetadataReader;

    class PosixMetadataReader extends AbstractMetadataReader
    {
        /**
         * {@inheritdoc}
         */
        protected function extractFromFile($file)
        {
            // if file is not local, forget it
            if (!stream_is_local($file)) {
                return array();
            }

            return array(
                'access' => posix_access($file),
            );
        }

        /**
         * {@inheritdoc}
         */
        protected function extractFromData($data)
        {
            // posix informations about raw data in non-sense
            return array();
        }

        /**
         * {@inheritdoc}
         */
        protected function extractFromStream($resource)
        {
            if (!stream_is_local($file)) {
                return array();
            }

            if (false !== $data = @stream_get_meta_data($resource)) {
                return array(
                    'access' => posix_access($data['uri']),
                );
            }

            return array();
        }
    }



Coordinates
-----------


The coordinate system use by Symfony Image is very similar to Cartesian Coordinate System, with some exceptions:

* Coordinate system starts at x,y (0,0), which is the top left corner and extends to right and bottom accordingly
* There are no negative coordinates, a point must always be bound to the box its located at, hence 0,0 and greater
* Coordinates of the point are relative its parent bounding box

Classes
+++++++

The whole coordinate system is represented in a handful of classes, but most importantly - its interfaces:

* ``Symfony\Component\Image\Image\PointInterface`` - represents a single point in a bounding box

* ``Symfony\Component\Image\Image\BoxInterface`` - represents dimensions (width, height)

PointInterface
++++++++++++++

Every coordinate contains the following methods:

* ``->getX()`` - returns horizontal position of the coordinate

* ``->getY()`` - returns vertical position of a coordinate

* ``->in(BoxInterface $box)`` - returns ``true`` if current coordinate appears to be inside of a given bounding ``$box``

* ``->__toString()`` - returns string representation of the current ``PointInterface``, e.g. ``(0, 0)``

Center coordinate
+++++++++++++++++

It is very well known use case when a coordinate is supposed to represent a center of something.

As part of showing off OO approach to image processing, I added a simple implementation of the core
``Symfony\Component\Image\Image\PointInterface``, which can be found at ``Symfony\Component\Image\Image\Point\Center``.
The way it works is simple, it expects and instance of ``Symfony\Component\Image\Image\BoxInterface``
in its constructor and calculates the center position based on that.

.. code-block:: php

    <?php

    $size = new Symfony\Component\Image\Image\Box(50, 50);

    $center = new Symfony\Component\Image\Image\Point\Center($size);

    var_dump(array(
        'x' => $center->getX(),
        'y' => $center->getY(),
    ));

    // would output position of (x,y) 25,25

BoxInterface
++++++++++++

Every box or image or shape has a size, size has the following methods:

* ``->getWidth()`` - returns integer width

* ``->getHeight()`` - returns integer height

* ``->scale($ratio)`` - returns a new ``BoxInterface`` instance with each side multiplied by ``$ratio``

* ``->increase($size)`` - returns a new ``BoxInterface``, with given ``$size`` added to each side

* ``->contains(BoxInterface $box, PointInterface $start = null)`` - checks that the given ``$box`` is contained inside the current ``BoxInterface`` at ``$start`` position. If no ``$start`` position is given, its assumed to be (0,0)

* ``->square()`` - returns integer square of current ``BoxInterface``, useful for determining total number of pixels in a box for example

* ``->__toString()`` - returns string representation of the current ``BoxInterface``, e.g. ``100x100 px``

* ``->widen($width)`` - resizes box to given width, constraining proportions and returns the new box

* ``->heighten($height)`` - resizes box to given height, constraining proportions and returns the new box



Drawing
-------



Symfony Image also provides a fully-featured drawing API, inspired by Python's PIL.
To use the api, you need to get a drawer instance from you current image instance, using ``ImageInterface::draw()`` method.

Example
+++++++

.. code-block:: php

    <?php
    $palette = new Symfony\Component\Image\Image\Palette\RGB();

    $image = $loader->create(new Box(400, 300), $palette->color('#000'));

    $image->draw()
        ->ellipse(new Point(200, 150), new Box(300, 225), $image->palette()->color('fff'));

    $image->save('/path/to/ellipse.png');

The above example would draw an ellipse on a black 400x300px image, of white color. It would place the ellipse in the
center of the image, and set its larger radius to 300px, with a smaller radius of 225px.
You could also make the ellipse filled,  by passing `true` as the last parameter.

Text
++++

As you've noticed from ``DrawerInterface::text()``, there is also ``Font`` class.
This class is a simple value object, representing the font. To construct a font, you have to pass the
``$file`` string (path to font file), ``$size`` value (integer value, representing size points) and
``$color`` (``Symfony\Component\Image\Image\Palette\Color\ColorInterface`` instance). After you have a font instance,
you can use one of its three methods to inspect any of the values it's been constructed with:

* ``->getFile()`` - returns font file path

* ``->getSize()`` - returns integer size in points (e.g. 10pt = 10)

* ``->getColor()`` - returns ``Symfony\Component\Image\Image\Palette\Color\ColorInterface`` instance, representing current font color

* ``->box($string, $angle = 0)`` - returns ``Symfony\Component\Image\Image\BoxInterface`` instance, representing the estimated size of the ``$string`` at the given ``$angle`` on the image




Colors
------

Symfony Image provides a fully-featured colors API with the Palette object :

Palette Class
+++++++++++++

Every image in Symfony Image is attached to a Palette. The palette handles the colors.
Symfony Image provides two palettes :

.. code-block:: php

   <?php

   $palette = new Symfony\Component\Image\Image\Palette\RGB();
   // or
   $palette = new Symfony\Component\Image\Image\Palette\CMYK();

When creating a new Image, the default RGB palette is used. It can be easily customized.
For example the following code creates a new Image object with a white background and a CMYK palette.

.. code-block:: php

   <?php

   $palette = new Symfony\Component\Image\Image\Palette\CMYK();
   $loader->create(new Symfony\Component\Image\Image\Box(10, 10), $palette->color('#FFFFFF'));

You can switch your palette at any moment, for example to turn a CMYK image in RGB mode :

.. code-block:: php

   <?php

   $image = $loader->open('my-cmyk-jpg.jpg');
   $image->usePalette(new Symfony\Component\Image\Image\Palette\RGB())
         ->save('my-rgb-jpg.jpg');

.. NOTE::
    Switching to a palette is the same a changing the colorspace.

About colorspaces support
+++++++++++++++++++++++++

Drivers do not handle colorspace the same way.
Whereas GD only supports RGB images, Imagick supports CMYK, RGB and Grayscale
colorspaces. Gmagick only supports CMYK and RGB colorspaces.

Color Class
+++++++++++

Color is a class in Symfony Image, and is created through a palette with two arguments in its constructor:
the RGB color code and a transparency percentage.
The following examples are equivalent ways of defining a fully-transparent white color.

.. code-block:: php

   <?php

   $white = $palette->color('fff', 100);
   $white = $palette->color('ffffff', 100);
   $white = $palette->color('#fff', 100);
   $white = $palette->color('#ffffff', 100);
   $white = $palette->color(0xFFFFFF, 100);
   $white = $palette->color(array(255, 255, 255), 100);

.. NOTE::
    CMYK colors does not support alpha parameters.

After you have instantiated an RGB color, you can easily get its Red, Green, Blue and Alpha (transparency) values:

.. code-block:: php

   <?php

   var_dump(array(
      'R' => $white->getRed(),
      'G' => $white->getGreen(),
      'B' => $white->getBlue(),
      'A' => $white->getAlpha()
   ));

The same behavior is available for CMYK colors :

.. code-block:: php

   <?php

   var_dump(array(
      'C' => $white->getCyan(),
      'M' => $white->getMagenta(),
      'Y' => $white->getYellow(),
      'K' => $white->getKeyline()
   ));

Profile Class
+++++++++++++

You can apply ICC profile on any Image class with the ``profile`` method :

.. code-block:: php

   <?php

   $profile = Symfony\Component\Image\Image\Profile::fromPath('your-ICC-profile.icc');
   $image->profile($profile)
         ->save('my-rgb-jpg-profiled.jpg');



Layers
------





``ImageInterface`` provides an access for multi-layers image such as PSD files
or animated gif.
By calling the ``layers()`` method, you will get an iterable layer collection
implementing the ``LayersInterface``. As you will see, a layer implements
``ImageInterface``

Disclaimer
++++++++++

Symfony Image is a fluent API to use Imagick, Gmagick or GD driver. These drivers
do not handle all multi-layers formats equally. For example :

 * PSD format should be flatten before being saved. (libraries would split it
   into different files),
 * animated gif must not be flatten otherwise the animation would be lost.
 * Tiff files should be split in multiple files or the result might be a pile
   of HD and thumbnail
 * GD does not support layers.

You have to run tests against the formats you are using and their support by
the driver you want before deploying in production.

Layers Manipulation
+++++++++++++++++++

Symfony Image ``LayersInterface`` implements PHP's ArrayAccess, IteratorAggregate and
Countable interfaces.

This provides many ways to manipulate layers.

Count Layers
++++++++++++

.. code-block:: php

    <?php
    $image = $loader->open('image.jpg');

    echo "Image contains " . count($image->layers()) . " layers";

Layers Iterations
+++++++++++++++++

.. code-block:: php

    <?php
    $image = $loader->open('image.jpg');

    foreach ($image->layers() as $layer) {
        // ...
    }

Layers Manipulation
+++++++++++++++++++

Symfony Image provides an object oriented interface to manipulate layers :

.. code-block:: php

    <?php
    $image = $loader->open('image.jpg');
    $layers = $image->layers();

    $layers->get(0)->save('layer-0.jpg');         // access a layer
    $layers->set(0, $loader->open('image2.jpg')); // set layer at offset 0
    $layers->add($loader->open('image3.jpg'));    // push a new layer in layers
    $layers->remove(1);                           // removes a layer at offset
    $layers->has(2);                              // test is a layer is present

You can also manipulate them like arrays :

.. code-block:: php

    <?php
    $image = $loader->open('image.jpg');
    $layers = $image->layers();

    $layers[0]->save('layer-0.jpg');          // access a layer
    $layers[0] = $loader->open('image2.jpg'); // set layer at offset 0
    $layers[]  = $loader->open('image3.jpg'); // push a new layer in layers
    unset($layers[1]);                        // removes a layer at offset
    isset($layers[2]);                        // test is a layer is present

.. NOTE::
    Layers can be compared as indexed arrays. You should not use string keys.

Generate Animated gif
+++++++++++++++++++++

Symfony Image provides a simple way to generate animated gif by manipulating layers :

.. code-block:: php

    <?php

    $image = $loader->open('image.jpg');

    $image->layers()
        ->add($loader->open('image2.jpg'))
        ->add($loader->open('image3.jpg'))
        ->add($loader->open('image4.jpg'))
        ->add($loader->open('image5.jpg'));

    $image->save('animated.gif', array(
        'animated' => true,
    ));

When saving an animated gif, you are only required to use the ``animated``
option.

There are more options that can customize the output, look at the following
example :


.. code-block:: php

    <?php

    $image->save('animated.gif', array(
        'animated'       => true,
        'animated.delay' => 500, // delay in ms
        'animated.loops' => 0,   // number of loops, 0 means infinite
    ));

Animated gif frame manipulation
+++++++++++++++++++++++++++++++

Resizing an animated cats.gif file :

.. code-block:: php

    <?php
    $image = $loader->open('cats.gif');

    $image->layers()->coalesce();
    foreach ($image->layers() as $frame) {
        $frame->resize(new Box(100, 100));
    }

    $image->save('resized-cats.gif', array('animated' => true));

The layers (frames) should be coalesced so that they are all in line with each other.
Otherwise you may end up with strange artifacts due to how animated GIFs *can* work.
Without going into too much detail, think of it as each frame being a patch to the previous one.
Also note that not the image, but each frame is resized. This again has to do with
how animated GIFs work. Not every frame has to be the full image.

The following example extract all frames of the cats.gif file :

.. code-block:: php

    <?php

    $i = 0;
    foreach ($loader->open('cats.gif')->layers() as $layer) {
        $layer->save("frame-$i.png");
        $i++;
    }

This one adds some text on frames :

.. code-block:: php

    <?php

    $image = $loader->open('cats.gif');
    $i = 0;
    foreach ($image->layers() as $layer) {
        $layer->draw()
              ->text($i, new Font('coolfont.ttf', 12, $image->palette()->color('white')), new Point(10, 10));
        $i++;
    }

    // save modified animation
    $image->save('cats-modified.gif', array('flatten' => 'false'));





Effects
-------




Symfony Image also provides a fully-featured effects API.
To use the api, you need to get an effect instance from you current image
instance, using ``ImageInterface::effects()`` method.

.. code-block:: php

    <?php

    $image = $loader->open('portrait.jpeg');

    $image->effects()
        ->negative()
        ->gamma(1.3);

    $image->save('negative-portrait.png');

The above example would open a "portrait.jpeg" image, invert the colors, then
corrects the gamma with a parameter of 1.3 then saves it to a new file
"negative-portrait.png".

.. NOTE::
    As you can notice, all effects are chainable.


The current Effects API currently supports these effects :

Negative
++++++++

The negative effect inverts the color of an image :

.. code-block:: php

    <?php

    $image = $loader->open('portrait.jpeg');

    $image->effects()
        ->negative();

    $image->save('negative-portrait.png');

Gamma correction
++++++++++++++++

Apply a gamma correction. It takes one float argument, the correction parameter.

.. code-block:: php

    <?php

    $image = $loader->open('portrait.jpeg');

    $image->effects()
        ->gamma(0.7);

    $image->save('negative-portrait.png');

Grayscale
+++++++++

Create a grayscale version of the image.

.. code-block:: php

    <?php

    $image = $loader->open('portrait.jpeg');

    $image->effects()
        ->grayscale();

    $image->save('grayscale-portrait.png');

Colorize
++++++++

Colorize the image. It takes one ``Symfony\Component\Image\Image\Palette\Color\ColorInterface`` argument, which represents the color applied on top of the image.

This feature only works with the Gd and Imagick drivers.

.. code-block:: php

    <?php

    $image = $loader->open('portrait.jpeg');

    $pink = $image->palette()->color('#FF00D0');

    $image->effects()
        ->colorize($pink);

    $image->save('pink-portrait.png');

Blur
++++

Blur the image. It takes a string argument, which represent the sigma used for
Imagick and Gmagick functions (defaults to 1).

.. code-block:: php

    <?php

    $image = $loader->open('portrait.jpeg');

    $image->effects()
        ->blur(3);

    $image->save('blurred-portrait.png');

.. NOTE::
    Sigma value has no effect on GD driver. Only GD's IMG_FILTER_GAUSSIAN_BLUR filter is applied instead.


Filters
-------


Image Transformations, aka Lazy Processing
++++++++++++++++++++++++++++++++++++++++++

Sometimes we're not comfortable with opening an image inline, and would like to apply some pre-defined
operations in the lazy manner.

For that, Symfony Image provides so-called image transformations.

Image transformation is implemented via the ``Filter\Transformation`` class, which mostly conforms to ``ImageInterface``
and can be used interchangeably with it.
The main difference is that transformations may be stacked and performed on a real ``ImageInterface`` instance later
using the ``Transformation::apply()`` method.

Example of a naive thumbnail implementation:

.. code-block:: php

    <?php

    $transformation = new Symfony\Component\Image\Filter\Transformation();

    $transformation->thumbnail(new Symfony\Component\Image\Image\Box(30, 30))
        ->save('/path/to/resized/thumbnail.jpg');

    $transformation->apply($loader->open('/path/to/image.jpg'));

The result of ``apply()`` is the modified image instance itself, so if we wanted to create a mass-processing
thumbnail script, we would do something like the following:

.. code-block:: php

    <?php

    $transformation = new Symfony\Component\Image\Filter\Transformation();

    $transformation->thumbnail(new Symfony\Component\Image\Image\Box(30, 30));

    foreach (glob('/path/to/lots/of/images/*.jpg') as $path) {
        $transformation->apply($loader->open($path))
            ->save('/path/to/resized/'.md5($path).'.jpg');
    }

The ``Filter\Transformation`` class itself is simply a very specific implementation of ``FilterInterface``,
which is a more generic interface, that lets you pre-define certain operations and variable calculations
and apply them to an ``ImageInterface`` instance later.

Filter Application Order
++++++++++++++++++++++++

Normally filters are applied in the order that they are added to the transformation.
However, sometimes we want certain filters to always apply first, and others to always apply last,
for example always apply a crop before applying a border.
You can do this by specifying a priority when passing a filter to the ``add()`` method:

.. code-block:: php

    <?php

    $transformation = new Symfony\Component\Image\Filter\Transformation();

    $transformation->add(new Symfony\Component\Image\Filter\Basic\Crop($point, $size), -10); //this filter has priority -10 and applies early
    $transformation->add(new Symfony\Component\Image\Filter\Advanced\Border($color), 10); //this filter has priority 10 and applies late
    $transformation->add(new Symfony\Component\Image\Filter\Basic\Rotate($angle)); //this filter has default priority 0 and applies in between

    //filters with equal priority will still be applied in the order they were added

This is especially useful when you add filters based on user-input.

Filters
+++++++

As we already know, ``Filter\Transformation`` is just a very special case of ``Filter\FilterInterface``.

Filter is a set of operations, calculations, etc., that can be applied to an ``ImageInterface`` instance using ``Filter\FilterInterface::apply()`` method.

Right now only basic filters are available - they simply forward the call to ``ImageInterface`` implementation itself, more filters coming soon...


