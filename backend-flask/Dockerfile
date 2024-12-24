# Creation of Container
# Docker is used to Build the Container Image
FROM python:3.10-slim-buster

# Inside Container
# Make a New Folder Inside the Container
WORKDIR /backend-flask

# Outside Container -> Inside Container
# This  Install the Python Libraries Used for the Application
# Inside the Container
COPY requirements.txt requirements.txt

# Inside Container
RUN pip3 install -r requirements.txt

# Outside Container -> Inside Container
# This will Copy Everything Inside the Current Directory
# First Period . /backend-flask(Outside Container)
# Second Period ./backend-flask(Inside Container)
COPY . .

# This will SETUP the Environment Variable(Env Vars)
# Inside the Container and Will Remain Set When the App is Running Inside the Container
ENV FLASK_ENV=development

EXPOSE ${PORT}

# CMD: Command
# This is the Command that is Used to Run the Flask Application

# python3 -m flask run --host=0.0.0.0 --port=4567
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]

# Host OS: This is the Actual Computer that is Running the Operating System(OS),
# Guest OS: This has to do with the Actual Virtualization on the Virtual Machine.
# HyperVisor will Virtualize the Computer on the Computer(PC), The HyperVisor will 
# Allowed the Guest OS to run on the Computer(PC) HERE
