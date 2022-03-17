# raspberry-pioneers



from gpiozero import DigitalInputDevice, Buzzer
import RPi.GPIO as GPIO
import time
import os
import glob

class DS18B20():
    def __init__(self):
        os.system('modprobe w1-gpio')
        os.system('modprobe w1-therm')
        base_dir = '/sys/bus/w1/devices/'
        device_folder = glob.glob(base_dir + '28*')
        self._count_devices = len(device_folder)
        self._devices = list()
        i = 0
        while i < self._count_devices:
            self._devices.append(device_folder[i] + '/w1_slave')
            i += 1
                
    def device_names(self):
        names = list()
        for i in range(self._count_devices):
            names.append(self._devices[i])
            temp = names[i][20:35]
            names[i] = temp
        return names

    # (one tab)
    def _read_temp(self, index):
        f = open(self._devices[index], 'r')
        lines = f.readlines()
        f.close()
        return lines

    def tempC(self, index = 0):
        lines = self._read_temp(index)
        retries = 5
        while (lines[0].strip()[-3:] != 'YES') and (retries > 0):
            lines = self._read_temp(index)
            retries -= 1
        if retries == 0:
            return 998
        equals_pos = lines[1].find('t=')
        if equals_pos != -1:
            temp = lines[1][equals_pos + 2:]
            return float(temp) / 1000
        else:
            return 999 # error
        
    def device_count(self):
        return self._count_devices
        

degree_sign = u'\xb0' # degree sign
devices = DS18B20()
count = devices.device_count()
names = devices.device_names()

def temperature(channel):
    container = devices.tempC(0)
    if container > 23:
        GPIO.output(channel,GPIO.HIGH)
        print ('Temp: {:.3f}{}C, '
            .format(container, degree_sign) +'is too high!')
    elif container < 15:
        GPIO.output(channel,GPIO.HIGH)
        print ('Temp: {:.3f}{}C, '
            .format(container, degree_sign) +'is too low!')
        time.sleep(1)
        GPIO.output(channel,GPIO.LOW)
        time.sleep(1)
        GPIO.output(channel,GPIO.HIGH)
        time.sleep(1)
        GPIO.output(channel,GPIO.LOW)
    else:
        print('Temp: {:.3f}{}C, '.format(container, degree_sign))
        GPIO.output(channel,GPIO.LOW)

def soilMoisture(moisture):
    time.sleep(1)
    if (not moisture):
        GPIO.output(23,GPIO.LOW)
        print('Moisture Detected')
    else:
        GPIO.output(23,GPIO.HIGH)
        print('You need to water your plant')

def flame(channel):
    time.sleep(1)
    if GPIO.input(channel):
        print('No flame detected')
        buzzer.off()
    else:
        print('Flame detected')
        buzzer.beep()

# flame buzzer outputs
GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)
GPIO.setup(23,GPIO.OUT)
GPIO.setup(24,GPIO.OUT)

d0_input = DigitalInputDevice(17)


GPIO.setmode(GPIO.BCM)
GPIO.setup(21,GPIO.IN)

buzzer = Buzzer(5)

while True:
    soilMoisture(d0_input.value)
    flame(21)
    temperature(24)
    print()
    time.sleep(5)
