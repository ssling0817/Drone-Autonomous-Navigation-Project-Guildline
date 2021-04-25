# Midterm_HCC

#### Change target
Change the detect target in`detect_UAV.py` to your specific target name. For example, target="Gemini".
Make sure this target exists in your `data/custom/classes.names`.

## Flying Algorithm
#### Takeoff
     sock.sendto("takeoff".encode(encoding="utf-8"), tello_address)
     time.sleep(0.03)
     time.sleep(3)
     sock.sendto("up 50".encode(encoding="utf-8"),tello_address)
     time.sleep(1)
#### Unable to detect after taking off
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
