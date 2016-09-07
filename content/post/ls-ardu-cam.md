+++
date = "2013-08-21T00:00:00+01:00"
draft = false
title = "Linksprite JPEG UART Camera + Arduino"
image = "ls-cam/title.png"
+++

**UPDATE 2016:
Sadly this camera died shortly after this first post, so further development was not carried out. Unfortunately, due to the price, and relative
capability compared to more modern options (a la [Raspberry Pi Camera](https://www.raspberrypi.org/products/camera-module/)) I do not intend to replace the camera.
Although it was educational while it lasted, it will never be anything more than just that, educational.**
 
Here is a small piece of code I threw together with help from the original source from Linksprite allowing me to interface with the Colour UART Camera from an Arduino board, over SoftwareSerial.
 
The reason I went for software serial is that this particular Arduino is to be used as a single node in a larger “internet-of-things” and the hardware serial port is required for communication over the XBee module that will be attached, to other nodes in the network.
 
Firstly, I’ll start with a diagram of the hardware. In this case I’m using a 3.3V Arduino Pro Mini. The voltage is critical, because the camera will need to run at 5V to be reliable. Fortunately, the Arduino is capable of regulating the voltage to it’s required level if it’s connected up right!

# Circuit

![Protoboard layout of Arduino connections for Linksprite UART camera](/images/ls-cam/prt01.png)

The circuit is extremely simple as you can see, but there is a very important point to pay attention to. The red (5V) wire is connected to RAW on the Arduino, and NOT VCC, this is because power to the raw pin will be regulated to the correct voltage (3.3V in this case), putting higher voltages straight on the VCC rail will likely damage your board.

Note this this circuit can be built with pretty much any Arduino board, but bear in mind that the code and its timings rely on a 16MHz processor, I have no idea what the results will be at different speeds, but at worst, it won’t work as expected.

# Notes

There are some assumptions made in the code, first, that your camera is already configured correctly; Image size has been set persistently to the maximum 640×480. It should work at lower resolutions, but is untested. Next, the baud on the camera should be set to 38400. This can all be configured using the Linksprite demo utility from their site (here; called “Evaluation Software”). I have attempted higher baud but received either corrupted images or slower results, if I correct this in the future, I will update this post accordingly.

# What it does

The source code will initialise the camera, take a snapshot and output the data over the hardware serial interface. The data can then be viewed in the Serial Monitor in the Arduino IDE, but will likely cause it to stop responding since it’s printing tens of kilobytes of data to a single line (new line characters would corrupt the data). The code will not loop, nor will it take multiple snapshots. One photograph is taken, and it will pause indefinitely. Hit reset on the Arduino to take another photograph.

If debugging is set to true, in the code, you will receive a bunch of debug output also, which should help when modifying the code and debugging problems. Debug output looks like this:

![Example debug serial output from camera](/images/ls-cam/ser01.jpg)

# Source Code

> NOTE: Set DEBUG to `true` to get some extra information in the Arduino Serial Terminal


``` c
#include <SoftwareSerial.h>

int j=0,k=0,count=0;
uint8_t MH,ML;
/* This must be unsigned for images larger than 7FFF, otherwise sign changes and breaks the image after that point */
uint16_t address = 0x0000;
byte incomingbyte,databyte;
boolean EndFlag=0;
int buffersize = 0x38;
unsigned long time;

boolean DEBUG = false;

union four_bytes 
{
	unsigned long sizebytes;
	unsigned char read_byte[2];
}imageSize;

SoftwareSerial camSerial(8,9);
void SendResetCommand();
void SendTakePhotoCommand();
void SendReadDataCommand();
void StopTakePhotoCommand();
void ReadJPEGFileSize();
void SetBaud56();

byte ZERO = 0x00;

void setup() 
{

	Serial.begin(115200);
	
	Serial.flush();
	
	while(!Serial)
	{
		; // wait for serial to begin
	}

	if(DEBUG)
	{
		Serial.println("************");
		Serial.println("*   INIT   *");
		Serial.println("************");
	}

	camSerial.begin(38400);
	delay(1000);

	SendResetCommand();

	byte ib[100]; 
	int iz = 0;

	while(!camSerial.available()); /* wait for data */

	delay(20);

	while(1)
	{
		if(camSerial.peek() != -1)
		{
			ib[iz] = camSerial.read();
			
			if(ib[iz-3]==0x6E && ib[iz-2]==0x64 && ib[iz-1]==0x0D && ib[iz]==0x0A)
			{
				if(DEBUG)
				{
					Serial.print("InitSize: ");Serial.println(iz);
				}
				break;
			}
			
			iz++;
		}
	}

	while(camSerial.available())
	{
		if(DEBUG)
		{
			Serial.print(camSerial.read(),HEX);
		}
		else
		{
			camSerial.read();
		}
		
	}

	//(int i=0;i<iz;i++)
	//{
	//	Serial.write(ib[i]);
	//  }

	if(DEBUG)
	{
		Serial.println("");
	}

	delay(1000);

}

void loop() 
{
	if(DEBUG)
	{
		Serial.println("Taking Photo");
	}

	delay(100);
	SendTakePictureCommand();

	while(!camSerial.available()); /* Wait for data */

	if(DEBUG)
	{
		Serial.print("Response: ");
	}
	
	while(camSerial.available())
	{
		incomingbyte = camSerial.read();
		
		if(DEBUG)
		{
			Serial.print(incomingbyte,HEX);
			Serial.print(" ");
		}
	}
	
	if(DEBUG)
	{
		Serial.println("");
	}

	ReadJPEGFileSize();

	delay(10);

	while(!camSerial.available()); /* wait for data */

	if(DEBUG)
	{
		Serial.print("Size Received: ");
	}

	for(int i=0;i<9;i++)
	{
		incomingbyte = camSerial.read();
		
		if(i>6)
		{
			imageSize.read_byte[8-i] = incomingbyte;
			
			if(DEBUG)
			{
				Serial.print(incomingbyte,HEX);
			}
		}

	}

	if(DEBUG)
	{
		Serial.print("(");Serial.print(imageSize.sizebytes/1000);Serial.println("kb)");
	}

	byte a[buffersize];

	if(DEBUG)
	{
		Serial.println("Read Data");
	}
	
	delay(1000);

	time = millis();
	
	while(camSerial.available())
	{
		camSerial.read();
	}

	while(!EndFlag)
	{
		j=0;
		k=0;
		int i=0;
		count=0;
		
		SendReadDataCommand();
		//delay(1);
		while(!camSerial.available()); /* Wait for data */
		
		// Bin the header
		for(i=0;i<5;i++)
		{
			incomingbyte=camSerial.read();
		}
		
		delayMicroseconds(300);
		
		// Read real data
		for(k=0;((k<buffersize)&&(!EndFlag));k++)
		{
			databyte=camSerial.read();
			a[k]=databyte;
			
			if((a[k-1]==0xFF) && (a[k]==0xD9))
			{
				EndFlag=1;
			}
			
			count++;
		}
		
		delayMicroseconds(300);
		
		// Bin the footer
		while(camSerial.available())
		{
			incomingbyte=camSerial.read();
		}
		
		if(DEBUG)
		{
			//Serial.println("");
		}
		
		for(j=0;j<count;j++)
		{
			if(a[j]<0x10 && DEBUG)
			{
				Serial.print("0");
				Serial.print(a[j], HEX);
			}
			else
			{
				Serial.write(a[j]);
			}
		}
		
		delay(20);
		
	}

	delay(10);

	if(DEBUG)
	{
		Serial.println("");  
		Serial.println("Image Received");
	}
	
	time = (millis()-time);
	if(DEBUG)
	{
		Serial.print("Time: ");Serial.print(time/1000);Serial.println(" seconds");
	}
	StopTakePhotoCommand();
	while(1);
}

void SendResetCommand()
{
	camSerial.write(0x56);
	camSerial.write(ZERO);
	camSerial.write(0x26);
	camSerial.write(ZERO);
}

void SendTakePictureCommand()
{
	camSerial.write(0x56);
	camSerial.write(ZERO);
	camSerial.write(0x36);
	camSerial.write(0x01);
	camSerial.write(ZERO);
}

void SendReadDataCommand()
{
	MH=address/0x100;
	ML=address%0x100;
	//Serial.print("Address int: ");Serial.println(address,HEX);
	//Serial.print("Reading Address: ");Serial.print(MH,HEX);Serial.println(ML,HEX);
	camSerial.write(0x56);
	camSerial.write(ZERO);
	camSerial.write(0x32);
	camSerial.write(0x0C);
	camSerial.write(ZERO);
	camSerial.write(0x0A);
	camSerial.write(ZERO);
	camSerial.write(ZERO);
	camSerial.write(MH);
	camSerial.write(ML);
	camSerial.write(ZERO);
	camSerial.write(ZERO);
	camSerial.write(ZERO);
	camSerial.write(buffersize);
	camSerial.write(ZERO);
	camSerial.write(0x01);
	address+=(buffersize);
}

void ReadJPEGFileSize()
{
	camSerial.write(0x56);
	camSerial.write(ZERO);
	camSerial.write(0x34);
	camSerial.write(0x01);
	camSerial.write(ZERO);
}

void StopTakePhotoCommand()
{
	camSerial.write(0x56);
	camSerial.write(0x56);
	camSerial.write(0x36);
	camSerial.write(0x01);
	camSerial.write(0x03);

	while(!camSerial.available());

	while(camSerial.available())
	{
		camSerial.read();
		//Serial.print(camSerial.read(),HEX);Serial.print(" ");
	}

	//Serial.println("");
}
```