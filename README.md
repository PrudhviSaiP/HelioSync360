# HelioSync360

# PROJECT HelioSync360 [Smart Dual Axis Solar Tracker]
![Screenshot 2025-02-14 022425](https://github.com/user-attachments/assets/9a655418-36ca-4368-b4d8-b3b4906c941d)


###### ABSTRACT : This project implements a dual-axis solar tracker using a servo motor controlled by the CH32V003F4U6 microcontroller with a 32-bit RISC-V core. The system adjusts solar panel orientation to maximize energy absorption, ensuring efficiency through servo control enhancing sustainability .
***
### OVERVIEW OF THE PROJECT
***

https://github.com/user-attachments/assets/080114c1-6a1c-4703-8555-eab8bd60dfa8


***
###### => video presentation
https://github.com/vardanchettri/Hackathon_2025/blob/main/presentation.mp4
***
### Key Features 
* Automatic Dual-Axis Tracking: Constantly adjusts solar panel orientation (azimuth and elevation) for maximum sunlight exposure.
* Smart Microcontroller: Built on the 32-bit RISC-V core  for low power consumption and high computational performance.
* Efficient Servo Motor Control: Utilizes real-time feedback for precise, low-power motor movements.
* Energy Optimization: Increases solar panel efficiency by up to 30%, lowering operational costs in the long term.
# Why This Matters 
#### Scalable and Cost-Effective
###### => "SCALABLE" , Perfect for small to medium-sized solar installations, easily scalable to meet growing energy needs.
#### Low Maintenance
###### => This design "minimizes manual intervention", saving time and labor costs.
***
# How It Works
*  Sensors detect sunlight intensity
* Servo motors adjust the solar panel's position
* Microcontroller processes feedback and ensures alignment
***
# Requirements 
###### HARDWARE
* VSD SQUADRON MINI
* SOLAR PANEL
* SERVO MOTORS
* DIODES,TRANSISTER
* POTENTIOMETER 
###### SOFTWARE
* PLATFORM.IO
* VISUAL STUDIO CODE
* FRAMEWORK : ARDUINO
* LANGUAGE : C++
***
# SCHEMATIC DIAGRAM
![image](https://github.com/user-attachments/assets/bf3c1d5e-9929-4610-903f-394a5858a67a)

***

## WORKING OF THE CODE(LOGIC)

###### FLOW CHART :

![Screenshot 2025-02-28 231532](https://github.com/user-attachments/assets/64912894-b02c-4019-ac04-a48575ea0aa0)
![Screenshot 2025-02-28 231724](https://github.com/user-attachments/assets/bdcaca75-f629-4827-acbd-3b914fea20da)




##### code
```

#include <ch32v00x.h>
#include <stdint.h>

#define LED_PIN PD1
#define SERVO_PIN PC1
#define SERVO_PIN2 PC2

void ADC_Function_Init(void) {
    ADC_InitTypeDef  ADC_InitStructure = {0};
    GPIO_InitTypeDef GPIO_InitStructure = {0};

    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOD, ENABLE); 
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
    RCC_ADCCLKConfig(RCC_PCLK2_Div8);

    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4;
    GPIO_Init(GPIOC, &GPIO_InitStructure);
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_2;
    GPIO_Init(GPIOD, &GPIO_InitStructure);
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_3;
    GPIO_Init(GPIOD, &GPIO_InitStructure);
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4;
    GPIO_Init(GPIOD, &GPIO_InitStructure);

    ADC_DeInit(ADC1);
    ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
    ADC_InitStructure.ADC_ScanConvMode = DISABLE;
    ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;
    ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
    ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
    ADC_InitStructure.ADC_NbrOfChannel = 1;
    ADC_Init(ADC1, &ADC_InitStructure);

    ADC_Cmd(ADC1, ENABLE);
    ADC_ResetCalibration(ADC1);
    while (ADC_GetResetCalibrationStatus(ADC1));
    ADC_StartCalibration(ADC1);
    while (ADC_GetCalibrationStatus(ADC1));
}

uint16_t Get_ADC_Val(uint8_t ch) {
    ADC_RegularChannelConfig(ADC1, ch, 1, ADC_SampleTime_241Cycles);
    ADC_SoftwareStartConvCmd(ADC1, ENABLE);
    while (!ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC));
    return ADC_GetConversionValue(ADC1);
}

void setup() {
    Serial.begin(9600);
    ADC_Function_Init();
    pinMode(LED_PIN, OUTPUT);
    pinMode(SERVO_PIN, OUTPUT);
    pinMode(SERVO_PIN2, OUTPUT);
}

int lastServoAngle1 = 90;
int lastServoAngle2 = 90;

void smoothServoMove(int servoPin, int *lastAngle, int targetAngle) {
    if (*lastAngle == targetAngle) return;
    
    int step = (targetAngle > *lastAngle) ? 1 : -1;
    while (*lastAngle != targetAngle) {
        *lastAngle += step;
        int pulseWidth = (*lastAngle * 2000 / 180) + 500;
        digitalWrite(servoPin, HIGH);
        delayMicroseconds(pulseWidth);
        digitalWrite(servoPin, LOW);
        delay(20); 
    }
}

void loop() {
    uint16_t ADC_val1 = Get_ADC_Val(ADC_Channel_2);
    uint16_t ADC_val2 = Get_ADC_Val(ADC_Channel_3);
    uint16_t ADC_val3 = Get_ADC_Val(ADC_Channel_4);
    uint16_t ADC_val4 = Get_ADC_Val(ADC_Channel_7);
    uint16_t avg12 = (ADC_val1 + ADC_val2) / 2;
    uint16_t avg34 = (ADC_val3 + ADC_val4) / 2;
    uint16_t avg14 = (ADC_val1 + ADC_val4) / 2;
    uint16_t avg23 = (ADC_val2 + ADC_val3) / 2;
    uint16_t avg1234 = (ADC_val1 + ADC_val2 + ADC_val3 + ADC_val4) / 4;
    uint16_t tolerance = 10;
    int targetServo1 = lastServoAngle1;
    int targetServo2 = lastServoAngle2;
    Serial.print("LDR1: "); Serial.println(ADC_val1);
    Serial.print("LDR2: "); Serial.println(ADC_val2);
    Serial.print("LDR3: "); Serial.println(ADC_val3);
    Serial.print("LDR4: "); Serial.println(ADC_val4);

    if (abs(avg12 - avg1234) > tolerance && avg12 < avg1234) targetServo1 += 5;
    else if (abs(avg34 - avg1234) > tolerance && avg34 < avg1234) targetServo1 -= 5;
    
    if (abs(avg14 - avg1234) > tolerance && avg14 < avg1234 && targetServo1 <= 90) targetServo2 += 5;
    else if (abs(avg23 - avg1234) > tolerance && avg23 < avg1234 && targetServo1 <= 90) targetServo2 -= 5;

    if (abs(avg14 - avg1234) > tolerance && avg14 < avg1234 && targetServo1 > 90) targetServo2 -= 5;
    else if (abs(avg23 - avg1234) > tolerance && avg23 < avg1234 && targetServo1 > 90) targetServo2 += 5;


    if (avg1234 > 950) {
      targetServo1 = 90;
      targetServo2 = 90;
    }

    targetServo1 = constrain(targetServo1, 50, 140);
    targetServo2 = constrain(targetServo2, 0, 180);

    smoothServoMove(SERVO_PIN, &lastServoAngle1, targetServo1);
    smoothServoMove(SERVO_PIN2, &lastServoAngle2, targetServo2);
    delay(100);
}





```

***

# Demonstration


https://github.com/user-attachments/assets/9ca694ca-0e59-4f70-ab6a-a6b1d8c19e4a


***
# AKNOWLEDGEMENT

We would like to extend my sincere gratitude to the RISC-V support team for their unwavering assistance and for always being available to resolve any technical challenges. Our deepest thanks go to our professor, whose guidance and encouragement have been instrumental throughout this project. We equally grateful to Bhagwan for His divine blessings, and to all my friends and elders whose steadfast support and wisdom have enriched my journey. This collaborative spirit has been fundamental in bringing this project to fruition.

Team : 
Akhil ,Vardan , Prudhvi Sai , Vishnu ,  Kabiraj

***
## 📜 License  
This project is licensed under the **MIT License** – see the [LICENSE](LICENSE) file for details.




 
