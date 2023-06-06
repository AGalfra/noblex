# ESPHome climate component noblex
This adds a new climate component to the ESPHome ecosystem, providing compatibility with Noblex brand climate equipment.

## Basic Usage
In your ESP device configuration add something like this to your *.yaml.
Example:

```yaml
external_components:
  source:
    type: git
    url: https://github.com/AGalfra/noblex.git
    
climate:
  - platform: noblex
    name: "Prueba AC"
    id: prueba_ac
    sensor: temp
    receiver_id: rcvr
    
remote_receiver:
  id: rcvr
  pin:
    number: GPIO14
    inverted: True
    mode: OUTPUT_OPEN_DRAIN
  tolerance: 45%

remote_transmitter:
  - id: receiver_hack
    carrier_duty_percent: 100%
    pin:
      number: GPIO14
      inverted: true
      mode: OUTPUT_OPEN_DRAIN      
```
##########################################################################################################
## IR transmission protocol details
Extracted from a Noblex A/C model NBX32H18N, deduced by reverse engineering by analyzing the frame received with a VS1838 type sensor.

As it is in most of these devices, each bit that is sent is made up of a sequence of "mark" and "space", the values of the pulses are:
- mark: 600 usec
- space (zero): 520 usec
- space (one): 1560 usec

Each frame that is sent is made up of:
- a header, composed of a mark pulse of 9000 usec and a space pulse of 4500 usec.
- 64 bits of information, with successive marks and spaces.
- A separator (GAP) bit consisting of a mark pulse of 600 and a space pulse of 19980. This separator corresponds to bit 36 of the frame.
- Four CRC bits, calculated as the big endian sum of the 8 bytes, also interpreted as big endian.
- A footer, composed of a mark of 600 usec.

```
        +--------+-----------------+-----+-------------+-----+--------+
        | Header |     35 bits     | GAP |   28 bits   | CRC | Footer |
        +--------+-----------------+-----+-------------+-----+--------+
```
### Mode
The mode is encoded with 3 bits at position [0:3] of the frame. There are 5 modes:
- Auto: 000
- Cool: 100
- Dry: 010
- Fan: 110
- Heat: 001

### Power bit
It is the 4 bit of the frame:
- On: 1
- Off: 0

### Fan
There are 2 bits at position [4:5] of the frame:
- Low: 10
- Medium: 01
- High: 11
- Auto: 00

### Swing
They are 2 bits, in position 6 and 36:

- On: both in one.
- Off: both at zero.

### Temperature
They are 4 bits at position [8:11] of the frame. It is calculated as: TEMPERATURE - 16 (which is the minimum value) and is expressed as big endian.

### Luz
It is represented by a bit in position 21.

- On: 1
- Off: 0

The rest of the bits remain unchanged, or respond to functions not supported in this A/C model.
