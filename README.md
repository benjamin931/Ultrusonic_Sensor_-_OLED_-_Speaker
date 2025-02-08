# This script use a HC-SR04 to measure distance. The measurement is displayed in the Shell.

from machine import Pin, PWM, I2C #Pin allows us to communicate with GPIO pins
import utime #Used to introduce a delay into our script
from ssd1306 import SSD1306_I2C
 
# Set up GPIO pins for communication with the sensor
trigger = Pin(16, Pin.OUT)
echo = Pin(17, Pin.IN)

speak = PWM(Pin(18))

W = 128
H = 64

i2c = I2C(1, sda = Pin(14), scl = Pin(15), freq=200000)
oled = SSD1306_I2C(W, H, i2c)

def play_tone(frequency, duration):
    if frequency > 0:
        speak.freq(frequency)
        speak.duty_u16(3000)
        utime.sleep(duration / 1000)
        speak.duty_u16(0)
    utime.sleep(0.01)
C = [261]
D = [294]
E = [329]
F = [349]
G = [392]
A = [440]
B = [494]
H_C = [523]

NOTE_DURATION = 500

try:
    while True:
        # Set the trigger to low so no sound is being transmitted.
        oled.fill(0)
        trigger.low()
        utime.sleep_us(2)
        # Send sound for 5 µs
        trigger.high()
        utime.sleep_us(5)
        trigger.low()
   
        # Echo pin on the sensor is high when sound is heard and low when there is no sound.
        # Echo pin will be low after untill the pulse hits an object and returns.
        # Echo will go high after pulse is detected.
        while echo.value() == 0:
            signaloff = utime.ticks_us()
        while echo.value() == 1:
            signalon = utime.ticks_us()
   
        # The time between signaloff and signalon gives us the travel time of the pulse.
        timepassed = signalon - signaloff
   
        # We only need to know the travel time to the object, so the value is divided by two.
        # The value 0.0343 is based on the speed of sound through air.
        distance_cm = (timepassed * 0.0343) / 2
        if distance_cm/2.54 <= 5:
            for frequency in C:
                play_tone(frequency, NOTE_DURATION)
        elif distance_cm/2.54 < 10:
            for frequency in D:
                play_tone(frequency, NOTE_DURATION)
        elif distance_cm/2.54 < 15:
            for frequency in E:
                play_tone(frequency, NOTE_DURATION)
        elif distance_cm/2.54 < 20:
            for frequency in F:
                play_tone(frequency, NOTE_DURATION)
        elif distance_cm/2.54 < 25:
            for frequency in G:
                play_tone(frequency, NOTE_DURATION)
        elif distance_cm/2.54 < 30:
            for frequency in A:
                play_tone(frequency, NOTE_DURATION)
        elif distance_cm/2.45 < 35:
            for frequency in B:
                play_tone(frequency, NOTE_DURATION)
        else:
            for frequency in H_C:
                play_tone(frequency, NOTE_DURATION)
            
            
            
   
        # Display the measurement in centimeters and inches
        #print("The distance from object is ",distance_cm,"cm")
        print("The distance from object is ",distance_cm/2.54, "in")
        oled.text("The distance", 10, 10)
        oled.text("from object is:", 5, 25)
        oled.text(str(distance_cm/2.54, ), 30, 40)
        oled.text("Inches", 40, 55)
        oled.show()
        utime.sleep(0.5) # Slight delay to read the measurement before restarting the loop
except KeyboardInterrupt:
    oled.fill(0)
    oled.text("There has been", 10, 16)
    oled.text("a Keyboard ", 25, 26)
    oled.text("Interrupt", 30, 36)
    oled.show()
    speak.deinit()
