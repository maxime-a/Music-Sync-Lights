## Working principle 

Processing program running on the PC analyze music with beat detection algorithm and audio level , these data are send via serial communication 
to an arduino nano. The arduino will command the adressable led strip.

## Processing code
```Java
import processing.serial.*;

Serial arduino;

/**
  * This sketch demonstrates use the BeatDetect object song SOUND_ENERGY mode and level in the mix to send command via serial port.
  */
  
import ddf.minim.*;
import ddf.minim.analysis.*;

Minim minim;
AudioInput in;
BeatDetect beat;

float eRadius;
int count,intensityM,period = 3; //Period set the number of sample taken before sending the audio level
int[] intensity = new int[period];


void setup()
{
  size(200, 200, P3D);
  
  minim = new Minim(this);
  in = minim.getLineIn();  
  beat = new BeatDetect();
  beat.setSensitivity(300);
  
  printArray(Serial.list());
  arduino = new Serial(this, Serial.list()[0],9600);
  
  ellipseMode(RADIUS);
  eRadius = 20;
  count = 0;
}

void draw()
{
  background(0);
  beat.detect(in.mix);
 
  float a = map(eRadius, 20, 80, 60, 255);
  fill(60, 255, 0, a);
 
  if ( beat.isOnset() )
  {
    eRadius = 80;
    arduino.write("#BEAT\n");
  }
  
  ellipse(width/2, height/2, eRadius, eRadius);
  eRadius *= 0.95;
  if ( eRadius < 20 ) eRadius = 20;
  
  intensity[count] = int(map(in.right.level(), 0.00,0.3,1,100));
  count++;
  if(count == period)
  {
     intensityM =0;
     for(int i=0;i<period;i++)
     {
     intensityM += intensity[i];
     }
     
     intensityM /= period;
     intensityM += 1;
     arduino.write("#BRIG"+intensityM+"\n");
     
     count = 0;
  }

}
'''

## Arduino code
'''

#include <Adafruit_NeoPixel.h>
#define LED_PIN    6
#define LED_COUNT 60

// Declare our NeoPixel strip object:
Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

int a,b,c;
String inputString = "";         // a string to hold incoming data
boolean stringComplete = false;  // whether the string is complete
String commandString = "";

// setup() function -- runs once at startup --------------------------------

void setup() 
{
  Serial.begin(9600);
  strip.begin();           // INITIALIZE NeoPixel strip object (REQUIRED)
  strip.show();            // Turn OFF all pixels ASAP
  strip.setBrightness(50); // Set BRIGHTNESS to about 1/5 (max = 255)
}

void loop()
{
  if(stringComplete)
  {
    stringComplete = false;
    getCommand();

    if(commandString.equals("BEAT"))
    {
    Serial.println("Setting beat");
    randomColor();
    }

    else if(commandString.equals("BRIG"))
    {
    Serial.println("Setting brightness");
    String value = inputString.substring(5,inputString.length()-1);
      if((value.toInt())!=0)
      {
        strip.setBrightness(value.toInt());
        Serial.println(value.toInt());
        strip.show();
      }   
    }

    //Forme #COLO255255255 RED-GRE-BLU
    else if(commandString.equals("COLO"))
      {
      Serial.println("Setting color");
      int red = (inputString.substring(5,inputString.length()-7)).toInt();
      int green = (inputString.substring(8,inputString.length()-4)).toInt();
      int blue = (inputString.substring(11,inputString.length()-1)).toInt();

      for(int i=0; i<strip.numPixels(); i++)
        { // For each pixel in strip...
        strip.setPixelColor(i, strip.Color(red,green,blue));        
        }
      strip.show();
    
      }
    
    inputString = "";
  }
  


}

void serialEvent() {
  while (Serial.available()) {
    // get the new byte:
    char inChar = (char)Serial.read();
    // add it to the inputString:
    inputString += inChar;
    // if the incoming character is a newline, set a flag
    // so the main loop can do something about it:
    if (inChar == '\n') {
      stringComplete = true;
    }
  }
}

void getCommand()
{
  if(inputString.length()>0)
  {
     commandString = inputString.substring(1,5);
  }
}

void randomColor()
{

  do
  {
  a = random(0, 255);
  b = random(0, 255);
  c = random(0, 255);
  }while(a < 20 && b < 20 && c < 20);
  
  for(int i=0; i<strip.numPixels(); i++)
  { // For each pixel in strip...
      strip.setPixelColor(i, strip.Color(a,b,c));        
  }
 
  strip.show(); 
}
```
