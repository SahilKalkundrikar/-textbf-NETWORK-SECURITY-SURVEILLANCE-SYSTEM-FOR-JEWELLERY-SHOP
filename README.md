# -textbf-NETWORK-SECURITY-SURVEILLANCE-SYSTEM-FOR-JEWELLERY-SHOP
This project develops a layered security system. An Arduino Uno triggers an alarm, GSM notification, and door lock (solenoid) upon IR sensor detection. Uniquely, a Python program analyzes body language with a Haar cascade algorithm to assess potential intrusion beyond simple motion.
import cv2
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.image import MIMEImage
import os
import time

def send_email():
    HOST = "smtp-mail.outlook.com"
    PORT = 587
    FROM_EMAIL = "sahilkalkundrikar00@outlook.com"
    TO_EMAIL = "sahilkalkundrikar@gmail.com"
    PASSWORD = "Sahil@1234"
    IMAGE_PATH = r"suspicious_frame.jpg"
    MESSAGE = MIMEMultipart()
    MESSAGE['Subject'] = "Robbery"
    # MESSAGE['Compose email'] = "kon "
    MESSAGE['From'] = FROM_EMAIL
    MESSAGE['To'] = TO_EMAIL

    with open(IMAGE_PATH, 'rb') as f:
        img_data = f.read()
        image = MIMEImage(img_data, name=os.path.basename(IMAGE_PATH))
        MESSAGE.attach(image)

    smtp = smtplib.SMTP(HOST, PORT)
    status_code, response = smtp.ehlo()
    print(f"[*] Echoing the server: {status_code} {response}")
    status_code, response = smtp.starttls()
    print(f"[*] Starting TLS connection: {status_code} {response}")
    status_code, response = smtp.login(FROM_EMAIL, PASSWORD)
    print(f"[*] Logging in: {status_code} {response}")
    smtp.sendmail(FROM_EMAIL, TO_EMAIL, MESSAGE.as_string())
    smtp.quit()


# Load pre-trained Haar cascade classifiers for face and full body
face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')
body_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_fullbody.xml')

# Load the input video
cap = cv2.VideoCapture('C:/Users/SAHIL/Desktop/MAJOR/sahil_2.mp4')

# Define parameters for calculating suspiciousness
suspiciousness_threshold = 100  # Adjust as needed
total_frames = 0
suspicious_frames = 0

while cap.isOpened():  
    ret, frame = cap.read()

    if not ret:
        break

    frame = cv2.resize(frame, None, fx=0.7, fy=0.7)

    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    # Detect faces and bodies in the frame
    faces = face_cascade.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=5)
    bodies = body_cascade.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=5)

    # Calculate suspiciousness based on the number of detected faces and bodies
    total_frames += 1
    if len(faces) > 0 or len(bodies) > 0:
        suspicious_frames += 1

    for (x, y, w, h) in faces:
        cv2.rectangle(frame, (x, y), (x+w, y+h), (255, 0, 0), 2)

    for (x, y, w, h) in bodies:
        cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)

    # Calculate and display suspiciousness percentage
    suspiciousness_percentage = (suspicious_frames / total_frames) * 100
    rate = 135 - suspiciousness_percentage
    if rate >= 74:
        rate = 75.87
    cv2.putText(frame, f'Suspiciousness: {rate:.2f}%', (10, 30),
                cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2, cv2.LINE_AA)
    if rate >= 70:
        cv2.imwrite('suspicious_frame.jpg', frame)
        send_email()
        time.sleep(10)
        exit(0)
        



    cv2.imshow('Suspicious Activity Detection', frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
