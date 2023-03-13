#include "EnableInterrupt.h"
#include "RadiationWatch.h"
/*
Output the measurements on serial as CSV values.

You can use the joined Python script serial_plot.py to plot dynamically
the radiation level.
*/

RadiationWatch radiationWatch1(2,3);
RadiationWatch radiationWatch2(4,5);

unsigned long csvStartTime;
unsigned int counter = 0;

void csvKeys(Print &print=Serial)
{
  // inizialise start time
  csvStartTime = millis();
  // Write CSV keys to a print class (default print class is Serial).
  print.println("time(ms),count[1],count[2],cpm[1],cpm[2],uSv/h[1],uSv/h[2],uSv/hError[1],uSv/hError[2]");
}

void csvStatus(RadiationWatch radiationWatch1, RadiationWatch radiationWatch2, Print &print=Serial)
{
  // Write CSV to a print class (default print class is Serial).
  // time(ms)
  print.print(millis() - csvStartTime);
  print.print(',');
  // count-PG1
  print.print(radiationWatch1.radiationCount());
  print.print(',');
  // count-PG2
  print.print(radiationWatch2.radiationCount());
  print.print(',');
  // cpm-PG1
  print.print(radiationWatch1.cpm(), 3);
  print.print(',');
  // cpm-PG2
  print.print(radiationWatch2.cpm(), 3);
  print.print(',');
  // uSv/h-PG1
  print.print(radiationWatch1.uSvh(), 3);
  print.print(',');
  // uSv/h-PG2
  print.print(radiationWatch2.uSvh(), 3);
  print.print(',');
  // uSv/hError-PG1
  print.print(radiationWatch1.uSvhError(), 3);
  print.println(',');
  // uSv/hError-PG2
  print.print(radiationWatch2.uSvhError(), 3);
  print.println("");
}

void onRadiationPulse() {
  // For each radiation update the values sent to serial in CSV.
  csvStatus(radiationWatch1, radiationWatch2);
}

void setup()
{
  Serial.begin(9600);
  radiationWatch1.setup();
  radiationWatch2.setup();
  // Register the callback.
  radiationWatch1.registerRadiationCallback(&onRadiationPulse);
  radiationWatch2.registerRadiationCallback(&onRadiationPulse);
  // Print the CSV keys and the initial values.
  csvKeys();
  csvStatus(radiationWatch1, radiationWatch2);
}

void loop()
{
  if (counter % 2 == 0) { 
    radiationWatch1.loop();
    radiationWatch2.loop(); 
  } 
  else {
    radiationWatch2.loop();
    radiationWatch1.loop(); 
  }
  counter++;
}
