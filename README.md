# Midterm_HCC

#### Change target
Change the detect target in`detect_UAV.py` to your specific target name. For example, target="Gemini".
Make sure this target exists in your `data/custom/classes.names`.

## Flying Algorithm
#### Takeoff
    $ sock.sendto("takeoff".encode(encoding="utf-8"), tello_address)
    $ time.sleep(0.03)
    $ time.sleep(3)
    $ sock.sendto("up 50".encode(encoding="utf-8"),tello_address)
    $ time.sleep(1)

#### Change the detection range
Calculate the bounding box center point, if the target has entered the acquisition range, but the target is not detected, set back 20cm.
    $ x_center=(x1+x2)/2
    $ y_center=(y1+y2)/2
    $ if(classes[int(cls_pred)]!=target and big):
unseen+=1
if(unseen>20):
sock.sendto("back 20".encode(encoding="utf-8"), tello_address)
time.sleep(sleep_time)
print("back 20")
unseen=0
big=False
