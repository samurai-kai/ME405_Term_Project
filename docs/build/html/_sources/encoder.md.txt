# Encoder Driver

    Encoder driver 
    by Kai De La Cruz & Kenzie Goldman 1/30/25

    from time import ticks_ms, ticks_diff, ticks_add 
    from pyb import Pin, Timer 
    import math

    class Encoder: 
        '''A quadrature encoder decoding interface encapsulated in a python 
        class'''

        def __init__(self, tim_num, chA_pin, chB_pin):
            '''Initializes an Encoder object'''
            self.prev_time = ticks_ms() #time of the most recent update
            self.position = 0.0 #total accumulated position of the encoder
            self.prev_count = 0.0 #counter value from the most recent update
            self.delta = 0.0 #change in count between last two updates
            self.velocity_buffer = []
            self.window_size = 5  # Number of samples to average over
            # copy params to attrbutes
            self.chA_pin = Pin(chA_pin)
            self.chB_pin = Pin(chB_pin)

            # Configure timer channels for encoder interface
            self.tim = Timer(tim_num, prescaler=0, period=0xFFFF)
            self.encd_CHA = self.tim.channel(1, pin=self.chA_pin, mode=Timer.ENC_AB)
            self.encd_CHB = self.tim.channel(2, pin=self.chB_pin, mode=Timer.ENC_AB)

        def update(self):
            '''Runs one update step on the encoder's timer counter to keep track of 
            the change in count and check for counter reload'''
            current_time = ticks_ms()
            self.dt = ticks_diff(current_time, self.prev_time)/1000
            self.prev_time = current_time

            current_count = self.tim.counter()
            self.delta = current_count - self.prev_count
            self.prev_count = current_count

            # Handle overflow/underflow
            if self.delta >= 32768:  # AR+1/2 
                self.delta -= 65536
            elif self.delta <= -32768:
                self.delta += 65536

            self.position += self.delta

        def get_position(self):
            '''Returns the most recently updated value of position as determined 
            within the update() method'''
            self.theta = (self.position/1440*2*math.pi)
            return float(self.theta)
            
        def get_velocity(self):
            '''Returns a smoothed measure of velocity using a moving average of 
            the most recent updates'''
            velocity = (self.delta / 1440) * (2 * math.pi) / self.dt 
            self.velocity_buffer.append(velocity)

            # Keep only the last `window_size` samples
            if len(self.velocity_buffer) > self.window_size:
                self.velocity_buffer.pop(0)

            # Return the average velocity
            return sum(self.velocity_buffer) / len(self.velocity_buffer)

        def zero(self):
            '''Sets the present encoder position to zero and causes future cupdates 
            to measure with respect to the new zero position'''
            self.position = 0
            self.delta = 0
            self.prev_count = 0
