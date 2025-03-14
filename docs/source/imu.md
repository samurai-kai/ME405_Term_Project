# IMU Driver    
    BNO055 IMU driver 
    by Kai De La Cruz & Kenzie Goldman 2/20/25

    import time from pyb import I2C import struct

    class BNO055: 
        # Fusion mode values 
        IMU_MODE = 0x08 
        COMPASS_MODE = 0x09
        M4G_MODE = 0x0A 
        NDOF_MODE = 0x0C 
        NDOF_FMC_OFF_MODE = 0x0B 
        imu_address = 0x28 
        config_mode = 0x00 

        def __init__(self, i2c, address=imu_address, mode=config_mode): 
            self.i2c = i2c 
            self.address = address 

        def set_mode(self, mode): 
            """Sets the operation mode of the BNO055."""
            self.i2c.mem_write(mode, self.address, 0x3D) 
            time.sleep(0.02) # Delay to ensure the mode change is processed

        def get_calibration_status(self): 
            """A method to retrieve the calibration status byte from the BNO055. 
            Ref page 47 of the datasheet for address.""" 
            data = self.i2c.mem_read(1, self.address, 0x35) 
            return data[0]

        def get_calibration_coefficients(self): 
            """A method to retrieve the calibration coefficients from the BNO055. 
            Ref page 47 of the datasheet for address.""" 
            data = self.i2c.mem_read(22, self.address, 0x55)
            calibration_data = list(data) 
            print(f"Calibration data: {calibration_data}") 
            return calibration_data 

        def get_euler_angles(self):
            """A method to retrieve the Euler angles from the BNO055. 
            Ref in-class notes for the register addresses."""
            buf = bytearray(6) 
            data = self.i2c.mem_read(6, self.address, 0x1A) 
            heading, roll, pitch = struct.unpack('\<hhh', data)

            return heading/16.0, roll/16.0, pitch/16.0

        def get_ang_velocity(self): 
            """A method to retrieve the angular velocity
            from the BNO055. Ref in-class notes for the register addresses.""" 
            data = self.i2c.mem_read(2, self.address, 0x14) 
            roll_rate = (data[1] << 8 | data[0]) / 900.0 # x-dir 
            pitch_rate = (data[3] << 8 | data[2]) / 900.0 # y-dir 
            yaw_rate = (data[5] << 8 | data[4]) / 900.0 # z-dir 
            return roll_rate, pitch_rate, yaw_rate

        def read_calibration_data(self, file_path): 
            """Read calibration data from a text file.""" 
            calibration_data = [] 
            try: 
                with open(file_path, 'r') as file: 
                    for line in file:
                        calibration_data.append(int(line.strip())) 
            except Exception as e:
                print(f"Error reading calibration data: {e}") 
            return calibration_data

        def write_calibration_data_to_imu(self, calibration_data): 
            """Write calibration data to the BNO055 IMU.""" 
            if len(calibration_data) != 22:
                raise ValueError("Invalid calibration data length")

            # Write accelerometer offsets
            self.write_register(0x55, calibration_data[0])
            self.write_register(0x56, calibration_data[1])
            self.write_register(0x57, calibration_data[2])
            self.write_register(0x58, calibration_data[3])
            self.write_register(0x59, calibration_data[4])
            self.write_register(0x5A, calibration_data[5])

            # Write magnetometer offsets
            self.write_register(0x5B, calibration_data[6])
            self.write_register(0x5C, calibration_data[7])
            self.write_register(0x5D, calibration_data[8])
            self.write_register(0x5E, calibration_data[9])
            self.write_register(0x5F, calibration_data[10])
            self.write_register(0x60, calibration_data[11])

            # Write gyroscope offsets
            self.write_register(0x61, calibration_data[12])
            self.write_register(0x62, calibration_data[13])
            self.write_register(0x63, calibration_data[14])
            self.write_register(0x64, calibration_data[15])
            self.write_register(0x65, calibration_data[16])
            self.write_register(0x66, calibration_data[17])

            # Write accelerometer radius
            self.write_register(0x67, calibration_data[18])
            self.write_register(0x68, calibration_data[19])

            # Write magnetometer radius
            self.write_register(0x69, calibration_data[20])
            self.write_register(0x6A, calibration_data[21])

        def write_register(self, register, value): 
            """Write a value to a specific register of the BNO055.""" 
            self.i2c.mem_write(value,self.address, register)

        def write_calibration_data(self, file_path, calibration_data): 
            """Write calibration data to a text file.""" 
            with open(file_path, 'w') as file:
                for value in calibration_data: 
                    file.write(f"{value}\n")
            print(f"[IMU] Calibration data written to {file_path}.")

        def load_calibration_data(self, file_path): 
            """Load calibration data from a text file.""" 
            with open(file_path, 'r') as file: 
                calibration_data = self.read_calibration_data(file_path) 
                if len(calibration_data) == 22:
                    return calibration_data 
                else: 
                    raise ValueError("Invalid calibration data length")

        def reset(self): 
            """Resets the BNO055 IMU""" 
            self.i2c.mem_write(0x20, self.address, 0x3F) # Reset command 
            time.sleep(0.7) # Wait for reset
            print("[IMU] Reset complete.")

        def scan(self):  
            """Scan the I2C bus for connected devices and return their 
            addresses.""" 
            devices = self.i2c.scan() 
            if devices: 
                print("I2C device at addresse:", [hex(device) for device in devices]) 
            else:
                print("No I2C devices found.")
