import cv2
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from paho.mqtt import client as mqtt_client

# MQTT Configuration
BROKER = 'broker.hivemq.com'  # Public broker for testing
PORT = 1883
TOPIC = "home/sensor/alert"
CLIENT_ID = "smart_home_client"

# Email Configuration
SMTP_SERVER = 'smtp.gmail.com'
SMTP_PORT = 587
SENDER_EMAIL = 'your_email@gmail.com'
SENDER_PASSWORD = 'your_email_password'
RECIPIENT_EMAIL = 'recipient_email@gmail.com'

# Camera Configuration
CAMERA_INDEX = 0  # Default camera, change if you have multiple cameras

# Function to send an email notification
def send_notification(subject, body):
    try:
        msg = MIMEMultipart()
        msg['From'] = SENDER_EMAIL
        msg['To'] = RECIPIENT_EMAIL
        msg['Subject'] = subject
        msg.attach(MIMEText(body, 'plain'))

        server = smtplib.SMTP(SMTP_SERVER, SMTP_PORT)
        server.starttls()
        server.login(SENDER_EMAIL, SENDER_PASSWORD)
        server.sendmail(SENDER_EMAIL, RECIPIENT_EMAIL, msg.as_string())
        server.quit()
        print("Notification sent successfully.")
    except Exception as e:
        print(f"Failed to send notification: {e}")

# Function to capture an image using the camera
def capture_image():
    cap = cv2.VideoCapture(CAMERA_INDEX)
    if not cap.isOpened():
        print("Error: Could not access the camera.")
        return

    ret, frame = cap.read()
    if ret:
        image_path = 'intruder_alert.jpg'
        cv2.imwrite(image_path, frame)
        print(f"Image captured and saved to {image_path}.")
    else:
        print("Error: Failed to capture image.")
    cap.release()

# MQTT callback when a message is received
def on_message(client, userdata, msg):
    print(f"Received message: {msg.payload.decode()}")
    # Example action on alert
    capture_image()
    send_notification(
        "Smart Home Alert",
        "An alert was detected by the sensor. Check the attached image."
    )

# Function to connect to the MQTT broker
def connect_mqtt():
    client = mqtt_client.Client(CLIENT_ID)
    client.on_connect = lambda client, userdata, flags, rc: print("Connected to MQTT broker.")
    client.on_message = on_message
    client.connect(BROKER, PORT)
    return client

# Main function to run the MQTT client
def run():
    client = connect_mqtt()
    client.subscribe(TOPIC)
    client.loop_start()  # Start loop to process incoming messages
    print(f"Listening to sensor alerts on topic: {TOPIC}")

if __name__ == '__main__':
    run()
