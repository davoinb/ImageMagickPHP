ImageMagickPHP
===============

[![SensioLabsInsight](https://insight.sensiolabs.com/projects/ff8b439c-772a-495e-9780-4e8e8e451254/mini.png)](https://insight.sensiolabs.com/projects/ff8b439c-772a-495e-9780-4e8e8e451254)
[![Build Status](https://travis-ci.org/Orbitale/ImageMagickPHP.png)](https://travis-ci.org/Orbitale/ImageMagickPHP)
[![Coverage Status](https://coveralls.io/repos/Orbitale/ImageMagickPHP/badge.png)](https://coveralls.io/r/Orbitale/ImageMagickPHP)

An ImageMagick "exec" component for PHP apps.

Installation
===============

Install with [Composer](https://getcomposer.org/), it's the best packages manager you can have :

```shell
composer require orbitale/imagemagick-php
```

Requirements
===============

* PHP 7.1 or higher
* [ImageMagick](https://www.imagemagick.org/) has to be installed on your server, and the binaries must be executable by the user running the PHP process.

Settings
===============

There are not many settings, but when you instantiate a new `Command` object, you may specify ImageMagick's executable directory directly in the constructor, for example :

```php
use Orbitale\Component\ImageMagick\Command;

// Default directory for many Linux distributions:
$command = new Command('/usr/bin');

// Or in Windows, depending of the install directory:
$command = new Command('C:\ImageMagick');

// If it is available in the global scope for the user running the script:
$command = new Command();
```

The constructor will automatically search for the `convert` (v6) or `magick` (v7) executable, test it, and throw an exception if it's not available.

*Note:* `convert` is looked up only because it's the most used binary. With ImageMagick 7, you can install the `magick` binary that will wrap all other IM commands (like `magick convert ...` instead of `convert ...`). This is better for encapsulation and to avoid conflicts with other `convert` tools.
Of course, ImageMagick 7 is recommended. 

/!\ Make sure your ImageMagick executables have the "+x" chmod option, and that the user has the rights to execute it.

Usage
===============

### Basic image type converter with ImageMagick's basic logo

Read the comments :

```php
require_once 'vendor/autoload.php';

use Orbitale\Component\ImageMagick\Command;

// Create a new command
$command = new Command();

$response = $command
    // The command will search for the "logo.png" file. If it does not exist, it will throw an exception.
    // If it does, it will create a new command with this source image.
    ->convert('logo.png')

    // The "output()" method will append "logo.gif" at the end of the command-line instruction as a filename.
    // This way, we can continue writing our command without appending "logo.gif" ourselves.
    ->output('logo.gif')

    // At this time, the command shall look like this :
    // $ "{ImageMagickPath}convert" "logo.png" "logo.gif"

    // Then we run the command by using "exec()" to get the CommandResponse
    ->run()
;

if ($response->isSuccessful()) {
    // If it has not failed, then we simply send it to the buffer
    header('Content-type: image/gif');
    echo file_get_contents('logo.gif');
}
```

### Resizing an image

```php
require_once 'vendor/autoload.php';

use Orbitale\Component\ImageMagick\Command;

// Create a new command
$command = new Command();

$response = $command

    ->convert('background.jpeg')
    
    // We'll use the same output as the input, therefore will overwrite the source file after resizing it.
    ->output('background.jpeg')

    // The "resize" method allows you to add a "Geometry" operation.
    // It must fit to the "Geometry" parameters in the ImageMagick official documentation (see links below & phpdoc)
    ->resize('50x50')

    ->run()
;

if ($response->isSuccessful()) {
    header('Content-type: image/jpeg');
    echo file_get_contents('background.jpeg');
}
```

### Supported commands:

* `convert`
* `mogrify`
* `identify`

### Currently supported options:

There are **a lot** of command-line options, and each have its own validation system.
 
This is why a "few" ones are implemented now.

**Note:** If an option is not implemented in the `Command` class, you should create an issue or make a Pull Request that implements the new option!

* [`-background`](http://www.imagemagick.org/script/command-line-options.php#background)
* [`-blur`](http://www.imagemagick.org/script/command-line-options.php#blur)
* [`-crop`](http://www.imagemagick.org/script/command-line-options.php#crop)
* [`-extent`](http://www.imagemagick.org/script/command-line-options.php#extent)
* [`-fill`](http://www.imagemagick.org/script/command-line-options.php#fill)
* [`-font`](http://www.imagemagick.org/script/command-line-options.php#font)
* [`-gaussian-blur`](http://www.imagemagick.org/script/command-line-options.php#gaussian-blur)
* [`-interlace`](http://www.imagemagick.org/script/command-line-options.php#interlace)
* [`-pointsize`](http://www.imagemagick.org/script/command-line-options.php#pointsize)
* [`-quality`](http://www.imagemagick.org/script/command-line-options.php#quality)
* [`-resize`](http://www.imagemagick.org/script/command-line-options.php#resize)
* [`-rotate`](http://www.imagemagick.org/script/command-line-options.php#rotate)
* [`-size`](http://www.imagemagick.org/script/command-line-options.php#size)
* [`-strip`](http://www.imagemagick.org/script/command-line-options.php#strip)
* [`-stroke`](http://www.imagemagick.org/script/command-line-options.php#stroke)
* [`-thumbnail`](http://www.imagemagick.org/script/command-line-options.php#thumbnail)
* [`-annotate`](http://www.imagemagick.org/script/command-line-options.php#annotate)
* [`-draw`](http://www.imagemagick.org/script/command-line-options.php#draw)
* [`xc:`](http://www.imagemagick.org/Usage/canvas/)

Feel free to ask if you want more!

### Some aliases that do magic for you:

* `Command::text()`:
This method uses multiple options added to the `-annotate` one to generate a text block.
You must specify its position and size, but you can specify color and the font file used.

* `Command::ellipse()`: (check source code for the heavy prototype!)
This method uses the `-stroke`, `-fill` and `-draw` options to create an ellipse/circle/disc on your picture.
**Note:** I recommend to check both the source code and the documentation to be sure of what you are doing.

Useful links
===============

* ImageMagick official website: http://www.imagemagick.org
* ImageMagick documentation:
    * [Installation binaries](https://www.imagemagick.org/script/download.php) (depending on your OS and/or distribution)
    * [Geometry](https://www.imagemagick.org/script/command-line-processing.php#geometry) (to resize or place text)
    * [All command-line options](https://imagemagick.org/script/command-line-options.php) ; they're not all available for now, so feel free to make a PR ! ;)
