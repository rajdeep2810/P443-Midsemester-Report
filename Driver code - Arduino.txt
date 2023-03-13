int signalPin1 = 2;
int noisePin1 = 3;
int signalPin2 = 4;
int noisePin2 = 5;

int index = 0;

int signal1 = 0;
int noise1 = 0;
int signal2 = 0;
int noise2 = 0;

unsigned int counter = 0;

unsigned int signalLock1 = 0;
unsigned int noiseLock1 = 0;
unsigned int signalLock2 = 0;
unsigned int noiseLock2 = 0;

unsigned int signalRegister1 = 0;
unsigned int signalRegister2 = 0;

unsigned long signalCount1 = 0;
unsigned long noiseCount_record1 = 0;
unsigned long signalCount2 = 0;
unsigned long noiseCount_record2 = 0;
unsigned long coincidentCount = 0;

unsigned long noiseCount1 = 0;
unsigned long noiseCount2 = 0;

unsigned long Timer = (millis()/1000)/60;
unsigned int timelimit = 1440;

void trigger_function1(){
  if (signal1 == 0 && signalLock1 == 0){
    signalLock1 = 1;
    signalRegister1++;
    signalCount1++;    
  } else if (signal1 == 1 && signalLock1 == 1){
    signalLock1 = 0;
  }
  if (noise1 == 1 && noiseLock1 == 0){
    noiseLock1 = 1;
    noiseCount1++;
  } else if (noise1 == 0 && noiseLock1 == 1){
    noiseLock1 = 0;
  }
}

void trigger_function2(){
  if (signal2 == 0 && signalLock2 == 0){
    signalLock2 = 1;
    signalRegister2++;
    signalCount2++;
  } else if (signal2 == 1 && signalLock2 == 1){
    signalLock2 = 0;
  }
  if (noise2 == 1 && noiseLock2 == 0){
    noiseLock2 = 1;
    noiseCount2++;
  } else if (noise2 == 0 && noiseLock2 == 1){
    noiseLock2 = 0;
  }
}

void coincident_trigger(){
  if (index % 100 == 0){
    coincidentCount = coincidentCount + min(signalRegister1, signalRegister2);
    signalRegister1 = 0;
    signalRegister2 = 0;
  }
}

void count_coincidence_marker(Print &print=Serial){
  print.print("[PG1]:");
  print.print(signalCount1);
  print.print(" , [PG2]:");
  print.print(signalCount2);
  print.print(" , Coincidence:");
  print.println(coincidentCount);
}

void terminator(Print &print=Serial){
  print.print("Total Time Elapsed (min):");
  print.println(timelimit);
  print.print("Total Counts from Pocket Geiger 1 :");
  print.println(signalCount1);
  print.print("Total Counts from Pocket Geiger 2 :");
  print.println(signalCount2);
  print.print("Total coincidences obtained from Pocket Geiger 1 and 2 :");
  print.println(coincidentCount);
  print.print("Noise signal Counts from Pocket geiger 1 :");
  print.println(noiseCount_record1);
  print.print("Noise signal Counts from Pocket geiger 2 :");
  print.println(noiseCount_record2);
  print.print("!! ACQUISITION ENDS NOW !!");
  while(1)
  {

  }
}

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("<< Arduino Program Initiated >>");
  pinMode(signalPin1, INPUT);
  digitalWrite(signalPin1, HIGH);
  pinMode(noisePin1, INPUT);
  digitalWrite(noisePin1, HIGH);
  pinMode(signalPin2, INPUT);
  digitalWrite(signalPin2, HIGH);
  pinMode(noisePin2, INPUT);
  digitalWrite(noisePin2, HIGH);
  Serial.println("Pocket Geiger Setup Loaded...");
  Serial.println("!! DATA ACQUISITION INITIATED !!");
}

void loop() {
  signal1 = digitalRead(signalPin1);
  signal2 = digitalRead(signalPin2);
  noise1 = digitalRead(noisePin1);
  noise2 = digitalRead(noisePin2);

  counter = random();
  
  if (counter % 2 == 0){
    trigger_function1();
    trigger_function2();
  } else {
    trigger_function2();
    trigger_function1();
  }
  
  coincident_trigger();

  if (index == 10000){
    if (noiseCount1 == 0 && noiseCount2 == 0){
      //count_coincidence_marker();
    } else if (noiseCount1 == 1){
      noiseCount_record1++;
    } else if (noiseCount2 == 1){
      noiseCount_record2++;
    }
    noiseCount1 = 0;
    noiseCount2 = 0;
    index = 0;    
  }
  index++;
  if ((millis()/1000)/60 - Timer >= timelimit){
    terminator();
  }
}
