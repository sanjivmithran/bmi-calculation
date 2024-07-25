# bmi-calculation

#include <HardwareSerial.h>
HardwareSerial Load_Cell (1);

#include<LiquidCrystal.h>
LiquidCrystal  lcd(13,12,14,27,26,25);

#include <Ultrasonic.h>
Ultrasonic ultrasonic(33,32);//TRIG,ECHO
int distance;

#include <EEPROM.h>

#define key1 35
#define key2 34
#define key3 39
#define bmi_key 23


String inputString = "";
bool stringComplete = false;
int load  , Age , a;
float hight,h,hs,w,bmi;
String weight;

void setup() 
{
 pinMode(key1, INPUT_PULLUP);
 pinMode(key2, INPUT_PULLUP);
 pinMode(key3, INPUT_PULLUP);
 pinMode(bmi_key, INPUT_PULLUP);
 inputString.reserve(200);
 Serial.begin(9600);
 Load_Cell.begin(9600,SERIAL_7E1, 19, 18);//RX,TX 

 EEPROM.begin(100);
 Age = EEPROM.read(5);
 
 lcd.begin(16,2);
 lcd.setCursor(0,0); lcd.print("   AUTOMATIC   ");
 lcd.setCursor(0,1); lcd.print("BMI CALCULATION");
 delay(3000);        lcd.clear(); 
}

////////////////////////////////////////////////////
void loop() 
{
 READ_Load_Cell();
 distance = ultrasonic.read();
// Serial.print("Distance in CM: ");
// Serial.println(distance);
 hight=distance;
 
 lcd.setCursor(0 ,0); lcd.print("H:");   lcd_Decimil_3(2 ,0,h);
 lcd.setCursor(6 ,0); lcd.print("W:" + weight);
 lcd.setCursor(0 ,1); lcd.print("A:");   lcd_Decimil_2(2 ,1,Age);
 lcd.setCursor(5 ,1); lcd.print("BMI:"); lcd.setCursor(9 ,1);lcd.print(bmi);
 if(!digitalRead(key1)) { lcd.clear();  while(!digitalRead(key1)); kepad(); delay(1000);}
 if(!digitalRead(bmi_key)) {while(!digitalRead(bmi_key));a++;if(a>1){a=0;}}
 if(a==1)
   {
    w=weight.toFloat();
    hight=195-hight;
    Serial.print("HIGHT : ");
    Serial.println(hight);
    h=hight/100;
    hs=h*h;
    bmi=w/hs;
    lcd.setCursor(0 ,0); lcd.print("H:");   lcd_Decimil_3(2 ,0,hight);
    lcd.setCursor(6 ,0); lcd.print("W:" + weight);
    lcd.setCursor(0 ,1); lcd.print("A:");   lcd_Decimil_2(2 ,1,Age);
    lcd.setCursor(5 ,1); lcd.print("BMI:");lcd.setCursor(9 ,1); lcd.print(bmi);
  /*  Serial.print("METER : ");
    Serial.println(h);
    Serial.print("HS : ");
    Serial.println(hs);
    Serial.print("WEIGHT : ");
    Serial.println(w);
    Serial.print("BMI : ");
    Serial.println(bmi); */
   delay(1000);
   }
  
  
}

///////////////////////////////////////////////////////////////////
void kepad()
{
 while(digitalRead(key1))
 {
  if(!digitalRead(key2)){delay(250); Age++; if(Age>99)Age=0; }
  if(!digitalRead(key3)){delay(250); Age--; if(Age <0)Age=99; }
  lcd.setCursor(0 ,0); lcd.print("Age:");   lcd_Decimil_3(4 ,0,Age);
 }
 lcd.clear();
 EEPROM.write(5,Age); 
 EEPROM.commit();  
 while(!digitalRead(key1)); delay(500);
}

////////////////////////////////////////////////////
void READ_Load_Cell()
{
 while ( stringComplete == false && Load_Cell.available()) 
 {
  char inChar = (char)Load_Cell.read();
  inputString += inChar;
  if (inChar == '\n') {stringComplete = true;}
  if(stringComplete){  Load_cell_received();}
  }
}

void Load_cell_received()
{
// Serial.print(inputString); 
 //K+000.040
 if(inputString[1]=='K' && inputString[2]=='+')
   {
    load = ((inputString[3]-'0')*100000) + 
           ((inputString[4]-'0')*10000) + 
           ((inputString[5]-'0')*1000) + 
           ((inputString[7]-'0')*100) + 
           ((inputString[8]-'0')*10) +
           ((inputString[9]-'0')*1) ;
           weight = "" ;
           for(int i=3; i<10; i++){weight+=inputString[i];}
   } 
// Serial.print("\t load =");  Serial.println(load); 
// Serial.print("\t weight ="); Serial.println(weight);  
 
 inputString =""; 
 stringComplete = false;
}


//////////////////////////////////////////////
void lcd_Decimil_3(int col , int row , int num)
{
 lcd.setCursor(col+0,row); lcd.print((num/100)%10);
 lcd.setCursor(col+1,row); lcd.print((num/10)%10);
 lcd.setCursor(col+2,row); lcd.print((num/1)%10);
}

//////////////////////////////////////////////
void lcd_Decimil_2(int col , int row , int num)
{
 lcd.setCursor(col+0,row); lcd.print((num/10)%10);
 lcd.setCursor(col+1,row); lcd.print((num/1)%10);
}
/////////////////////////////////
void Serial_Decimil_2(int num)
{
 Serial.print(num/10);
 Serial.print(num%10);
}

void Serial_Decimil_3(int num , char com)
{
 int ans1,ans2,ans3,a;
 ans1=num/100;
 a=num%100;
 ans2=a/10;
 ans3=a%10;
 Serial.print(ans1);
 Serial.print(ans2);
 Serial.print(ans3);
}
