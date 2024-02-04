```c
#define SCL PB2
#define SDA PB0

#include <avr/pgmspace.h>
//#include <Tiny4kOLED.h>
#include <TinyDebug.h>
#include <Tiny4kOLED_bitbang.h>

#include "map.h"
#include "textures.h"

#define PIN_LEFT 5
#define PIN_RIGHT 3
#define PIN_UP 1
#define PIN_DOWN 4

int8_t x_frame_pos = 0;
int8_t y_frame_pos = 0;
int8_t x_player_pos = 1;
int8_t y_player_pos = 1;
uint16_t collected = 0;

void setup()
{
  OSCCAL = 0xFF;
  Debug.begin();
  Debug.println(F("Hello, TinyDebug!"));

  pinMode(PIN_LEFT, INPUT_PULLUP);
  pinMode(PIN_RIGHT, INPUT_PULLUP);
  pinMode(PIN_UP, INPUT_PULLUP);
  pinMode(PIN_DOWN, INPUT_PULLUP);

  oled.begin(128, 64, sizeof(tiny4koled_init_128x64br), tiny4koled_init_128x64br);

  // Two fonts are supplied with this library, FONT8X16 and FONT6X8
  
  oled.setFont(FONT6X8);

  // To clear all the memory
  oled.clear();
  oled.on();

  delay(200);
  
}

int8_t get_map_element(int8_t x_pos, int8_t y_pos)
{
  uint8_t bpos, mapb, bitindex;

  bpos = y_pos * 16 + (x_pos >> 1);
  mapb = pgm_read_byte(maze_map + bpos);
  bitindex = (x_pos & 1) ? 0 : 4;
  return (mapb >> bitindex) & 0x0f;
}

void draw_map(int8_t xo, int8_t yo)
{
  uint8_t x, y;

  for (y=0; y<8; y++)
  {
    for (x=0; x<16; x++)
    {
      uint8_t sx = x * 8;
      uint8_t sy = y;
      uint8_t xm = xo + x;
      uint8_t ym = yo + y;
      uint8_t map_element;
      if (xm == x_player_pos && ym == y_player_pos)
      {
        map_element = 16;
      }
      else
      {
        map_element = get_map_element(xm, ym);
      }
      if (collected & (1 << map_element))
      {
        map_element = 0;
      }
      oled.bitmap(sx, sy, sx + 8, sy + 1, img_blocks + map_element * 8);
    }
  }

}

void loop()
{
  uint8_t changed = 1;
  uint32_t lastPress = 0;

  while (1)
  {
    uint8_t dir_l = 0, dir_r = 0, dir_u = 0, dir_d = 0;
    if (changed)
    {
      draw_map(x_frame_pos, y_frame_pos);
    }
    changed = 0;

    if (millis() - lastPress > 130)
    {
      dir_l = 1 - digitalRead(PIN_LEFT);
      dir_r = 1 - digitalRead(PIN_RIGHT);
      dir_u = 1 - digitalRead(PIN_UP);
      dir_d = 1 - digitalRead(PIN_DOWN);
      int8_t new_x_player_pos = x_player_pos;
      int8_t new_y_player_pos = y_player_pos;
      if (dir_l && x_player_pos > 0)
      {
        new_x_player_pos--;
      }
      if (dir_r && x_player_pos < 31)
      {
        new_x_player_pos++;
      }
      if (dir_u && y_player_pos > 0)
      {
        new_y_player_pos--;
      }
      if (dir_d && y_player_pos < 15)
      {
        new_y_player_pos++;
      }
      int8_t field = get_map_element(new_x_player_pos, new_y_player_pos);
      if (field == 0 || field >=6)
      {
        //Debug.println(x_player_pos);
        lastPress = millis();
        changed = 1;
        x_player_pos = new_x_player_pos;
        y_player_pos = new_y_player_pos;
        if (field >= 6)
        {
          collected |= (1<<field);
        }

        if (x_player_pos < x_frame_pos + 1)
        {
          x_frame_pos = x_player_pos - 1;
        }
        if (x_player_pos > x_frame_pos + 14)
        {
          x_frame_pos = x_player_pos - 14;
        }
        if (y_player_pos < y_frame_pos + 1)
        {
          y_frame_pos = y_player_pos - 1;
        }
        if (y_player_pos > y_frame_pos + 6)
        {
          y_frame_pos = y_player_pos - 6;
        }
      }

      //Debug.print(x_player_pos);
      //Debug.print(F(" "));
      //Debug.println(y_player_pos);
    }
  }
}
```
