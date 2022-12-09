# physical-computing

### Week5-Ideation

For this project I want to make an band through Arduino in which people can experience the fun in conducting. I think this installation will be extremely cool to create. People can conduct through the control of the stick. The project seemed very interesting to me, but it was difficult for me to make it work well alone. yihan has a great visual ability and we decided to work together as a team to make this project better.

At first I was going to do realistic bands that people could conduct. yihan said that if it was a simulated band, the fun would be a little more generic. He came up with a super fun theme - a vegetable band. People could conduct a vegetable band and it would be more fun to play with.I thought it was a really brilliant idea, so we hit it off and started to build a scheme for the project.

The idea is that the player can control the music by waving the stick and directing three or four vegetable players, to control the tempo, intensity and so on, like a real conductor. When the stick is waved, the tempo of the music BMP will make an adjustment according to the speed of the stick, and as soon as the stick is pointed at a single vegetable, it will make that part louder.

![38151670542571_ pic](https://user-images.githubusercontent.com/112803802/206593683-0e1e9cb5-1e91-4881-8f09-e6b97a7dd8d8.jpg)

* Input1: Photoresistors

* Input2: Gyroscope

* Output1: Oled

* Output2: Speaker

The essence of command is the baton, so we wanted to use the gyroscope as an input to a normal stick so that we could get information about the position of the baton, the speed and so on, and it would look like the stick was conducting.As it requires two inputs and outputs, we added a photo resistor to detect whether the stick is picked up and thus whether an interactive state is enabled. We also added an oled display to show the state of the vegetables, if the conductor is too fast then the fruit is also a very tired state, so we also wanted to give this feedback to the experience with a screen, which I think is one of the most very fun parts.

![OLEO](https://user-images.githubusercontent.com/112803802/206622740-4c4328a7-42fe-40f8-a480-07712cb6799c.jpeg)

### Week6-Sensor choose & Separate experimentation

In week six we started testing the individual components, then yihan was responsible for one input and one output and I was responsible for one input and one output.

The input and output components I was responsible for were the gyroscope and the MP3 music playback component. For the gyroscope sensor component, we chose three components that could have gyroscope functionality for comparison testing.

* A: ADXL343 Accelerometer

* B: Adafruit BNO055 Absolute Orientation Sensor

* C: Arduino BLE Sense

When I ran the test I found that A is only able to get the acceleration, not the position, and C is one of the Arduino's main boards, which is different from the more common Arduino boards when it comes to wiring, making it easy to find the answer to some of the questions.

For all these reasons, we have chosen B as the sensor component we will use.

```
#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BNO055.h>
#include <utility/imumaths.h>

uint16_t BNO055_SAMPLERATE_DELAY_MS = 10; //how often to read data from the board
uint16_t PRINT_DELAY_MS = 500; // how often to print the data
uint16_t printCount = 0; //counter to avoid printing every 10MS sample

//velocity = accel*dt (dt in seconds)
//position = 0.5*accel*dt^2
double ACCEL_VEL_TRANSITION =  (double)(BNO055_SAMPLERATE_DELAY_MS) / 1000.0;
double ACCEL_POS_TRANSITION = 0.5 * ACCEL_VEL_TRANSITION * ACCEL_VEL_TRANSITION;
double DEG_2_RAD = 0.01745329251; //trig functions require radians, BNO055 outputs degrees


/* Set the delay between fresh samples */
#define BNO055_SAMPLERATE_DELAY_MS (100)

// Check I2C device address and correct line below (by default address is 0x29 or 0x28)
//                                   id, address
Adafruit_BNO055 bno = Adafruit_BNO055(55, 0x28);

/**************************************************************************/

    //Arduino setup function (automatically called at startup)

/**************************************************************************/
void setup(void)
{
  Serial.begin(9600);
  Serial.println("Orientation Sensor Test"); Serial.println("");

  /* Initialise the sensor */
  if(!bno.begin())
  {
    /* There was a problem detecting the BNO055 ... check your connections */
    Serial.print("Ooops, no BNO055 detected ... Check your wiring or I2C ADDR!");
    while(1);
  }

  delay(1000);

  bno.setExtCrystalUse(true);
}

/**************************************************************************/
/*
    Arduino loop function, called once 'setup' is complete (your own code
    should go here)
*/
/**************************************************************************/
void loop(void)
{
  
  /* Get a new sensor event */
  sensors_event_t event;
  bno.getEvent(&event);

  unsigned long tStart = micros();
  sensors_event_t orientationData , linearAccelData;
  bno.getEvent(&orientationData, Adafruit_BNO055::VECTOR_EULER);
  //  bno.getEvent(&angVelData, Adafruit_BNO055::VECTOR_GYROSCOPE);
  bno.getEvent(&linearAccelData, Adafruit_BNO055::VECTOR_LINEARACCEL);

  /* Display the floating point data */
  //Serial.print("X: ");
  // Serial.print(event.orientation.x, 4);
  // velocity of sensor in the direction it's facing
  headingVel = ACCEL_VEL_TRANSITION * linearAccelData.acceleration.x / cos(DEG_2_RAD * orientationData.orientation.x)*100;
  if (headingVel==0.00)Serial.print("AAAAAAAAA");
  Serial.print("Speed: ");
  Serial.print(headingVel);
   Serial.print("       ");
  int n=event.orientation.x;
  if (n<=100)
  Serial.print("Left ");
  else if (100<n&&n<=170)
  Serial.print("Middle ");
  else if (170<n&&n<=250)
  Serial.print("Right");
  // Serial.print("\tY: ");
  // Serial.print(event.orientation.y, 4);
  // Serial.print("\tZ: ");
  // Serial.print(event.orientation.z, 4);

  // Serial.print("\tRate: ");
  // Serial.print(event.orientation.x/event.orientation.y, 4);
  /* Optional: Display calibration status */
  //displayCalStatus();

  /* Optional: Display sensor status (debug only) */
  //displaySensorStatus();

  /* New line for the next sample */
  Serial.println("");

  /* Wait the specified delay before requesting nex data */
  delay(BNO055_SAMPLERATE_DELAY_MS);
}
```

During the testing of the BNO055, I have not been able to display the data properly. After a series of troubleshooting, I found that it was due to a mistake in the wiring of the library: because in the sample given by the library, SCL and SDA are plugged in above the two data jacks 2 and 3, but in reality there are two signal jacks SCL and SDA on the Leonardo board, so it should be the Arduino uno board that is connected to 2 and 3. Due to the different board models and the differences between the Arduino motherboards, this problem really bothered me for many hours.

<img width="1400" alt="IMG_4012" src="https://user-images.githubusercontent.com/112803802/206614169-79ea5fa6-5f81-4735-967e-f16c3f88f489.png">

For the part where the sound comes out, I chose the mp3 component between the buzzer and the mp3 component, as the buzzer produces a relatively simple sound and cannot produce a more beautiful sound.

Also on the mp3 component we had a big problem. We needed to process multiple tracks of sound, but the mp3 component could only play one audio from the list individually, like an mp3, and could not meet our needs, but this is already relatively intelligent sound processing component. So we ended up with a multi-track sound that we could combine into one audio file for output.

Here's a very funny joke: because I thought the usb and headphone cable plugged into one as both signal input and power supply, but they are actually separate :), the usb is the power supply and the headphone cable is the signal input. I wasted a bit of time because of my stupidity and even thought the components were broken.

```
void setup() {
  // Start the serial port at 38.4K
  Serial1.begin( 38400 );

  // Set volume
  Serial1.print( "v" );
  Serial1.write( 0 ); // 0 = maximum volume, 255 = minimum volume
}

void loop() {
  // Loop from 1 to 5
  for ( int i = 1; i <= 5; i++ ) {
    // Play file i ( 1 to 5 )
    Serial1.print( "p" );
    Serial1.write( i );

    // Wait a bit
    delay( 1000 );
  }
}
```

### Week7-Code integration

Since the integration part of the code and some of the production part together would have been more time consuming instead, we decided to divide the work and I was responsible for the integration part of the code. I think the biggest difficulty with the integration of the code is that if you get them all running at the same time in one file, the next step is to get them to adapt to the desired logic.

The first difficult thing happened at the beginning, because after combining them in one code, the computer could not get to the Arduino. I found out that it was a problem with the power supply, as there were too many components to make a proper power supply. I considered an external power supply with batteries to keep them running properly. 

![151670556380_ pic](https://user-images.githubusercontent.com/112803802/206617512-37260921-80be-40eb-ac68-7b8852f818bc.jpg)

Here is the little led bulb that gave its life to check if the battery was charged (because I forgot to connect the resistor when I connected the battery, which caused it to burn out the moment it was powered up, and also showed that the battery was still charged.) Here's a half-minute silence for the little bulb and its dedication to my project.

But the battery I have is 9v, the power supply that the Arduino is connected to above the components is 5v, the sensor is borrowed from the college and I am very afraid of burning out the components because of the battery voltage, I see that the mp3 components support a separate power supply from 5v-12v. So I used a 9v power supply to power the mp3 components separately, but the problem with the separate power supply was that the mp3 did not produce sound, so the separate power supply did not solve the problem either. Then I found out that the vin port on the Arduino board can be used for one positive output, so when I split the three components into three parts for positive and negative output input, it was enough to support the power supply, and then the power supply problem was solved.

The MP3 component also had a problem with the code synthesis not ringing, and later after checking it was found to be a problem with serial: because when set up is set up it will initialise Serial.begin(9600), but mp3 is set up separately Serial1.begin(9600) is initialised, and then 1 and l in the Arduino font are very similar looking The initialization of the mp3 is a separate set initialization, which I didn't notice when I was doing it, and found out after a long time of troubleshooting that they need to be set up separately.

### Week8-Solder

At first we misunderstood what the solder wire was for. We thought the purpose of the solder wire was to transfer the breadboard part to the solder job, then as we hadn't made the box size we needed to allow for more length of wire, so that led to a very messy soldering of the wire and then just soldering the positive and negative parts all over it, as the misunderstanding interpreted it as a replacement for the breadboard. 

![3971670559512_ pic](https://user-images.githubusercontent.com/112803802/206623092-e7b6d145-b4b3-4e23-bfb9-923ea3eb221d.jpg)

Later we had a tutorial with Phoenix, who told us that we needed to solder some of the original parts that needed to be fixed, and then we made a prototype box and soldered the wires on the basis of a preset position.

![3981670559583_ pic](https://user-images.githubusercontent.com/112803802/206623232-aed1ac15-3aa6-4a29-9ade-62925c147601.jpg)

I also tested the code during the week, because pointing to different locations will have different data, so we need to connect the locations to the data, but at the same time this connection is more complicated and tedious work, and then I spent a lot of time doing this work.

![IMG_4011](https://user-images.githubusercontent.com/112803802/206621707-fd4f92ee-31e8-4afb-b7a7-da7950caa7d3.jpeg)

I used wasabi disposable chopsticks as a command stick to simulate a command state, which worked very well because its length and width were perfect for the whole experience of connecting to the components, which worked very well.

### Week9-Detail update & Suitable housing make

This week we've basically run through the code and started on the appearance part of the production, yihan led me to draw a very nice box for it and then went to put it together, and also the sound part, we would feel say a little bit not very nice.One of my friends said it sounded like a fart, and then we iterated the sound, and at first we thought that one of the controls for the sound changed with the individual sounds. Then we thought we could loop it and loop each note and control the individual syllables, so that the elements of the sound could be more diverse and richer and people could feel the benefits of a piece of music.

![161670561097_ pic](https://user-images.githubusercontent.com/112803802/206625819-4566b74a-9d1c-4962-a3f7-e8a4c71aaec1.jpg)

<img width="1016" alt="截屏2022-12-09 04 58 12" src="https://user-images.githubusercontent.com/112803802/206627314-12aee3ff-ec08-4d18-be98-532bcc095103.png">

###### Thinking：This project has made me feel the power of collaboration. Everyone has their own strengths, and when the group will play their own strengths, it will make the project better, because one person is limited, and it also made me feel that many people think differently. Some students would choose to do the installation first and then the code, we would choose to do the code first and then the appearance, but it's not as different as I thought, it's probably more like walking on the left foot first and walking on the right foot first, only two feet, walking at the same time can get a good result, one leg can't jump far.
