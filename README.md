# Midterm_HCC

#### Change target
Change the detect target in`detect_UAV.py` to your specific target name. For example, target="Gemini".
Make sure this target exists in your `data/custom/classes.names`.

## Flying Algorithm
### Takeoff
     sock.sendto("takeoff".encode(encoding="utf-8"), tello_address)
     time.sleep(0.03)
     time.sleep(3)
     sock.sendto("up 50".encode(encoding="utf-8"),tello_address)
     time.sleep(1)
### Unable to detect after taking off
     if(detect==False):
          msg = "up 20"
          msg = msg.encode(encoding="utf-8")
          sent = sock.sendto(msg, tello_address)
          print('up 20')
          time.sleep(2)
          sock.sendto("forward 50".encode(encoding="utf-8"),tello_address)
          time.sleep(1)
          print("forward 30")
If it is unable for the drone to detect the target, fly up 20cm and forward 30cm.

### Detected
#### Adjust the depth
     if(classes[int(cls_pred)]==target):
          detect=True
          if(box_w>960*0.7 or box_h>720*0.7):
               finish=True
               print('big enough!Capture!')
     if(box_w>960*0.2 or box_h>720*0.2):
          ran=60
          if((x_center<width/2+ran)and(x_center>width/2ran)and(y_center>height/2-ran)and(y_center<height/2+ran)):
               if(box_w>960*0.3 or box_h>720*0.3):
                    sock.sendto("forward 20".encode(encoding="utf-8"),tello_address)
                    time.sleep(sleep_time)
                    print("forward 20")
                    big=True
          else:
               sock.sendto("forward 40".encode(encoding="utf-8"),tello_address)
               time.sleep(sleep_time)
               print("forward 40")
If the size of bounding box is more than 70% of the screen, it is ready to do screenshot.
The size of the bounding box is more than 20% of the screen, and the center tolerance offset is reduced to the original one to 60 pixel.
If the bounding box size is more than 30% of the screen and the center value is within the allowed range, fly forward 20cm.
If the center value is within the allowable range, advance 40m.

#### Adjust the target to the center of the screen
     if(x_center<width/2-ran):
          sock.sendto("left 20".encode(encoding="utf-8"), tello_address)
          time.sleep(sleep_time)
          print("left 20")
     elif(x_center>width/2+ran):
          sock.sendto("right 20".encode(encoding="utf-8"), tello_address)
          time.sleep(sleep_time)
          print("right 20")
     if(y_center<height/2-ran):
          sock.sendto("up 20".encode(encoding="utf-8"), tello_address)
          time.sleep(sleep_time)
          print("up 20")
     elif(y_center>height/2+ran):
          sock.sendto("down 20".encode(encoding="utf-8"), tello_address)
          time.sleep(sleep_time)
          print("down 20")
Bounding box center value is on the right side of the screen, fly left 20cm.
Bounding box center value is on the left side of the screen, fly right 20cm.
Bounding box center value is on the down side of the screen, fly up 20cm.
Bounding box center value is on the up side of the screen, fly down 20cm.

#### Already detected and change the detection range
     if(classes[int(cls_pred)]!=target and big):
          unseen+=1
          if(unseen>20):
               sock.sendto("back 20".encode(encoding="utf-8"), tello_address)
               time.sleep(sleep_time)
               print("back 20")
               unseen=0
               big=False
Calculate the bounding box center point, if the target has entered the acquisition range, 
but the target is not detected, set back 20cm, 
because the drone might fly too close to detect successfully.

### Capture and land
     def on_press(key):
          if(key==Key.f1):
               save_result()
          if(key==Key.f5):
               land()
          listener = keyboard.Listener(on_press=on_press)
          listener.start()
Press F1 to capture the screen and F5 to land the drone.

### Save the result
     def save_result():
          global num
          global target
          image = ImageGrab.grab()
          image.save("result/"+target+str(num)+".jpg")
          num=num+1
