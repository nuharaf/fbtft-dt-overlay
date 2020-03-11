### Collection of device tree overlay for small lcd display

This overlay is tested with mainline kernel (5.4 at the time of testing) on Orange pi zero. You can use it as is if you follow the wiring diagram or adjust it for your own hardware.

#### Adjusting gpio

Aside from using the spi pin, a few gpio is needed to control reset line, data/register select and sometime led backlight. You can hardwire led control pin to 3.3V to save one gpio. The part needed to be changed is on the opiz_display_pins and opizdisplay.

For example , to change reset line from PA10 to PA6 

**1.**
change this :

```
	fragment@1 {
		target = <&pio>;
		__overlay__ {
			opiz_display_pins: opiz_display_pins {
                pins = "PA2", "PA10", "PA18";
                function = "gpio_out";
			};
		};
	};
```
into this :
```
	fragment@1 {
		target = <&pio>;
		__overlay__ {
			opiz_display_pins: opiz_display_pins {
                pins = "PA2", "PA6", "PA18";
                function = "gpio_out";
			};
		};
	};
```

This part of the overlay define the pin group which will be used/consumed by the spi controller node.

**2.**
The next is to adjust the display node, from this :

```

			opizdisplay: opiz-display@0{
				compatible = "ilitek,ili9341";
				reg = <0>;
				pinctrl-names = "default";
				pinctrl-0 = <&opiz_display_pins>;

				spi-max-frequency = <32000000>;
				rotate = <90>;
				bgr;
				fps = <30>;
				buswidth = <8>;
				reset-gpios = <&pio 0 10 1>;
				dc-gpios = <&pio 0 2 0>;
			};
```

into this :
```

			opizdisplay: opiz-display@0{
				compatible = "ilitek,ili9341";
				reg = <0>;
				pinctrl-names = "default";
				pinctrl-0 = <&opiz_display_pins>;

				spi-max-frequency = <32000000>;
				rotate = <90>;
				bgr;
				fps = <30>;
				buswidth = <8>;
				reset-gpios = <&pio 0 6 1>;
				dc-gpios = <&pio 0 2 0>;
			};
```
### Explanation of gpio cell
If you notice the pin definition has 3 number on it. This is how a pin defined in allwinenr H2 soc (probably other allwinenr soc too). For allwinner, refer to this [binding doc](https://elixir.bootlin.com/linux/latest/source/Documentation/devicetree/bindings/pinctrl/allwinner,sun4i-a10-pinctrl.yaml)

excrept :
```
      GPIO consumers must use three arguments, first the number of the
      bank, then the pin number inside that bank, and finally the GPIO
      flags.
```
