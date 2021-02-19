# Pico Explorer Pack <!-- omit in toc -->

Our Pico Explorer Pack offers a vibrant 1.14" (240x240) IPS LCD screen for your Raspberry Pi Pico it also includes four switches, a piezo speaker, and a DRV8833 motor driver.

We've included helper functions to handle every aspect of drawing to the screen and interfacing with the buttons, piezo and motor driver. [See the library reference](#reference) for details.

- [Example Program](#example-program)
- [Reference](#reference)
  - [init](#init)
  - [set_motor](#set_motor)
  - [get_adc](#get_adc)
  - [set_audio_pin](#set-audio_pin)
  - [set_tone](#set_tone)
  - [is_pressed](#is-pressed)
  - [update](#update)
  - [set_pen](#create-pen)
  - [create_pen](#create-pen)
  - [clear](#clear)
  - [pixel](#pixel)
  - [pixel_span](#pixel-span)
  - [rectangle](#rectangle)
  - [circle](#circle)
  - [character](#character)
  - [text](#text)
  - [set_clip](#set-clip)
  - [remove_clip](#remove-clip)

## Example Program

The following example sets up Pico Explorer, displays some basic demo text on the screen, then waits for button A to be pressed.  It then spins a motor, and plays a tone on the piezo speaker.

```python
import picoexplorer
import time

# Initialise the board
buffer = bytearray(picoexplorer.get_height() * picoexplorer.get_width() * 2)
picoexplorer.init(buffer)

# display text on the screen
black_pen = picoexplorer.create_pen(0, 0, 0)         # Create a black pen
picoexplorer.set_pen(black_pen)
picoexplorer.clear()                                 # Blank the screen
picoexplorer.set_pen(255, 255, 255)                  # Set a white pen
picoexplorer.text("pico explorer!", 10, 10, 240, 5)  # Add some text
picoexplorer.update()                                # Update the display with our changes

print("Press Button A...")
while picoexplorer.is_pressed(picoexplorer.BUTTON_A) == False:
    pass
print("Button A was pressed!")

# Turn MOTOR1
picoexplorer.set_motor(picoexplorer.MOTOR1, picoexplorer.FORWARD, 1)
time.sleep(1)
picoexplorer.set_motor(picoexplorer.MOTOR1, picoexplorer.STOP)

# Get and print an ADC reading from channel 0
print(picoexplorer.get_adc(picoexplorer.ADC0))

# Set a pin for audio tones, and play an A (440 Hz)
picoexplorer.set_audio_pin(picoexplorer.GP0)
picoexplorer.set_tone(440)
time.sleep(1)
picoexplorer.set_tone(0)
```

## Functions

### init

Sets up the Pico Explorer. `init` must be called before any other functions since it configures the required PWM and GPIO pins.  `init()` needs a bytearray type display buffer that MicroPython's garbage collection isn't going to eat, make sure you create one and pass it in like so:

```python
buf = bytearray(picoexplorer.get_width() * picoexplorer.get_height() * 2)
picoexplorer.init(buf)
```

### set_motor

The Pico Explorer has two motor channels, capable of spinning the motors both backwards and forwards.  The motors are driven by PWM via the onboard DRV8833 chip, which means you can control the speed of rotation too.  The `channel` parameter should be given a value of `MOTOR1` or `MOTOR2`, which match the pins labelled "MOTORS" on the board.  The `action` parameter should be a value of `FORWARD`, `REVERSE`, or `STOP`.  The `speed` parameter should be a value of 0.0 to 1.0, with higher values spinning the motor faster.

```python
picoexplorer.set_motor(channel, action, speed)
```

### get_adc

The Pico Explorer breaks out three ADC channels, connected to the Pico's ADC-capable pins (26, 27 and 28) which correspond to channels 0, 1 and 2 respectively. Reading these will perform a 12-bit ADC read and then return the result as a float scaled from `0.0` to `1.0`.  The channels can be addressed using the constants `picoexplorer.ADC0`, `picoexplorer.ADC1`,  and `picoexplorer.ADC2`

```python
picoexplorer.get_adc(channel)
```

### set_audio_pin

The `set_audio_pin` function will configure the pin that Pico Explorer uses for audio output. These can be selected using constants from `GP0` to `GP7`, which match the labels on the board.  You can then use the `set_tone` function to play tunes.

```python
picoexplorer.set_audio_pin(picoexplorer.GP0)
```

Note: You must bridge this pin over to the `AUDIO` socket on the Pico Explorer header in order to drive the onboard Piezo.  A jumper wire does the trick nicely.

### set_tone

The `set_tone` function will play an audio tone out of your chosen audio pin, which should be bridged to the "AUDIO" socket on the board.  The `frequency` parameter should be the tone, in Hz, which you want played through the piezo speaker.

```python
picoexplorer.set_tone(frequency)  # Whats the Duty Cycle for?
```

### is_pressed

Reads the GPIO pin connected to one of Pico Explorer's buttons, returning True if it's pressed and False if it is released.

```python
picodisplay.is_pressed(button)
```

The `button` value should be a number denoting a button on the board, or one of the constants `picoexplorer.BUTTON_A`, `picoexplorer.BUTTON_B`, `picoexplorer.BUTTON_X` or `picoexplorer.BUTTON_Y`.

### update

Changes made to the explorer screen  go into the buffer but won't appear until you use the `update` function:

```python
picodisplay.update()
```

### set_pen

To draw to the screen you need to use a pen.  This function sets the pen colour to be used by subsequent calls to drawing functions.  The values for `r`, `g` and `b` should be from 0-255 inclusive.

```python
picodisplay.set_pen(r, g, b)
```

### create_pen

Creates a pen which can be stored as a variable for faster re-use of the same colour through calls to `set_pen`.  The values for `r`, `g` and `b` should be from 0-255 inclusive.

```python
pen_colour = picodisplay.create_pen(r, g, b)
picodisplay.set_pen(pen_colour)
```

### clear

Fills the display buffer with the currently set pen colour.

```python
picodisplay.clear()
```

### pixel

Sets a single pixel in the display buffer to the current pen colour.  The `x` and `y` parameters determine the X and Y coordinates of the drawn pixel in the buffer.

```python
picodisplay.pixel(x, y)
```

### pixel_span

Draws a horizontal line of pixels to the buffer.  The `x` and `y` parameters specify the coordinates of the first pixel of the line.  The `l` parameter describes the length of the line in pixels.  This function will only extend the line towards the end of the screen, i.e. the `x` coordinate should specify the left hand extreme of the line.

```python
picodisplay.pixel_span(x, y, l)
```

### rectangle

Draws a rectangle filled with the current pen colour to the buffer.  The `x` and `y` parameters specify the upper left corner of the rectangle, `w` specifies the width in pixels, and `h` the height.

```python
picodisplay.rectangle(x, y, w, h)
```

![Rectangle function explanation image](/micropython/modules/pico_display/images/rectangle.png)

### circle

Draws a circle filled with the current pen colour to the buffer.  The `x` and `y` parameters specify the centre of the circle, `r` specifies the radius in pixels.

```python
picodisplay.circle(x, y, r)
```

![Circle function explanation image](/micropython/modules/pico_display/images/circle.png)

### character

Draws a single character to the display buffer in the current pen colour.  The `c` parameter should be the ASCII numerical representation of the character to be printed, `x` and `y` describe the top-left corner of the character's drawing field.  The `character` function can also be given an optional 4th parameter, `scale`, describing the scale of the character to be drawn.  Default value is 2.

```python
char_a = ord('a')
picodisplay.character(char_a, x, y)
picodisplay.character(char_a, x, y, scale)
```

### text

Draws a string of text to the display buffer in the current pen colour.  The `string` parameter is the string of text to be drawn, and `x` and `y` specify the upper left corner of the drawing field.  The `wrap` parameter describes the width, in pixels, after which the next word in the string will be drawn on a new line underneath the current text.  This will wrap the string over multiple lines if required.  This function also has an optional parameter, `scale`, which describes the size of the characters to be drawn.  The default `scale` is 2.

```python
picodisplay.text(string, x, y, wrap)
picodisplay.text(string, x, y, wrap, scale)
```

![Text scale explanation image](/micropython/modules/pico_display/images/text_scale.png)

### set_clip

This function defines a rectangular area outside which no drawing actions will take effect.  If a drawing action crosses the boundary of the clip then only the pixels inside the clip will be drawn.  Note that `clip` does not remove pixels which have already been drawn, it only prevents new pixels being drawn outside the described area.  A more visual description of the function of clips can be found below.  Only one clip can be active at a time, and defining a new clip replaces any previous clips.  The `x` and `y` parameters describe the upper-left corner of the clip area, `w` and `h` describe the width and height in pixels.

```python
picodisplay.set_clip(x, y, w, h)
```

![Clip function explanation image](/micropython/modules/pico_display/images/clip.png)

### remove_clip

This function removes any currently implemented clip.
