# ESPHome climate component noblex
This adds a new climate component to the ESPHome ecosystem, providing compatibility with Noblex brand climate equipment.

## Basic Usage
In your ESP device configuration add something like this to your *.yaml.
Example:

```yaml
external_components:
  source:
    type: git
    url: https://github.com/
    
climate:
  - platform: noblex
    name: "Prueba AC"
    id: prueba_ac
    sensor: temp
    receiver_id: rcvr
```
