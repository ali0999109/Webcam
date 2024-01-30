# Webcam monitor overview.
<p>
This app is built to monitor when someone or something comes into the webcams view to send an email alert with a picture.

  
</p>


## App in action, i will use an item to trigger the email alert

![image](https://github.com/ali0999109/Webcam/assets/145396907/8374342b-f721-4680-be0f-413f864e75c4)


---------------------------------------------------------------------------------------

![image](https://github.com/ali0999109/Webcam/assets/145396907/3cb126ca-41ff-4cbd-a49b-a7d58a7d4068)





# Code for frontend and backend email setup.
``` import cv2
import time
import glob
import os
from email_link import send_email
from threading import Thread

video= cv2.VideoCapture(0)
time.sleep(1)
first_frame = None
status_list = []
count = 1

def clean_folder():
    print ("clean folder function started")
    images= glob.glob("images/*.png")
    for image in images:
        os.remove(image)
    print ("clean folder function ended")




while True:
    status = 0
    check,frame = video.read()

    gray_frame = cv2.cvtColor(frame,cv2.COLOR_BGR2GRAY)
    gray_frame_gau = cv2.GaussianBlur(gray_frame,(21,21),0)

    if first_frame is None:
        first_frame = gray_frame_gau

    delta_frame = cv2.absdiff(first_frame,gray_frame_gau)

    thresh_frame = cv2.threshold(delta_frame,60,255,cv2.THRESH_BINARY)[1]
    dil_frame = cv2.dilate(thresh_frame,None,iterations=2)
    cv2.imshow("My Video",dil_frame)

    contours,check = cv2.findContours(dil_frame,cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    for contour in contours:
        if cv2.contourArea(contour) < 5000:
            continue
        x,y,w,h = cv2.boundingRect(contour)
        rectangle = cv2.rectangle(frame,(x,y),(x+w,y+h),(0,255,0),3)
        if rectangle.any:
            status = 1
            cv2.imwrite(f"images/{count}.png", frame)
            count = count + 1
            all_images = glob.glob("images/*.png")
            index = int(len(all_images) / 2)
            image_with_object = all_images[index]

    status_list.append(status)
    status_list = status_list[-2:]

    if status_list[0] == 1 and status_list[1] == 0:
        email_thread = Thread(target=send_email,args=(image_with_object, ))
        email_thread.daemon = True
        clean_thread = Thread(target=clean_folder)
        clean_thread.daemon = True
        email_thread.start()

    print (status_list)

    cv2.imshow("video",frame)




    key = cv2.waitKey(1)
    if key == ord('q'):
        break

video.release()

clean_thread.start()




# Back end email alert setup
import smtplib
import imghdr
from email.message import EmailMessage
password = "tqvdoowzzuuuftqg"
sender = "aazad349@gmail.com"
receiver = "aazad349@gmail.com"


def send_email(image_path):
    print ("send email function started")
    email_message = EmailMessage()
    email_message["Subject"] = "Object detected"
    email_message.set_content("Here is a picture")


    with open(image_path,"rb") as file:
        content = file.read()
        
    email_message.add_attachment(content,maintype="image",subtype=imghdr.what(None,content))

    gmail = smtplib.SMTP("smtp.gmail.com",587)
    gmail.ehlo()
    gmail.starttls()
    gmail.login(sender,password)
    gmail.sendmail(sender,receiver,email_message.as_string())
    gmail.quit()
    print ("send_email function ended")


if __name__ == "__main__":
    send_email()










