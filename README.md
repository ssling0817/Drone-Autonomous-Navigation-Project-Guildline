# HCC Midterm Project

### Installation
##### Clone and install requirements
    $ git clone https://github.com/ssling0817/UAV_Object_Detection
    $ cd PyTorch-YOLOv3/
    $ sudo pip3 install -r requirements.txt
Visit https://github.com/ssling0817/UAV_Object_Detection for detail operation.

## Data augmentation
#### Brightness conversion
    def brightness(img, value):
        hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
        hsv = np.array(hsv, dtype = np.float64)
        hsv[:,:,1] = hsv[:,:,1]*value
        hsv[:,:,1][hsv[:,:,1]>255]  = 255
        hsv[:,:,2] = hsv[:,:,2]*value 
        hsv[:,:,2][hsv[:,:,2]>255]  = 255
        hsv = np.array(hsv, dtype = np.uint8)
        img = cv2.cvtColor(hsv, cv2.COLOR_HSV2BGR)
        return img
##### Brightness conversion with label
    num_brightness_aug = 4
    for imgpath in tqdm(glob.glob(target_Image_Folder + '*')):
        img = cv2.imread(imgpath)
        name = imgpath.split('/', 4)[3].split('.')[0]
        valist = []
        for i in range(num_brightness_aug):
            if i % 2 == 0:
                low, high = 0.4, 0.8
            else:
                low, high = 1.2, 1.6
            value = random.uniform(low, high)
            valist.append(value)
        for value in valist:
            img_r = brightness(img, value)
            cv2.imwrite(target_Image_Folder + name + '_brightness' + str(value)[:5] + '.jpg', img_r)
            path = target_Label_Folder + name + '.txt'
            infile = open(path, 'r')
            for line in infile:
                outfile = open(target_Label_Folder + name + '_brightness' + str(value)[:5] + '.txt', 'w')
                outfile.write(line)
                outfile.close()
            infile.close()


#### Horizontal and vertical flip
    def horizontal_flip(img, flag):
        if flag:
            return cv2.flip(img, 1)
        else:
            return img
    def vertical_flip(img, flag):
        if flag:
            return cv2.flip(img, 0)
        else:
            return img
##### Flip with label
    for imgpath in tqdm(glob.glob(target_Image_Folder + '*')):
        img = cv2.imread(imgpath)
        name = path.split('/', 4)[3].split('.')[0]
        img_r = horizontal_flip(img, True)
        cv2.imwrite(target_Image_Folder + name + '_horizontal' + '.jpg', img_r)
        img_r = vertical_flip(img, True)
        cv2.imwrite(target_Image_Folder + name + '_vertical' + '.jpg', img_r)
        path = target_Label_Folder + name + '.txt'
        infile = open(path, 'r')
        for line in infile:
            data = line.split(' ')
            x = float(data[1]) - 0.5
            x = x * (-1)
            x = x + 0.5
            outfile = open(target_Label_Folder + name + '_horizontal' + '.txt', 'w')
            data[1] = str(x)
            outfile.write(' '.join(data))
            outfile.close()
            data = line.split(' ')
            y = float(data[2]) - 0.5
            y = y * (-1)
            y = y + 0.5
            outfile = open(target_Label_Folder + name + '_vertical' + '.txt', 'w')
            data[2] = str(y)[:8]
            outfile.write(' '.join(data))
            outfile.close()
        infile.close()



#### Rotation and Rotation Matrix Functions
    def rotation(img, angle):
        h, w = img.shape[:2]
        M = cv2.getRotationMatrix2D((int(w/2), int(h/2)), angle, 1)
        img = cv2.warpAffine(img, M, (w, h))
        return img

    def rotation_matrix(x, y, angle):
        rad = math.radians(angle)
        rx = x * math.cos(rad) - y * math.sin(rad)
        ry = x * math.sin(rad) + y * math.cos(rad)
        return rx, ry
##### Rotate with label
    num_angle_aug = 3
    for imgpath in tqdm(glob.glob(source_Image_Folder + '*')):
        name = imgpath.split('/', 4)[3].split('.')[0]
        anglist = []
        for i in range(num_angle_aug):
            angle = 10
            angle = int(random.uniform(-angle, angle))
            while angle == 0 or angle in anglist:
                angle = 10
                angle = int(random.uniform(-angle, angle))
            anglist.append(angle)
        anglist.append(0)
        for angle in anglist:
            flag = True
            path = source_Label_Folder + name + '.txt'
            infile = open(path, 'r')
            for line in infile:
                data = line.split(' ')
                x, y = float(data[1]) - 0.5, float(data[2]) - 0.5
                x, y = rotation_matrix(x, y, angle * (-1))
                x, y = x + 0.5, y + 0.5
                if x > 1 or x < 0 or y > 1 or y < 0:
                    flag = False
                    break
                outfile = open(target_Label_Folder + name + '_rotation' + str(angle) + '.txt', 'w')
                data[1], data[2] = str(x)[:8], str(y)[:8]
                outfile.write(' '.join(data))
                outfile.close()
            infile.close()
            if flag:
                img = cv2.imread(imgpath)
                img_r = rotation(img, angle)
                cv2.imwrite(target_Image_Folder + name + '_rotation' + str(angle) + '.jpg', img_r)


#### Select source and target folders
    star = 'Sagittarius'
    source_Image_Folder = 'org/Image/' + star + '/'
    source_Label_Folder = 'org/Label/' + star + '/'
    target_Image_Folder = 'new/Image/' + star + '/'
    target_Label_Folder = 'new/Label/' + star + '/'
Take `Sagittarius` as an example, it is going to select from its `org/Image` and `org/Label` folders, and save in `new/Image` and `new/Label` folders.

####









## Flying Algorithm
### Takeoff
     sock.sendto("takeoff".encode(encoding="utf-8"), tello_address)
     time.sleep(0.03)
     time.sleep(3)
     sock.sendto("up 50".encode(encoding="utf-8"),tello_address)
     time.sleep(1)
#### Change target
Change the detect target in`detect_UAV.py` to your specific target name. For example, target="Gemini".
Make sure this target exists in your `data/custom/classes.names`.

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
