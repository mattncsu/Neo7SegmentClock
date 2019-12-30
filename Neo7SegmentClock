//***************************************************************
// A clock using pixels arranged in a 4-digit 7-segment display
//
//  This example uses 3 Pixels Per Segment (pps).
//  3pps x 7segments x 4digits + 2 digit colon = 86 pixels total
//
//
//  Based on 7Segment code by Marc Miller, (https://github.com/marmilicious/FastLED_examples/)
//  and ESP32 Simpletime example
//
//***************************************************************

#include <FastLED.h>
#include "secrets.h"
#include <WiFi.h>
#include <time.h>


const long  gmtOffset_sec = -18000;
const int   daylightOffset_sec = 3600;

#define DATA_PIN    18
#define CLK_PIN     13
#define LED_TYPE    WS2812
#define COLOR_ORDER GRB
#define NUM_LEDS    86
#define BRIGHTNESS  10
#define FRAMES_PER_SECOND 100

uint8_t pps = 3;  // number of Pixels Per Segment
CHSV segON1000(HUE_GREEN,255,255);  // color of 1000s digit segments
CHSV segON100(HUE_AQUA,255,255);  // color of 100s digit segments
CHSV segON10(HUE_PURPLE,255,255);  // color of 10s digit segments
CHSV segON(HUE_RED,255,255);  // color of 1s digit segments
CHSV colON(HUE_YELLOW,255,255); //color of colon
//CRGB colON = CRGB::Orange;

/* CRGB leds[NUM_LEDS];  <--not using this.  Using CRGBArray instead. */
CRGBArray<NUM_LEDS> leds;

// Name segments (based on layout in link above) and define pixel ranges.
CRGBSet segA(  leds(pps*0,  pps-1+(pps*0)  ));
CRGBSet segB(  leds(pps*1,  pps-1+(pps*1)  ));
CRGBSet segC(  leds(pps*2,  pps-1+(pps*2)  ));
CRGBSet segD(  leds(pps*3,  pps-1+(pps*3)  ));
CRGBSet segE(  leds(pps*4,  pps-1+(pps*4)  ));
CRGBSet segF(  leds(pps*5,  pps-1+(pps*5)  ));
CRGBSet segG(  leds(pps*6,  pps-1+(pps*6)  ));
CRGBSet col(leds(84,85));

int count = 8888;  // keeps track of what number to display


//---------------------------------------------------------------

void printLocalTime()
{
  struct tm timeinfo;
  if(!getLocalTime(&timeinfo)){
    Serial.println("Failed to obtain time");
    return;
  }
  Serial.println(&timeinfo, "%A, %B %d %Y %H:%M:%S");
  count=(timeinfo.tm_hour*100+timeinfo.tm_min);
}

void setup() {
  Serial.begin(115200);  // Allows serial monitor output (check baud rate)

  //connect to WiFi
  Serial.printf("Connecting to %s ", ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
      delay(500);
      Serial.print(".");
  }
  Serial.println(" CONNECTED");
  
  
  //init and get the time
  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
  printLocalTime();

  
  FastLED.addLeds<LED_TYPE,DATA_PIN,COLOR_ORDER>(leds, NUM_LEDS).setCorrection(TypicalSMD5050);

  FastLED.setBrightness(BRIGHTNESS);
  FastLED.clear();  // Initially clear all pixels
}

bool colon;

//---------------------------------------------------------------
void loop()
{
  EVERY_N_MILLISECONDS(200){
    setSegments(count);  // Determine which segments are ON or OFF
    printLocalTime();
  }
  EVERY_N_MILLISECONDS(1000){
    if (colon){
      col = colON;
    } else {
      col = CRGB::Black;
    }
    colon = !colon;
  }
  EVERY_N_MINUTES(5){
    configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
  }
  FastLED.delay(1000/FRAMES_PER_SECOND); 
}



//---------------------------------------------------------------
void setSegments(int count){
  // Based on the current count set number segments on or off
  uint8_t c1 = 0;  // Variable to store 1s digit
  uint8_t c10 = 0;  // Variable to store 10s digit
  uint8_t c100 = 0;  // Variable to store 100s digit
  uint8_t c1000 = 0;  // Variable to store 100s digit
  int c;
  CHSV segCOLOR(0,0,0);

  c1 = count % 10;
  c10 = (count / 10) % 10;
  c100 = (count / 100) % 10;
  c1000 = (count / 1000) % 10;
    
  Serial.print("count = "); Serial.print(count);  // Print to serial monitor current count
  Serial.print("\t  1000s: "); Serial.print(c1000);  // Print 1000s digit
  Serial.print("  100s: "); Serial.print(c100);  // Print 100s digit
  Serial.print("   10s: "); Serial.print(c10);  // Print 10s digit
  Serial.print("   1s: "); Serial.println(c1);  // Print 1s digit

  // Operate on 1s digit segments first, shift them over,
  // then 10's digit, and then do the 100s digit segments.
 for (uint8_t i=0; i < 4 ; i++) {
    if (i == 0) {
      c = c1;
      segCOLOR = segON;
    }
    if (i == 1) {
      c = c10;
      segCOLOR = segON10;
    }
    if (i == 2) {
      c = c100;
      segCOLOR = segON100;
    }
    if (i == 3) {
      c = c1000;
      segCOLOR = segON1000;
    }
//    Serial.print("i="); Serial.print(i); Serial.print(" c=");Serial.println(c);
    segA = segB = segC = segD = segE = segF = segG = CRGB::Black;  // Initially set segments off

    if (c == 0) { segA = segB = segC = segD = segE = segF = segCOLOR; }
    if (c == 1) { segB = segC = segCOLOR; }
    if (c == 2) { segA = segB = segD = segE = segG = segCOLOR; }
    if (c == 3) { segA = segB = segC = segD = segG = segCOLOR; }
    if (c == 4) { segB = segC = segF = segG = segCOLOR; }
    if (c == 5) { segA = segC = segD = segF = segG = segCOLOR; }
    if (c == 6) { segA = segC = segD = segE = segF = segG = segCOLOR; }
    if (c == 7) { segA = segB = segC = segCOLOR; }
    if (c == 8) { segA = segB = segC = segD = segE = segF = segG = segCOLOR; }
    if (c == 9) { segA = segB = segC = segF = segG = segCOLOR; }

    if (i == 0) {  // Shift segments over to 1s digit display area
      for (uint8_t p=0; p < (7*pps); p++) {
        leds[p+(3*7*pps)] = leds[p];
      }
    }

    if (i == 1) {  // Shift segments over to 10s digit display area
      for (uint8_t p=0; p < (7*pps); p++) {
        leds[p+(2*7*pps)] = leds[p];
      }
    }

    if (i == 2) {  // Shift segments over to 100s digit display area
      for (uint8_t p=0; p < (7*pps); p++) {
        leds[p+(1*7*pps)] = leds[p];
      }
    }

  }
  for (uint8_t p=0; p < NUM_LEDS;p++) {
  Serial.print(leds[p]);
  if((p+1) % (7*pps) == 0){
    Serial.print(" ");
  }
}
Serial.println();
}//end setSegments

