# DAC

```
#include <driver/dac.h>

#define DAC_PIN 25
#define NFET_PIN 26
#define NFET_THRESHOLD 1.2 // NFET阈值电压，单位为V

void setup() {
  pinMode(DAC_PIN, INPUT);
  pinMode(NFET_PIN, OUTPUT);
  Serial.begin(115200);
}

void loop() {
  for (int dac_value = 0; dac_value <= 255; dac_value++) {
    float v_out = dac_value * 3.3 / 255;

    if (v_out >= NFET_THRESHOLD) {
      analogWrite(NFET_PIN, HIGH); 
      Serial.printf("v_out_thres= %.2f V\r\n", v_out);
    } 
    else {
      analogWrite(NFET_PIN, LOW); 
    }
    // Serial.printf("v_out = %.2f V\r\n", v_out);
  }

  analogWrite(NFET_PIN, LOW); 
  delay(1000);  
}
```
v_out_thres= 3.16 V <br>
v_out_thres= 3.17 V <br>
v_out_thres= 3.18 V <br>
v_out_thres= 3.20 V <br>
v_out_thres= 3.21 V <br>
v_out_thres= 3.22 V <br>
v_out_thres= 3.24 V <br>
v_out_thres= 3.25 V <br>
v_out_thres= 3.26 V <br>
v_out_thres= 3.27 V <br>
v_out_thres= 3.29 V <br>
v_out_thres= 3.30 V <br>
v_out_thres= 1.20 V <br>
v_out_thres= 1.22 V <br>
v_out_thres= 1.23 V <br>
v_out_thres= 1.24 V <br>
v_out_thres= 1.26 V <br>
v_out_thres= 1.27 V <br>


# ADC
```
#include <esp32-hal-adc.h>

#define ADC_UP_PIN   34 
#define ADC_DOWN_PIN 35

float resistance = 4.1; 

void setup() {
  pinMode(ADC_UP_PIN, INPUT);
  pinMode(ADC_DOWN_PIN, INPUT);
  Serial.begin(115200);
}

void loop() {
  int adc_up_raw = analogRead(ADC_UP_PIN);
  int adc_down_raw = analogRead(ADC_DOWN_PIN);

  float adc_up_voltage = adc_up_raw * (3.3 / 4095); 
  float adc_down_voltage = adc_down_raw * (3.3 / 4095); 
  float voltage = adc_up_voltage - adc_down_voltage;
  float current = voltage / resistance;

  Serial.print("Voltage: ");
  Serial.println(voltage, 8); // 保留两位小数输出电压
  Serial.print("Current: ");
  Serial.println(current, 8); 

  delay(1000); // 延迟一秒
}
```
Voltage: 0.00322345 <br>
Current: 0.00078621 <br>
Voltage: 0.00000000 <br>
Current: 0.00000000 <br>
Voltage: 0.00161174 <br>
Current: 0.00039311 <br>
Voltage: -0.00564101 <br>
Current: -0.00137586 <br>
Voltage: -0.00483516 <br>
Current: -0.00117931 <br>
Voltage: 0.00161174 <br>
Current: 0.00039311 <br>
Voltage: 0.00161171 <br>
Current: 0.00039310 <br>


# System
```
#include <esp32-hal-adc.h>
#include <driver/dac.h>

#define ADC_UP_PIN   34 
#define ADC_DOWN_PIN 35 
#define DAC_PIN 25
#define NFET_PIN 26
#define NUM_POINTS 255

float resistance = 5;
float current_values[NUM_POINTS];
float voltage_values[NUM_POINTS];


void setup() {
  analogReadResolution(12); // 设置ADC分辨率为12位
  pinMode(DAC_PIN, INPUT);
  pinMode(NFET_PIN, OUTPUT);
  pinMode(ADC_UP_PIN, INPUT);
  pinMode(ADC_DOWN_PIN, INPUT);
  Serial.begin(115200);

}

void loop() {
  Serial.printf("- - - - - - - \r\n");
  Serial.println("Starting measurement...\r\n");

  int index=0;
  for (int i = 105;i < NUM_POINTS; i = i + 10){    
    float dac_in = i;
    dacWrite(DAC_PIN, dac_in); 
    float dac_out = dac_in * (3.3/ 255);
    dacWrite(NFET_PIN, dac_out); 
    delay(100);

    float up_voltage = (float)analogRead(ADC_UP_PIN) * (3.3 / 4095); 
    // Serial.printf("%.8f \r\n", up_voltage);
    float down_voltage = (float)analogRead(ADC_DOWN_PIN) * (3.3 / 4095); 
    float current = (down_voltage - up_voltage) / resistance; 
    if(current < 0){
      current = -current;
    }
    // Serial.printf("%.8f \r\n", current);
    voltage_values[index] = up_voltage;
    current_values[index] = current;
    index = index +1;
    if(index == (NUM_POINTS - 105) / 10){
      index = 0;
    }
  }

  for(int i=0; i< 10; i++){
    Serial.printf("%.8f \r\n", voltage_values[i]);
  }

  Serial.printf("* * * * * * * *\r\n");

  for(int i=0; i< 10; i++){
    Serial.printf("%.8f \r\n", current_values[i]);
  }

  delay(1000);  
  Serial.println("Ending measurement...");

}


```


