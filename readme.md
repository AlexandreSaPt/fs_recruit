# Recruits Task - Week #2
- ## [git para noobs](https://hackmd.io/@PedroRomao/HJ0GJSae1x)
- ## Add this file to your local git repository create a new branch and work on it, when you are done or as you complete the questions merge the branch into the main branch (remote main branch will have your final work, this means main branch on GitHub, please make the repository public for the next weeks).
- ## Perform the following tasks to the best of your hability, sometimes, in the questions, there are multiple answers just tell us what you think, feel free to use the group to ask questions.

### 1
**1.** Check out the [el-sw repository](https://github.com/fs-feup/el-sw/tree/main) code and documentation  and try to generally understand what the software does in each device (there is no need to understand all the little details).
### 2
When we read values from the brake sensor (C1) and the apps (C3) we do not use the most recent reading and use instead a different approach. Explain the approach and why you think it is used.

**Answer:** 
When reading the values from the brake sensor and the APPS, the program performs what is referred in the Brake Light Diagram as "Moving Average".

![diagram-bl](./diagram-bl.png)

This approach, instead of using the value that is read by the sensors in that moment, it uses the average of a sample (of which size is expressed by the macro AVR_SAMPLE).
This sample is created using the FIFO principle (a queue). It stores the last (AVR_SAMPLES - 1) samples, adds the value read by the sensor and performs the average. 
This, I believe, is used to attenuate the occurrence of "spikes" in the data, moments where the value of the sensor could be somewhat wrong. With this, we make sure that there is some corroboration in the data set.
Here we have the code snippets that read the brake sensor and the APPS respectively, where we can see the code reading from the sensors, inserting it to the buffer that holds the rest of the dataset and finally performing the average.




**Brake Sensor**
```c++
        #define SENSOR_SAMPLE_PERIOD 20    // ms
        #define AVG_SAMPLES 20

        ...

        brake_sensor_timer = 0;
        brake_val = analogRead(BRAKE_SENSOR_PIN);
        bufferInsert(avgBuffer1, AVG_SAMPLES, brake_val);
        brake_val = average(avgBuffer1, AVG_SAMPLES);

```

**APPS**
```c++
        #define APPS_READ_PERIOD_MS 20      // ms
        #define AVG_SAMPLES 5

        ...

        int v_apps1 = analogRead(APPS_1_PIN);
        int v_apps2 = analogRead(APPS_2_PIN);

        bufferInsert(avgBuffer1, AVG_SAMPLES, v_apps1);
        bufferInsert(avgBuffer2, AVG_SAMPLES, v_apps2);

        v_apps1 = average(avgBuffer1, AVG_SAMPLES);
        v_apps2 = average(avgBuffer2, AVG_SAMPLES);
        sendAPPS(v_apps1,v_apps2);
```

### 3
Check out the R2D(Ready To Drive) code on the C3 state machine. In the condition below we use a timer (R2DTimer) to check the brake was engaged instead of just checking the brake pressure received from can, why?
```c++
        if ((r2dButton.fell() and TSOn and R2DTimer < R2D_TIMEOUT) or R2DOverride)
        {
            playR2DSound();
            initBamocarD3();
            request_dataLOG_messages();
            R2DStatus = DRIVING;
            break;
        }
```

**Answer:**
In the if statement above, in order to switch the *R2DStatus* to *DRIVING*, not only has the TS to be on, but the *r2dButton* has to be pressed at the same time as the brakes. For the brakes to "count" as pressed, the value read has to be grater than 165 and regarding the button, the method *fell()* returns if the button signal has gone from LOW to HIGH since the last call of the method *update()*.
This may originate a problem since this can be a little time sensitive in the sense that the brakes were pressed but when you take your hand off the button, the brake pressure may be now below that threshold. So, in order to remove this "bad timing", the *R2DTimer* along with its *R2D_TIMEOUT* has been implemented. Now, whenever the brake is pressed passed that threshold, the timer (which is an object from the *ElapsedMillis* class) is set to 0, now giving the driver R2D_TIMEOUT milliseconds to release the *r2dButton*.

**Resetting the R2DTimer**
```c++

        void canSniffer(const CAN_message_t& msg) {
            switch (msg.id) {
                ...
                case C3_ID:
                    brakeValue = (msg.buf[2] << 8) | msg.buf[1];
                    if (brakeValue > 165)
                        R2DTimer = 0;
                    break;
                ...
            }
        }
```


### 4
What is the ID of the can message sent to the bamocar to request torque?
**Answer:**
According to the *CAN Table*, which is present in the Team's Google Drive, we can see a row with the following comment "Torque Request to Bamocar". Therefor, the ID of the CAN message sent to the bamocar to request torque is 0x201.

### 5 
The code below is not amazing, tell us some things you would change to improve it, you can write them down in text or correct the code:
```c++
// this is a class for my car
class mycar {
private:
    int sensor_reading1; // hydraulic pressure sensor
    int sensor_reading2; // temperature sensor
    int sensor_reading3; // humidity sensor
    int sensor_reading4; // light sensor
    int sensor_reading5; // sound sensor
    int sensor_reading6; // distance sensor
    int sensor_reading7; // accelerometer sensor
    int sensor_reading8; // gyroscope sensor

    int sensor_reading9; // old sensor, not used anymore

public:
    mycar() : sensor_reading1(0), sensor_reading2(0), sensor_reading3(0), sensor_reading4(0),
            sensor_reading5(0), sensor_reading6(0), sensor_reading7(0), sensor_reading8(0) {}

    // Method will update readings by analog reading and print them 
    void updateprint() {
        sensor_reading1 = analogRead(0); // pin 0 is connected to the hydraulic pressure sensor
        sensor_reading2 = analogRead(1); // pin 1 is connected to the temperature sensor
        sensor_reading3 = analogRead(2); // pin 2 is connected to the humidity sensor
        sensor_reading4 = analogRead(3); // pin 3 is connected to the light sensor
        sensor_reading5 = analogRead(4); // pin 4 is connected to the sound sensor
        sensor_reading6 = analogRead(5); // pin 5 is connected to the distance sensor
        sensor_reading7 = analogRead(6); // pin 6 is connected to the accelerometer sensor
        sensor_reading8 = analogRead(7); // pin 7 is connected to the gyroscope sensor
        func(sensor_reading1, sensor_reading2, sensor_reading3, sensor_reading4, 
              sensor_reading5, sensor_reading6, sensor_reading7, sensor_reading8);// print the readings
    }

    // function to print the readings of the sensors
    void func(int sensor_reading1, int sensor_reading2, int sensor_reading3, int sensor_reading4, 
              int sensor_reading5, int sensor_reading6, int sensor_reading7, int sensor_reading8) {
        Serial.print("Sensor Reading 1: "); Serial.println(sensor_reading1);
        Serial.print("Sensor Reading 2: "); Serial.println(sensor_reading2);
        Serial.print("Sensor Reading 3: "); Serial.println(sensor_reading3);
        Serial.print("Sensor Reading 4: "); Serial.println(sensor_reading4);
        Serial.print("Sensor Reading 5: "); Serial.println(sensor_reading5);
        Serial.print("Sensor Reading 6: "); Serial.println(sensor_reading6);
        Serial.print("Sensor Reading 7: "); Serial.println(sensor_reading7);
        Serial.print("Sensor Reading 8: "); Serial.println(sensor_reading8);
        //all readings were serial printed
    }
};
```

The first problem I get with this class declaration is the name of the properties. Eventhought the code has comments in front of each property, when we use it in the rest of the code, it will start to be difficult to understand the code without the constant need to check for this declaration. So, I'll begin be changing their names to a more "readable" one. Also, there is a property with a comment saying that that property isn't used anymore and doesn't appear in the rest of the code, so I will be removing it. I also believe that there should be a comment in from of each property specifying the units of the reading of each sensor and also the reading interval. Since I don't know those values in the context of this class, I will not be putting any of those in my example.

Moreover, the *updateprint()* method has a lot of things I'd like to address.
To start with, the pin number in the *analogRead()* function have 2 issues for me. It is not explicit what those numbers are and, in case you want to change them in the future, you would have to go after what numbers you would have to change in the rest of the code, which not only in unnecessary work but it also makes it easier to mess up. So, to correct it, I would create some macros with the pin number. I will be naming them in uppercase since it is common practice when declaring values that don't change during execution.
In this method, we also have 2 things going on inside it: updating and printing. I will start by changing the printing method's name to *printReadings()*, since "func" doesn't really tell us much about the function. This time, I don't want this function to have arguments since the arguments were the object's properties and you have access to those already and this way the code becomes more readable. With this done, I would go ahead and crate a new method called *updateReadings()*, that would take care of the update logic, making it easier to debug if necessary, and add it to the *updateprint()* function, so that it only calls these 2 functions.
Now regarding the Class Constructor, although I find it somewhat harder to understand, I believe it is because of my lack of experience with it and since it is a faster way to initialize a class, I will leave it there as is, just making some paragraphs and tabs to make it more readable.
Taking a closer look at the now named *printReadings()* method, being more familiar with the "printf" syntax, the way the "printing" is done in this code does bother me, but couldn't find a better way to do it.
The one thing that needs fixing now is the comments. With all of the changes that I have made, some comments became redundant and there is also some comments I'd like to put, just to clarify this a little bit more. We can use some of the standardized tags for this, just like in your documentation.
Here is the code with the said changes.

```c++
#define HYDRAULIC_PRESSURE_PIN 0
#define TEMPERATURE_PIN 1
#define HUMIDITY_PIN 2
#define LIGHT_PIN 3
#define SOUND_PIN 4
#define DISTANCE_PIN 5
#define ACCELEROMETER_PIN 6
#define GYROSCOPE_PIN 7


/**
 * @brief This class represents a car and is used to read the values from the sensors and print them to the serial monitor
*/

class mycar {
private:
    int hydraulic_pressure_sensor; // variable to store the value from the hydraulic pressure sensor
    int temperature_sensor; // variable to store the value from the temperature sensor
    int humidity_sensor; // variable to store the value from the humidity sensor
    int light_sensor; // variable to store the value from the light sensor
    int sound_sensor; // variable to store the value from the sound sensor
    int distance_sensor; // variable to store the value from the distance sensor
    int accelerometer_sensor; // variable to store the value from the accelerometer sensor
    int gyroscope_sensor; // variable to store the value from the gyroscope sensor


public:
    /**
     * @brief Constructor of new mycar object, initializes all sensor readings to 0
     */
    mycar() : 
        hydraulic_pressure_sensor(0), 
        temperature_sensor(0), 
        humidity_sensor(0), 
        light_sensor(0),
        sound_sensor(0), 
        distance_sensor(0), 
        accelerometer_sensor(0), 
        gyroscope_sensor(0) 
        {}

    /**
     * @brief Function to update the readings of the sensors and print them to the serial monitor
     */
    void updateprint() {
        updateReadings();
        printReadings();
    }

    /**
     * @brief Function to update the readings of the sensors
     */
    void updateReadings(){
        hydraulic_pressure_sensor = analogRead(HYDRAULIC_PRESSURE_PIN);
        temperature_sensor = analogRead(TEMPERATURE_PIN);
        humidity_sensor = analogRead(HUMIDITY_PIN);
        light_sensor = analogRead(LIGHT_PIN);
        sound_sensor = analogRead(SOUND_PIN);
        distance_sensor = analogRead(DISTANCE_PIN);
        accelerometer_sensor = analogRead(ACCELEROMETER_PIN);
        gyroscope_sensor = analogRead(GYROSCOPE_PIN);
    }

    /**
     * @brief Function to print the readings of the sensors to the serial monitor
     */
    void printReadings() {
        Serial.print("Sensor Reading 1: "); Serial.println(hydraulic_pressure_sensor);
        Serial.print("Sensor Reading 2: "); Serial.println(temperature_sensor);
        Serial.print("Sensor Reading 3: "); Serial.println(humidity_sensor);
        Serial.print("Sensor Reading 4: "); Serial.println(light_sensor);
        Serial.print("Sensor Reading 5: "); Serial.println(sound_sensor);
        Serial.print("Sensor Reading 6: "); Serial.println(distance_sensor);
        Serial.print("Sensor Reading 7: "); Serial.println(accelerometer_sensor);
        Serial.print("Sensor Reading 8: "); Serial.println(gyroscope_sensor);
        //all readings were serial printed
    }
};
```
