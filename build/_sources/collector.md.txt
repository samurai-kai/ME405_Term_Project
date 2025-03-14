# Collector Class
    Collector class 
    by Kai De La Cruz & Kenzie Goldman 1/30/25

    from array import array 
    from pyb import Timer

    class Collector: 
        '''A data collector class that interfaces with an ADC
        to collect data''' 
        
        def __init__(self): 
        '''Initializes a Collector object''' 
            self.a_size = 50 
            self.time = array('f', [0]*self.a_size)
            self.pos = array('f', [0]*self.a_size) 
            self.vel = array('f', [0\]*self.a_size) 
            self.idx = 0 self.full = False  # timer setup
            self.tim7 = Timer(7, freq=10000)  # 10 kHz timer

        def collect(self, pos_value, vel_value):
            '''Collects position data in an array'''
            if self.idx < self.a_size:
                self.pos[self.idx] = pos_value
                self.vel[self.idx] = vel_value
                self.time[self.idx] = self.idx * 0.01  
                self.idx += 1
            else:
                self.full = True

        def is_full(self):
            '''Returns True if the collector is full'''
            return self.full

        def reset(self):
            '''Resets the collector to collect new data'''
            self.idx = 0
            self.full = False

        def return_data(self):
            '''Returns the collected data as columns for position and velocity'''
            return self.time[:self.idx], self.pos[:self.idx], self.vel[:self.idx]
