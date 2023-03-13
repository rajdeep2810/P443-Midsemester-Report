#include "EnableInterrupt.h"
#include "RadiationWatch.h"

/* Initializing pocket geiger 1 */
RadiationWatch radiationWatch1(2,3);
/* Initializing pocket geiger 2 */
RadiationWatch radiationWatch2(4,5);

unsigned long Timer = (millis()/1000)/60;
unsigned int counter = 0;
unsigned long seed;
unsigned int coincident_signal_value = 0;
unsigned long pg_counter_1 = 0;
unsigned long pg_counter_2 = 0;
unsigned int coincident_signal;
int markervalue = 450;
int timelimit = 2880;
String response;

void initiator(Print &print=Serial)
{
  print.println("Pocket Geiger [1 & 2] initiated !"); 
}
void registrymaker(Print &print=Serial)
{
  if ((millis()/1000)/60 - Timer >= timelimit)
  {
    print.print("Total Time Elapsed (min):");
    print.println(timelimit);
    print.print("Total Counts from Pocket Geiger 1 :");
    print.println(pg_counter_1);
    print.print("Total Counts from Pocket Geiger 2 :");
    print.println(pg_counter_2);
    print.print("Total coincidences obtained from Pocket Geiger 1 and 2 :");
    print.println(coincident_signal_value);
  }
}

void onRadiation1()
{
  pg_counter_1++;
}

void onRadiation2()
{
  pg_counter_2++;
}



void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  randomSeed(analogRead(0));
  Serial.println("Arduino controller initiated !");
  radiationWatch1.setup();
  radiationWatch2.setup();
  Serial.println("Pocket Geiger setup loaded in Arduino !");
  initiator();
  radiationWatch1.registerRadiationCallback(&onRadiation1);
  radiationWatch2.registerRadiationCallback(&onRadiation2);
}

void loop() {
  // put your main code here, to run repeatedly:  
  counter = random();
  if (counter % 2 == 0)
  {
    radiationWatch1.loop();
    radiationWatch2.loop();
  } else
  {
    radiationWatch2.loop();
    radiationWatch1.loop();
  }
  
  coincident_signal = analogRead(A3);

  Serial.print("Threshold:");
  Serial.print(markervalue);
  Serial.print(" , ");  
  Serial.print("Response:");
  Serial.print(coincident_signal);
  Serial.print(" , ");
  Serial.print("[PG1] counts:");
  Serial.print(pg_counter_1);
  Serial.print(" , ");
  Serial.print("[PG2] counts:");
  Serial.print(pg_counter_2); 
  Serial.print(" , ");
  Serial.print("Coincidences detected:");
  Serial.println(coincident_signal_value);

  if (coincident_signal >= 125)
  {
    coincident_signal_value++;
  }
  registrymaker();
  if ((millis()/1000)/60 - Timer >= timelimit)
  {
    
    Serial.print("!! ACQUISITION ENDS NOW !!");
    while(1)
    {

    }      
  }
  }
