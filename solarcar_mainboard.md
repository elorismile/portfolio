# SolarCar Main Control Board
![53866110773_d6358c1af1_o](https://github.com/user-attachments/assets/daf715f5-a0cd-4f39-ab20-6801103e74a4)

![Screenshot From 2025-01-09 11-20-29](https://github.com/user-attachments/assets/b9b9d0e8-0fdc-4766-9e34-8349d86b49d2)

# Version 0
![20241018_163621](https://github.com/user-attachments/assets/ff940f88-3c56-4809-9c61-33637766f3bb)
Version 0 was created by the previous team, and featured an arduino to control the ignition sequence and a Raspberry Pi that ran the user interface. However, it was very unreliable and had the following problems:
- Boot up takes >2 minutes, and will sometimes get stuck
- Needs wifi access, which is not available on the road
- No real time clock without wifi
- No datalogging capability
- Very power hungry (around 20 watts)
  
To fix these issues, I began development on a replacement that uses an ESP32-S3 to replace both the Raspberry Pi and the Arduino. 

# Version 1
![20241020_225354](https://github.com/user-attachments/assets/9ee0763b-678e-4f72-b8f8-f5a410323928)

Version 1 was created on a very tight time schedule, since we had track testing scheduled and needed a functional main board with datalogging. I ordered prototyping parts from Adafruit, and within 2 weeks I had a functional prototype ready to go just in time for testing. 

Version 1 presented the following isses which were addressed with version 2:
- SD card holder didn't maintain contact, it required a zip tie to keep it pressed into the socket
- GPS only updated at 1Hz
- Poor GPS reception

# Version 2
![Screenshot From 2024-12-08 04-06-20](https://github.com/user-attachments/assets/8c0c13a2-734a-4312-9b21-6c95756b9b5a)

# Version 3
Version 3 is mostly electrically the same as version 2, but repositions stuff to make it easier to use. 
![Screenshot_From_2025-01-22_13-57-49](https://github.com/user-attachments/assets/e54388ce-cccd-4f8d-88f4-c1bf09cbb582)


[return to main page](main.md)
