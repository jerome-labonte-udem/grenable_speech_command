This project explains how to use the Xiao Ble Sense micro-controller to recognize
speech commands. It combines instructions from various sources with modifications to
be compatible with the latest Tensorflow and Google Colab changes.


## Disclaimer
To allow users to benefits from future updates, changes necessary to make the example compatible with the tflite micro 
examples are written here and explained. If you wish to use a newer version of the example, just reproduce those changes
(line numbers and structure might be different). 

For example, changes proposed by https://github.com/lakshanthad/tflite-micro-arduino-examples are now incompatible with
the current version. 

###
Environment at the time of writing (November 2022)

Arduino platform Seeeduino:mbed 2.9.0 
Arduino IDE 2.0.3

Python 3.7 (Python <= 3.7 is necessary to support tensorflow 1.15.0)


## Hardware and software setup
### Arduino IDE Configuration
Follow the instructions at https://wiki.seeedstudio.com/XIAO_BLE/ to configure your
XIAO BLE SENSE board and Arduino IDE.

### Install the tflite micro arduino examples
Go to https://github.com/tensorflow/tflite-micro-arduino-examples and download the library 
as a .zip file (Clidk the green "<>Code" Button and the "Download ZIP") and install it in the
Arduino IDE (Sketch -> Include Library -> Add .ZIP Library, and then select the downloaded .zip file)

### Modify the micro_speech example to be compatible with XIAO BLE SENSE

Since the XIAO BLE SENSE is mostly compatible with the Arduino NANO 33 BLE, only
minor changes have to be done to support it. You will find the libraries directory under your 
main Arduino Directory :

* in libraries/tflite-micro-arduino-examples_main/examples/micro_speech/arduino_command_responder.cpp :

    Change line 16:
    ```
    #if defined(ARDUINO) && !defined(ARDUINO_ARDUINO_NANO33BLE)
    ```
    
    to
    ```
    #if defined(ARDUINO) && !defined(ARDUINO_ARDUINO_NANO33BLE) && !defined(ARDUINO_SEEED_XIAO_NRF52840_SENSE)
    ```
    This prevents the compiler from crashing if ARDUINO_ARDUINO_NANO33BLE is not defined by the board platform.
    Since the NANO33BLE and SEEED XIAO BLE SENSE are almost identical, code for NANO33BLE command responder should 
    work for both boards.


* in libraries/tflite-micro-arduino-examples_main/examples/micro_speech/arduino_audio_provider.cpp :

    Change line 16:
    ```
    #if defined(ARDUINO) && !defined(ARDUINO_ARDUINO_NANO33BLE)
    ```
    
    to
    ```
    #if defined(ARDUINO) && !defined(ARDUINO_ARDUINO_NANO33BLE) && !defined(ARDUINO_SEEED_XIAO_NRF52840_SENSE)
    ```
    This prevents the compiler from crashing if ARDUINO_ARDUINO_NANO33BLE is not defined by the board platform.
    Since the NANO33BLE and SEEED XIAO BLE SENSE are almost identical, code for NANO33BLE audio provider should 
    work for both boards.

* in  libraries/tflite-micro-arduino-examples_main/src/tensorflow:lite/micro/system_setup.cpp :
    
    Change line 22:
    ```
    #if defined(ARDUINO) && !defined(ARDUINO_ARDUINO_NANO33BLE)
    ```
    to
    ```
    #if defined(ARDUINO) && !defined(ARDUINO_ARDUINO_NANO33BLE) && !defined(ARDUINO_SEEED_XIAO_NRF52840_SENSE)
    ```
    This prevents the compiler from crashing if ARDUINO_ARDUINO_NANO33BLE is not defined by the board platform.
    Since the NANO33BLE and SEEED XIAO BLE SENSE are almost identical, code for NANO33BLE audio provider should 
    work for both boards.


* in libraries/tflite-micro-arduino-examples_main/src/peripherals/peripherals.h
    
    Change line 52:
    ```
    constexpr pin_size_t kLED_DEFAULT_GPIO = D13;
    ```
    
    to
    ```
    constexpr pin_size_t kLED_DEFAULT_GPIO = LED_BUILTIN;
    ```
    This prevents the compiler from crashing if  D13 is not defined by the board platform.

### Opening and Compiling the example

  * In the Arduino IDE, open the micro speech example (File -> Examples -> 
  Arduino_TensorFlowLite -> micro_speech).
  * Make sure you have the right board selected (Tools -> 
  Boards:[...] -> Seeed nRF52 mbed-enabled Boards -> Seed XIAO BLE Sense - nRF52840)
  * Make sure the sketch compiles without error

## Training the model
You can either train the model locally (on you personal computer) or using Google Colab platform. It should about an 
hour. You can see progress in the Tensorboad cell. Accuracy curve should increase gradually as training progresses.

### Training on Colab
 Go to https://colab.research.google.com/github/tensorflow/tflite-micro/blob/main/tensorflow/lite/micro/examples/micro_speech/train/train_micro_speech_model.ipynb 
and make the following changes :
* Install and configure Python 3.7 : add the following cell at the top of the notebook
  ```
  !sudo apt-get update -y
  !sudo apt-get install python3.7
  from IPython.display import clear_output 
  clear_output()
  !sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 1
  
  # Choose one of the given alternatives:
  !sudo update-alternatives --config python3
  
  # This one used to work but now NOT(for me)!
  # !sudo update-alternatives --config python
  
  # Check the result
  !python3 --version
  
  # Attention: Install pip (... needed!)
  !sudo apt install python3-pip
  !python -m pip install --upgrade pip
  ```
  Run it and enter the correct number to select python 3.7 when prompted


* [OPTIONAL] Cell 1 : Change the words that the model should detect (from the list described in the comment above WANTED_WORDS = ...)
* Change Cell 3 : 
    ```
    %tensorflow_version 1.x
    import tensorflow as tf
    ```
    
    to
    ```
  !python -m pip uninstall -y tensorflow
  !python -m pip install tensorflow-cpu==1.15.0
  !python -m pip install protobuf==3.20.0

  import tensorflow as tf
    ```
* Run all the cells in order (This should take 1-2 hours). When done, either save the model.cc file to your local drive or 
copy the value from the last cell output to the example file (see next step)

    
  ### Training Locally
  * Create a new python 3.7 virtual environment and activate it
  * from this directory, install the requirements
    ```
    pip install -r requirements.txt
    ```
  * Launch jupyter notebook and go the url shown in the terminal
    ```
    jupyter notebook
    ```
  * Open the train_micro_speech_model.ipynb notebook
    * [OPTIONAL] Cell 1 : Change the words that the model should detect (from the list described in the comment above WANTED_WORDS = ...)
    * Run all the cells in order (This should take 1-2 hours). When done, either save the model.cc file to your local drive or 
    copy the value from the last cell output to the example file (see next step)
    
  ### Copying model the device
Make the following changes to the micro_speech example, using the Arduino IDE or any other text editor.
  * in libraries/tflite-micro-arduino-examples_main/examples/micro_speech/micro_features_micro_model_settings.h :

    Change line 40:
    ```
    constexpr int kCategoryCount = 4;
    ```
    So that kCategoryCount is equal to the number of WANTED_WORDS + 2. For example, if WANTED_WORDS="yes,no,stop",
  kCategoryCount=5. The two additionnal categories are "silence" and "unknown".
  
  * in libraries/tflite-micro-arduino-examples_main/examples/micro_speech/micro_features_micro_model_settings.cpp :

    Change line 18-23:
    ```
    const char* kCategoryLabels[kCategoryCount] = {
        "silence",
        "unknown",
        "yes",
        "no",
    };
    ```
    So that the list of words consists of "silence" and "unknown" followed by the words defined in WANTED_WORDS, in 
    the same order.
  * in libraries/tflite-micro-arduino-examples_main/examples/micro_speech/micro_features_model.cpp :

    Change line 35 and next:
    ```
    const unsigned char g_model[] DATA_ALIGN_ATTRIBUTE = {
    [...]
    };
    const int g_model_len = ...;
    ```
    Copy the value of the array in the saved model.cc file (or copy/paste from the notebook) inside the 
    g_model[] DATA_ALIGN_ATTRIBUTE curly brackets to replace the current values and change the g_model_len value to
    the one found at the end of the model.cc / cell output.

## Testing and running on the XIAO BLE Sense
Recompile and upload the example on the XIAO BLE Sense board by connecting the board via usb and make sure that the 
correct port is selected. If you did not change the WANTED_WORDS ("yes,no"), the board led will blink everytime 
inference is run and the led should light up green 
when it hears "yes", red when it hears "no" and blue if it detects an unknown word. You can change this behaviour by
editing the libraries/tflite-micro-arduino-examples_main/examples/micro_speech/arduino_command_responder.cpp file.
  
For example, you can comment out lines 82-86 to stop the blinking :
```
// if (count & 1) {
//   digitalWrite(LED_BUILTIN, HIGH);
// } else {
//   digitalWrite(LED_BUILTIN, LOW);
// }
```
If you changed a word, you can edit lines 56-64 to detect commands starting with letter(s) other than "y", "n"
and "u" or do something else when a command is detected (activate a motor, etc); 


## Future works:
* Instructions for recording, processing and using custom instructions
* Centralized repository containing stable versions of all the code needed to reproduce the experiment.
  

Sources:
https://wiki.seeedstudio.com/XIAO-BLE-Sense-TFLite-Mic/

https://wiki.seeedstudio.com/XIAO_BLE/

https://github.com/lakshanthad/tflite-micro-arduino-examples

https://colab.research.google.com/github/tensorflow/tflite-micro/blob/main/tensorflow/lite/micro/examples/micro_speech/train/train_micro_speech_model.ipynb

https://github.com/tensorflow/tflite-micro/tree/main/tensorflow/lite/micro/examples/micro_speech

Requirements
Python <= 3.7

convert to 16 bit Little-endian PCM 16kHz wave file

from commands directory:

```
for i in */*; do ffmpeg -i "$i" -acodec pcm_s16le -ac 1 -ar 16000 "../clean/${i}.wav"; done
```
keep only 1 sec per wav:
``` 
git clone https://github.com/petewarden/extract_loudest_section.git
cd extract_loudest_section
make
```
from clean directory:
```
for i in */*.wav; do /tmp/extract_loudest_section/gen/bin/extract_loudest_section "./$i" "/home/jerome/clean/${i%/*}"; done
```