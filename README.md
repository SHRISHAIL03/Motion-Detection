# Motion-Detection
Motion Detection System with Live Image Capture and Email Notification
import cv2
import pygame
import smtplib
import time
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email import encoders

# Initialize pygame mixer for sound
pygame.mixer.init()

# Load the alarm sound
alarm_sound = pygame.mixer.Sound("C:\\Users\\ADMIN\\Downloads\\police-operation-siren-144229.mp3")

# SMTP server configuration (example for Gmail)
smtp_server = "smtp.gmail.com"
smtp_port = 587
smtp_username = "shri480n@gmail.com"
smtp_password = "djmq rawv yqev dezu"

# Email configuration
email_from = "shri480n@gmail.com"
email_to = "shri480n@gmail.com"
email_subject = "Motion Detected!"

# Function to send an email with an attached video
def send_email(video_path):
    msg = MIMEMultipart()
    msg['From'] = email_from
    msg['To'] = email_to
    msg['Subject'] = email_subject

    # Attach the video to the email using MIMEBase
    with open(video_path, "rb") as attachment:
        part = MIMEBase('application', 'octet-stream')
        part.set_payload(attachment.read())
        encoders.encode_base64(part)
        part.add_header('Content-Disposition', f'attachment; filename="{video_path}"')
        msg.attach(part)

    # Connect to the SMTP server and send the email
    with smtplib.SMTP(smtp_server, smtp_port) as server:
        server.starttls()
        server.login(smtp_username, smtp_password)
        server.sendmail(email_from, email_to, msg.as_string())

try:
    # Capture video from the webcam
    cap = cv2.VideoCapture(0)

    # Get the frames per second (fps) of the input video stream
    fps = int(cap.get(cv2.CAP_PROP_FPS))

    # Define the codec and create a VideoWriter object to save the video
    fourcc = cv2.VideoWriter_fourcc(*'XVID')
    out = cv2.VideoWriter("motion_video.avi", fourcc, fps, (640, 480))

    # Initialize previous frame
    _, prev_frame = cap.read()

    # Flag to track whether an alarm has been triggered
    alarm_triggered = False

    # Variable to store the start time of motion
    motion_start_time = 0

    while True:
        # Read current frame from the webcam
        ret, frame = cap.read()

        # Get the difference between the current and previous frame
        diff_frame = cv2.absdiff(prev_frame, frame)

        # Convert the difference frame to grayscale
        gray = cv2.cvtColor(diff_frame, cv2.COLOR_BGR2GRAY)

        # Apply threshold to the grayscale image
        _, threshold = cv2.threshold(gray, 30, 255, cv2.THRESH_BINARY)

        # Find contours in the thresholded image
        contours, _ = cv2.findContours(threshold, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        for contour in contours:
            # If the contour area is above a certain threshold, consider it as motion
            if cv2.contourArea(contour) > 1000:
                if not alarm_triggered:
                    print("Motion Detected!")
                    # Play the alarm sound
                    alarm_sound.play()
                    alarm_triggered = True

                    # Record a 10-second clip when motion is detected
                    motion_start_time = time.time()

                    while time.time() - motion_start_time <= 10:
                        out.write(frame)
                        _, frame = cap.read()

        # If there is no motion, reset the alarm trigger
        if not contours:
            alarm_triggered = False

        # Update the previous frame
        prev_frame = frame.copy()

        # Display the frames
        cv2.imshow('Motion Detection', frame)
        cv2.imshow('Difference Frame', diff_frame)

        # Press 'q' to exit the loop
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Process interrupted by user")

finally:
    # Release the webcam and close windows
    cap.release()
    out.release()
    cv2.destroyAllWindows()
    # Send an email with the attached video after recording
    send_email("motion_video.avi")
