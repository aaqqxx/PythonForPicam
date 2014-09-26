# coding:utf-8
#!/usr/bin/env python

__author__ = 'XingHua'

from PiTypes import *
from PiTypesMore import *
from PiParameterLookup import *
from PiFunctions import *
from PIL import Image
import numpy as np
import struct
import matplotlib.pyplot as plt

import time

import logging
import sys

logging.basicConfig(level=logging.DEBUG,
                    format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
                    datefmt='%a, %d %b %Y %H:%M:%S',
                    filename='myapp.log',
                    filemode='w')

console = logging.StreamHandler()
console.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s')
console.setFormatter(formatter)
logging.getLogger('').addHandler(console)

"""

"""


def pointer(x):
    """Returns a ctypes pointer"""
    ptr = ctypes.pointer(x)
    return ptr


def load(x):
    """Loads DLL library where argument is location of library"""
    x = ctypes.cdll.LoadLibrary(x)
    return x


class PICamera_Params_STRUCT():
    def __init__(self):
        self.temperature_setpoint = piflt()
        self.exposure_time = piflt()
        self.readout_control_mode = PicamEnumeratedType()
        self.adc_quality = PicamEnumeratedType()
        # piflt adc_speed=
        self.adc_speed = PicamEnumeratedType()
        self.adc_analog_gain = PicamEnumeratedType()
        self.adc_em_gain = piint()
        self.trigger_response = PicamEnumeratedType()
        self.shutter_mode = PicamEnumeratedType()


class PICamera():
    def __init__(self):
        self.inited = pibln()
        self.camera = PicamHandle()
        self.id = PicamCameraID()
        self.serial_number = ctypes.c_char_p('Demo Cam 1')
        self.data = PicamAvailableData()
        self.errors = PicamAcquisitionErrorsMask()
        self.readoutstride = piint(0)
        self.init()
        self.cnt = 0
        pass

    def init(self):
        print "error =", Picam_IsLibraryInitialized(pointer(self.inited))
        if not self.inited == ctypes.c_bool(0).value:
            print "before Picam_InitializeLibrary()......"
            self.errors = Picam_InitializeLibrary()
            print "PicamError error =", self.errors

        # logging.error( Picam_OpenFirstCamera(pointer(self.camera)))
        # logging.error(PicamAcquisitionErrorsMask(0))

        # print Picam_OpenFirstCamera(pointer(self.camera))

        if Picam_OpenFirstCamera(pointer(self.camera)) == "PicamError_None":
            Picam_GetCameraID(pointer(self.camera), pointer(self.id))
            res = 0
            print "Open First Camera successfully!"
            # print self.id.serial_number
        else:
            print 'Preparing to connect Demo Camera'
            model = ctypes.c_int(10)
            # serial_number = ctypes.c_char_p('Demo Cam 1')
            print 'Demo camera connetcted with return value = ', Picam_ConnectDemoCamera(model, self.serial_number,
                                                                                         pointer(self.id))
            print '\n'

            print 'Camera model is ', self.id.model
            print 'Camera computer interface is ', self.id.computer_interface
            print 'Camera sensor_name is ', self.id.sensor_name
            print 'Camera serial number is', self.id.serial_number
            print '\n'
            # self.camera = PicamHandle()
            print 'Opening First Camera', Picam_OpenFirstCamera(ctypes.addressof(self.camera))

            res = 0
        return res, None
        pass

    def acquire(self):
        readoutstride = piint(0)
        readout_count = pi64s(1)
        readout_time_out = piint(-1)
        available = PicamAvailableData()
        errors = PicamAcquisitionErrorsMask()

        Picam_GetParameterIntegerValue(self.camera, ctypes.c_int(PicamParameter_ReadoutStride),
                                       ctypes.byref(readoutstride))

        Picam_Acquire.argtypes = PicamHandle, pi64s, piint, ctypes.POINTER(PicamAvailableData), ctypes.POINTER(
            PicamAcquisitionErrorsMask)
        Picam_Acquire.restype = piint

        res = Picam_Acquire(self.camera, readout_count, readout_time_out, ctypes.byref(available),
                            ctypes.byref(errors))
        print "Picam_Acquire res is", res

        if res == "PicamError_None":
            # print available.initial_readout
            DataArrayPointerType = ctypes.POINTER(pi16u * 1048576)
            print "available.initial_readout: ", available.initial_readout
            tmp = ctypes.cast(available.initial_readout, DataArrayPointerType)

            tmp1 = np.array(tmp.contents)
            tmp1 = tmp1.reshape(1024, 1024)
            tmp1 = np.asarray(tmp1, dtype=int)
            # tmp1=np.arange(0,1024*1024,1).reshape(1024,1024)
            im = Image.fromarray(tmp1)
            im.show()

            im.save(r".\test111%s.png" % str(self.cnt))
            self.cnt = self.cnt + 1

            print "Initial readout type is", type(available.initial_readout)
            return available
        else:
            print "Error: Camera only collected ", available.readout_count
            return None
        pass

    def acquire_to_save(self):
        available = self.acquire()
        """ Test Routine to Access Data """

        """ Create an array type to hold 1024x1024 16bit integers """
        DataArrayType = pi16u * 1048576

        """ Create pointer type for the above array type """
        DataArrayPointerType = ctypes.POINTER(pi16u * 1048576)

        """ Create an instance of the pointer type, and point it to initial readout contents (memory address?) """
        DataPointer = ctypes.cast(available.initial_readout, DataArrayPointerType)

        """ Create a separate array with readout contents """
        data = DataPointer.contents

        """ Write contents of Data to binary file"""
        libc = ctypes.cdll.msvcrt
        fopen = libc.fopen
        fopen.argtypes = ctypes.c_char_p, ctypes.c_char_p
        fopen.restype = ctypes.c_void_p

        fwrite = libc.fwrite
        fwrite.argtypes = ctypes.c_void_p, ctypes.c_size_t, ctypes.c_size_t, ctypes.c_void_p
        fwrite.restype = ctypes.c_size_t

        fclose = libc.fclose
        fclose.argtypes = ctypes.c_void_p,
        fclose.restype = ctypes.c_int

        fp = fopen('PythonBinOutput.raw', 'wb')
        readoutstride = piint(0)
        print "Getting readout stride. ", Picam_GetParameterIntegerValue(self.camera,
                                                                         ctypes.c_int(PicamParameter_ReadoutStride),
                                                                         ctypes.byref(readoutstride))
        print "readoutstride.value is", readoutstride.value
        print 'fwrite returns: ', fwrite(data, readoutstride.value, 1, fp)

        fclose(fp)
        res = 0
        return res, None
        pass

    def get_camera_model(self):
        string = ctypes.c_char_p("              ")
        Picam_GetEnumerationString(PicamEnumeratedType(2), self.id.model, ctypes.byref(string))
        res = string.value + self.id.serial_number + self.id.sensor_name
        print res
        return res
        # pass

    def set_temperature(self, temp):
        temp = struct.unpack("!i", temp)[0]
        temp = piflt(temp)
        Picam_SetParameterFloatingPointValue(self.camera, ctypes.c_float(PicamParameter_SensorTemperatureSetPoint),
                                             ctypes.byref(temp))
        self.commit_common_parameters()
        return 0, None
        pass

    def read_temperature(self):
        temperature = piflt(0)
        error = Picam_ReadParameterFloatingPointValue(
            self.camera,
            PicamParameter_SensorTemperatureReading,
            pointer(temperature))
        print error, "temperature is ", temperature
        return 0, struct("!d", temperature.value)


    def set_exposure_time(self, exposure_time):
        # exposure_time = struct.unpack("!f", exposure_time)[0]
        print exposure_time
        exposure_time1 = piflt(exposure_time)
        print type(exposure_time1), exposure_time1

        # """ PICAM_API Picam_GetParameterFloatingPointValue( PicamHandle camera, PicamParameter parameter, piflt* value) """


        print "PicamParameter_ExposureTime is ", PicamParameter_ExposureTime

        print "set exposure time is ", Picam_SetParameterFloatingPointValue(self.camera,
                                                                            ctypes.c_int(PicamParameter_ExposureTime),
                                                                            exposure_time1)
        # PicamParameter
        # PicamParameter
        print exposure_time1.value
        # readoutstride = piint(0)
        # print "Getting readout stride. ", Picam_GetParameterIntegerValue(self.camera,
        # ctypes.c_int(PicamParameter_ReadoutStride),
        # ctypes.byref(readoutstride))
        # print readoutstride.value
        self.commit_common_parameters()
        return 0, None


    def set_readout_mode(self, readout_mode):
        error = Picam_SetParameterIntegerValue(
            self.camera,
            PicamParameter_AdcQuality,
            ctypes.c_int(readout_mode));
        logging.info("readout mode set successfully...")
        return 0, None

    def set_adc_quality(self, quality):
        error = Picam_SetParameterIntegerValue(
            self.camera,
            PicamParameter_AdcQuality,
            ctypes.c_int(quality));
        logging.info("AdcQuality set successfully...")
        return 0, None

    def set_adc_speed(self, speed):
        logging.info("adc speed is %s" % speed)
        error = Picam_SetParameterFloatingPointValue(
            self.camera,
            PicamParameter_AdcSpeed,
            ctypes.c_int(speed));
        logging.info("Adc speed set successfully..%s" % error)
        return 0, None


    def set_adc_analog_gain(self, analog_gain):
        error = Picam_SetParameterIntegerValue(
            self.camera,
            PicamParameter_AdcAnalogGain,
            ctypes.c_int(analog_gain))
        logging.info("AdcSpeed set successfully..%s" % error)
        return 0, None


    def set_adc_em_gain(self, em_gain):
        error = Picam_SetParameterIntegerValue(
            self.camera,
            PicamParameter_AdcEMGain,
            ctypes.c_int(em_gain))
        logging.info("AdcEMGain set successfully..%s" % error)
        return 0, None


    def set_trigger_mode(self, trigger_mode):
        error = Picam_SetParameterIntegerValue(
            self.camera,
            PicamParameter_TriggerResponse,
            ctypes.c_int(trigger_mode))
        logging.info("TriggerResponse set successfully..%s" % error)
        return 0, None


    def set_shutter_mode(self, shutter_mode):
        error = Picam_SetParameterIntegerValue(
            self.camera,
            PicamParameter_ShutterTimingMode,
            ctypes.c_int(shutter_mode))
        logging.info("shutter mode set successfully..%s" % error)
        return 0, None


    def get_exposure_time(self):
        # exposure_time = piflt()
        # print "get exposure time is ", Picam_SetParameterFloatingPointValue(self.camera,
        # ctypes.c_int(PicamParameter_ExposureTime),
        # exposure_time)
        # print "get_exposure_time is",exposure_time
        tmp = piflt(1)
        print "get exposure time", Picam_GetParameterFloatingPointValue(self.camera,
                                                                        ctypes.c_int(PicamParameter_ExposureTime),
                                                                        pointer(tmp))
        print "exposure time = ", tmp.value
        return 0, struct.pack('!d', tmp.value)


    def commit_common_parameters(self):
        # pibln* committed
        committed = pibln()
        Picam_AreParametersCommitted(self.camera, pointer(committed))
        if committed.value == True:
            print "Parameters have not changed"
        else:
            print "Parameters have been modified"

        failed_parameters_count = piint(1)
        # tmp = ctypes.c_void_p()

        print "@@@@@@@@@", self.get_exposure_time()

        # tmp = piint()
        # failed_parameters = (ctypes.c_int * 1)(PicamParameter_ExposureTime)
        failed_parameters = ctypes.POINTER(ctypes.c_int)()
        # print failed_parameters

        # failed_parameters = ctypes.POINTER(ctypes.POINTER(ctypes.c_int))()

        # print  (ctypes.c_int)*2

        # failed_parameters = ctypes.c_int()
        print failed_parameters
        # failed_parameters = pointer(tmp)
        # Picam_Acquire.argtypes = PicamHandle, pi64s, piint, ctypes.POINTER(PicamAvailableData), ctypes.POINTER(
        # PicamAcquisitionErrorsMask)
        # Picam_Acquire.restype = piint

        # Picam_CommitParameters.argtypes = PicamHandle, ctypes.POINTER(ctypes.POINTER(ctypes.c_int)), ctypes.POINTER(
        # piint)
        # Picam_CommitParameters.restype = piint

        print "commit paramter... ", Picam_CommitParameters(self.camera, ctypes.pointer(failed_parameters),
                                                            ctypes.byref(failed_parameters_count))

        # print "commit paramter... ", Picam_CommitParameters(self.camera, failed_parameters,
        # ctypes.byref(failed_parameters_count))

        print "failed_parameters_count is ", failed_parameters_count
        return 0, None


    def read_common_parameters(self):
        PICamera_info_str = ""
        fval = piflt()
        ival = piint()
        error = Picam_ReadParameterFloatingPointValue(
            self.camera,
            PicamParameter_SensorTemperatureReading, pointer(fval))
        logging.info(error)
        if error == "PicamError_None":
            PICamera_info_str += "Temperature is " + str(fval.value) + " degrees C\n"

        error = Picam_GetParameterFloatingPointValue(
            self.camera,
            PicamParameter_ExposureTime, pointer(fval))
        logging.info("get exposure time status " + str(error))
        if error == "PicamError_None":
            PICamera_info_str += "Exposure time is " + str(fval.value) + " ms\n"

        error = Picam_GetParameterIntegerValue(self.camera, PicamParameter_ReadoutControlMode, pointer(ival))
        logging.info("get readout mode status " + str(error))
        if error == "PicamError_None":
            PICamera_info_str += "readout mode is " + str(ival.value) + "\n";

        error = Picam_GetParameterIntegerValue(self.camera, PicamParameter_AdcQuality, pointer(ival))
        logging.info("get ADC quality status " + str(error))
        if error == "PicamError_None":
            PICamera_info_str += "ADC quality is " + str(ival.value) + "\n"

        error = Picam_GetParameterFloatingPointValue(self.camera, PicamParameter_AdcSpeed, pointer(fval))
        logging.info("get ADC speed status " + str(error))
        if error == "PicamError_None":
            PICamera_info_str += "ADC speed is " + str(fval.value) + "\n"

        error = Picam_GetParameterIntegerValue(self.camera, PicamParameter_AdcAnalogGain, pointer(ival))
        logging.info("get ADC analog gain status " + str(error))
        if error == "PicamError_None":
            PICamera_info_str += "ADC analog gain is " + str(ival.value) + "\n"

        error = Picam_GetParameterIntegerValue(self.camera, PicamParameter_AdcEMGain, pointer(ival))
        logging.info("get ADC EM Gain status " + str(error))
        if error == "PicamError_None":
            PICamera_info_str += "ADC EM Gain is " + str(ival.value) + "\n"

        error = Picam_GetParameterIntegerValue(self.camera, PicamParameter_TriggerResponse, pointer(ival))
        logging.info("get trigger mode status " + str(error))
        if error == "PicamError_None":
            PICamera_info_str += "trigger mode is " + str(ival.value) + "\n"

        error = Picam_GetParameterIntegerValue(self.camera, PicamParameter_ShutterTimingMode, pointer(ival))
        logging.info("get shutter mode status " + str(error))
        if error == "PicamError_None":
            PICamera_info_str += "shutter mode is " + str(ival.value) + "\n"

        logging.info(":\n" + PICamera_info_str)
        return 0, struct.pack("%ds" % len(PICamera_info_str), PICamera_info_str)


    def open_shutter(self):
        """
        typedef enum PicamShutterTimingMode
        {
            PicamShutterTimingMode_Normal            = 1,
            PicamShutterTimingMode_AlwaysClosed      = 2,
            PicamShutterTimingMode_AlwaysOpen        = 3,
            PicamShutterTimingMode_OpenBeforeTrigger = 4
        } PicamShutterTimingMode; /* (5) */
        :return:
        """
        error = Picam_SetParameterIntegerValue(
            self.camera,
            PicamParameter_ShutterTimingMode,
            ctypes.c_int(3))
        logging.info("shutter open successfully..%s", error)
        return 0, None


    def close_shutter(self):
        error = Picam_SetParameterIntegerValue(
            self.camera,
            PicamParameter_ShutterTimingMode,
            ctypes.c_int(2))
        logging.info("shutter close successfully..%s", error)
        return 0, None



if __name__ == "__main__":
    PIcamer.get_camera_model()
    PIcamer.read_temperature()
    PIcamer.get_exposure_time()
    PIcamer.acquire()
    
    PIcamer.set_exposure_time(0.5)
    
    PIcamer.acquire()
    PIcamer.get_exposure_time()
    PIcamer.read_common_parameters()
