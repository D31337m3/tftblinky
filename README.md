# TFTBlinky - CircuitPython Display Backlight Controller

A CircuitPython library for controlling TFT display backlight blinking while preserving display functionality. This library automatically detects the appropriate backlight control method for your board and provides a simple interface for blinking effects.

## Features

- **Automatic Detection**: Automatically detects whether your board uses brightness control or GPIO pin control for the backlight
- **Non-Destructive**: Preserves display content and functionality while controlling only the backlight
- **Multiple Control Methods**: Supports both `display.brightness` and GPIO pin control methods
- **State Restoration**: Automatically restores original backlight state after operations
- **Flexible Timing**: Customizable on/off timing and blink count

## Supported Boards

This library works with any CircuitPython board that has a built-in TFT display, including:

- **Adafruit PyPortal** series
- **Adafruit CLUE**
- **Adafruit FunHouse**
- **Adafruit MagTag** (e-ink, limited support)
- **Adafruit Matrix Portal**
- **ESP32-S2/S3 boards** with TFT displays
- **Raspberry Pi Pico** with TFT add-ons
- Any board with `board.DISPLAY` and backlight control

## Installation

1. Copy `tftblinky.py` to your CircuitPython device's `/lib` folder or root directory
2. Ensure your CircuitPython installation includes the required libraries:
   - `board`
   - `displayio`
   - `digitalio`
   - `time`

## Quick Start

```python
from tftblinky import TFTBlinky

# Create backlight controller
backlight = TFTBlinky()

# Blink 5 times with default timing (1 second on, 1 second off)
backlight.blink()

# Custom blink pattern: 3 times, 0.5s on, 0.3s off
backlight.blink(count=3, on_time=0.5, off_time=0.3)
```

## API Reference

### Class: `TFTBlinky`

#### `__init__()`

Initializes the backlight controller and automatically detects the control method.

**Detection Priority:**
1. `display.brightness` control (preferred)
2. GPIO pins in order: `DISPLAY_BACKLIGHT`, `TFT_BACKLIGHT`, `BACKLIGHT`

#### `blink(count=5, on_time=1.0, off_time=1.0)`

Blinks the backlight for the specified pattern.

**Parameters:**
- `count` (int): Number of blink cycles (default: 5)
- `on_time` (float): Duration in seconds to keep backlight on (default: 1.0)
- `off_time` (float): Duration in seconds to keep backlight off (default: 1.0)

**Example:**
```python
# Quick flash pattern
backlight.blink(count=10, on_time=0.1, off_time=0.1)

# Slow breathing pattern  
backlight.blink(count=3, on_time=2.0, off_time=2.0)

# SOS pattern simulation
backlight.blink(count=3, on_time=0.2, off_time=0.2)  # S
time.sleep(0.5)
backlight.blink(count=3, on_time=0.6, off_time=0.2)  # O  
time.sleep(0.5)
backlight.blink(count=3, on_time=0.2, off_time=0.2)  # S
```

#### `set_backlight(state)`

Manually controls the backlight state.

**Parameters:**
- `state` (bool): `True` for on, `False` for off

**Returns:**
- `bool`: The state that was set

**Example:**
```python
backlight.set_backlight(False)  # Turn off
time.sleep(2)
backlight.set_backlight(True)   # Turn on
```

#### `restore_original_state()`

Restores the backlight to its original state when the class was initialized.

```python
# After any operations, restore original state
backlight.restore_original_state()
```

## Control Methods

### Brightness Control
Used on boards where `board.DISPLAY` has a `brightness` attribute:
- Controls backlight by setting `display.brightness` between 0.0 and 1.0
- Preserves original brightness level
- More precise control possible

### GPIO Pin Control  
Used on boards with dedicated backlight control pins:
- Controls backlight via digital GPIO pins
- Searches for common pin names automatically
- Binary on/off control only

## Error Handling

The library gracefully handles various error conditions:

```python
backlight = TFTBlinky()

# Check if initialization was successful
if backlight.control_method is None:
    print("No backlight control available on this board")
else:
    print(f"Using {backlight.control_method} control method")
    backlight.blink()
```

## Advanced Usage

### Custom Notification Patterns

```python
class NotificationBlinker:
    def __init__(self):
        self.backlight = TFTBlinky()
    
    def error_pattern(self):
        """Fast red-alert style blinking"""
        self.backlight.blink(count=10, on_time=0.1, off_time=0.1)
    
    def success_pattern(self):
        """Gentle confirmation blink"""
        self.backlight.blink(count=2, on_time=0.3, off_time=0.3)
    
    def attention_pattern(self):
        """Slow attention-getting pulse"""
        self.backlight.blink(count=5, on_time=1.5, off_time=0.5)

# Usage
notifier = NotificationBlinker()
notifier.success_pattern()
```

### Integration with Sensors

```python
import board
import analogio
from tftblinky import TFTBlinky

# Light sensor example
light_sensor = analogio.AnalogIn(board.LIGHT)
backlight = TFTBlinky()

def check_light_level():
    light_value = light_sensor.value
    if light_value < 1000:  # Dark environment detected
        backlight.blink(count=3, on_time=0.2, off_time=0.8)
        print("Low light detected!")

while True:
    check_light_level()
    time.sleep(10)
```

## Troubleshooting

### No Backlight Control Found
```
Could not find backlight control method. Please check your board documentation.
```
**Solutions:**
- Verify your board has a built-in display with backlight control
- Check your board's pinout documentation for backlight pin names
- Manually specify the backlight pin if using a custom setup

### Display Not Detected
```
No built-in display detected.
```
**Solutions:**
- Ensure `board.DISPLAY` is available on your board
- Check that display initialization code runs before TFTBlinky
- Verify CircuitPython version supports your display

### Import Errors
```
ImportError: no module named 'displayio'
```
**Solutions:**
- Update to CircuitPython 4.0+ which includes `displayio`
- Ensure you're not using a "minimal" CircuitPython build
- Check that your board supports the required libraries

## Board-Specific Notes

### PyPortal Series
- Uses `display.brightness` control
- Original brightness is typically 1.0
- Works with all PyPortal variants

### CLUE
- Uses `display.brightness` control  
- May have ambient light sensor integration opportunities

### ESP32-S2/S3 Boards
- Control method varies by manufacturer
- Some use GPIO pins, others use brightness control
- Check your specific board documentation

## Contributing

To contribute to TFTBlinky:

1. Test on your specific board and document results
2. Add support for additional backlight pin names
3. Report issues with specific board models
4. Suggest new blinking patterns or features

## License

This library is released under the MIT License. See the source file for full license text.

## Version History

- **v1.0.0**: Initial release with automatic detection and dual control methods
- Support for brightness and GPIO pin control
- Automatic state restoration
- Comprehensive error handling
