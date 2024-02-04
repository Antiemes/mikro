```c
#define SCL PB2
#define SDA PB0

#include <avr/pgmspace.h>
#include <Tiny4kOLED_bitbang.h>

void setup()
{
  oled.begin(128, 64, sizeof(tiny4koled_init_128x64br), tiny4koled_init_128x64br);

  oled.setFont(FONT6X8);
  oled.clear();
  oled.on();

  delay(200);
  
}

void loop()
{
  oled.setCursor(0, 0);
  oled.print("Hello, World!");
}
```
