# ezsign.py

This is a library that communicates via USB serial with Santek Technology Corporation's electronic paper display "Santek EZ Sign 4.2” E-paper Display." It allows displaying a specified page, reading image data from a specified page and saving it to a file, and writing image file data to a specified page.

[Santek EZ Sign 4.2](https://www.santekshop.jp/product/santek-ez-sign-4-2-e-paper-display/)


## Usage

Connect to a PC via USB cable and check the serial port name. On Windows, check the COM port name that appears in the Device Manager after connection.

On Linux, check the messages log where the name should be listed.

Example serial port names:

    COM3 
    /dev/ttyACM0 
    /dev/tty.usbserial-Disabled

Ensure that the EZ Sign 4.2 is powered on. Even when powered off, the PC may recognize it as a serial port, but it will not respond to commands from the EZSign library.

Use the EZ Sign 4.2 when its power is on (LED is lit blue).

The Python libraries `pyserial`, `Pillow`, and `argparse` are required. If not installed, install them using:

    poetry install --no-root

To run any script within this project, use:

    poetry run python script_name.py [arguments]

For example:

    poetry run python conv2bwr.py -b inputfile.png outputfile.png

### Using as a Python Library

To use it as a Python library, place it in the designated library directory and call the `EZSign` class.

Open the serial port using `pyserial` and pass it as the first argument to the `EZSign` constructor. Set the baud rate and parity as follows:

    Serial_Port=serial.Serial('COM3', baudrate=38400, parity= 'N')
    ezsign = EZSign(Serial_Port)

#### Display a Specific Page

    ezsign.showpage(page_number)

The page number ranges from 1 to 5. Specifying `EZSign.NEXT` switches to the next page, and `EZSign.PREV` switches to the previous page.

Page rewriting takes about 30 seconds, during which the EZ Sign 4.2's LED blinks.

#### Read Image Data from a Specified Page and Save to File

    ezsign.readimage(page_number, file_name)

The page number ranges from 1 to 5. Specify the destination file name, such as 'image01.png'.

Since it uses Pillow’s `save()`, the file format depends on the specified extension.

On the electronic paper of EZSign, black pixels are saved as RGB(0,0,0), white as RGB(255,255,255), and red as RGB(255,0,0).

#### Write Image File Data to a Specified Page

    ezsign.writeimage(page_number, file_name)

The page number ranges from 1 to 5. Specify the image file name, such as 'image01.png'.

Since it uses Pillow's `open()`, any image format supported by Pillow can be read.

Black pixels in the image are mapped as RGB(0,0,0), white pixels as RGB(255, 255, 255), and red pixels as RGB(255, 0, 0) on the electronic paper.

The image file's width and height can be any size, but it is internally resized to 400x300. Non-400x300 images may be distorted or have resizing noise.


### Running Directly with Command-line Arguments

Specify the serial port name, command, and arguments. In the following example, `COM3` is the serial port.

Specify an uncompressed format such as `.png` for image files. While compressed formats like `.jpg` can be used, they may store non-standard pixel values, preventing proper display on the electronic paper.

#### Display a Specific Page

    Example: Display page 1
    poetry run python EZSign.py COM3 -s 1

#### Read Image Data from a Specific Page and Save to File

    Example: Read image data from page 1 and save it as image.png
    poetry run python EZSign.py COM3 -r 1 image.png

#### Write Image File Data to a Specified Page

    Example: Write image.png to page 1
    poetry run python EZSign.py COM3 -w 1 image.png

If EZSign.py stops responding, disconnect and reconnect the USB connection between the PC and EZ Sign 4.2, then try again. Ensure that the EZ Sign 4.2 is powered on (LED is blue, not green).


-----------------------------------
# Red-Black Two-Color Conversion Script `conv2bwr.py`

This script uses Pillow to convert an image file into a red-black-white three-color image for display on the EZSign 4.2 e-paper display.

The converted image can be written to EZ Sign 4.2 using:

    poetry run python ezsign.py -w page_number outputfile.png

Specify an uncompressed format like `.png` for the output image file.

The input image file can have any dimensions, but this script resizes all images to 400x300. Non-4:3 aspect ratio images may appear distorted after conversion.

Black and white binarization uses Pillow’s default dithering method.

For red pixels, the script applies HSV conversion. Pixels close to Hue(0) (red) with sufficiently high Saturation and Value are treated as red. This red data is binarized using Pillow and then merged with the black-and-white image.

(Depending on the original image, unintended red regions may appear. Use the `-b` option to disable red or adjust `--huerange` and `--redvalue` options.)

## Usage

Specify the input image file and the output red-black-white image file.

Example input image file: A race bike image generated using [Fooocus](https://github.com/lllyasviel/Fooocus).

![inputfile.png](/images/MotoGPimage2.png)


### Basic Usage

    Example: Convert inputfile.png to a 400x300 red-black-white image and save it as outputfile.png
    poetry run python conv2bwr.py inputfile.png outputfile.png

![outputfile_BWR.png](/images/MotoGPimage2_BWR.png)

### `-b` Option: Disable Red

    Example: Convert inputfile.png to a 400x300 black-and-white image
    poetry run python conv2bwr.py -b inputfile.png outputfile.png

![outputfile_BW.png](/images/MotoGPimage2_BW.png)

### `--avoidresize`: Do Not Resize to 400x300

    Example: Convert inputfile.png to a red-black-white image without resizing (EZSign resizes when writing)
    poetry run python conv2bwr.py --avoidresize inputfile.png outputfile.png

### `--huerange`: Set the Range of Hue Values Considered Red

    Example: Convert inputfile.png using a hue range of ±40 (default is 20)
    poetry run python conv2bwr.py --huerange 40 inputfile.png outputfile.png

![outputfile_hue40.png](/images/MotoGPimage2_hue40.png)

### `--redvalue`: Set the Threshold for Red Pixels

    Example: Convert inputfile.png with a red pixel threshold of 200 (default is 170)
    poetry run python conv2bwr.py --redvalue 100 inputfile.png outputfile.png

![outputfile_value100.png](/images/MotoGPimage2_value100.png)

## Conversion and Display Examples

Example: Race bike image generated by AI

    poetry run python conv2bwr.py MotoGPimage2.png MotoGPimage2_BWR.png
    poetry run python ezsign.py COM3 -w 3 MotoGPimage2_BWR.png

![MotoGPimage2.png](/images/MotoGPimage2.png) ![MotoGPimage2_BWR.png](/images/MotoGPimage2_BWR.png)

![EZSign display sample1](/images/EZsignDisplaySample1.jpg)

(Similar examples follow for illustrated banknotes and landmarks.)

