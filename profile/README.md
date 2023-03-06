# RCIOTS - Remote Control IOT Service

## Introduction

In this organization https://www.github.com/rciots are hosted the repositories used to deploy as a proof of concept, the first "Cloud Native" online arcade machine https://claw.rciots.com (not always available), which runs on Microshift (edge-* repos) and Openshift (dc-* repos).

[![Youtube example claw.rciots.com](https://raw.githubusercontent.com/rciots/.github/main/profile/images/youtube.png)](https://www.youtube.com/watch?v=Yd9cCu0k9_g)

## Hardware

The hardware used in the solution is the following:

Intel NUC 10: running Single Node Openshift
Fitlet2: Running Microshift

Components:
- USB webcam
- USB Arduino Mega 2560 + CNC shield v3 Hat + 3pcs A4988
- 3 x nema 17 stepper motors
- 4 x end stop sensors with signal
- A claw from a claw machine
- 12v 10A 120W power supply (power for the motors, claw and Led illumination)
- 5v relay (open / close the claw)

## Software

Hi there! I'm Mario Parra, Cloud Consultant, not developer, so I have started the project with the knowledge that I have in my hand, and in my case it's a bit of Javascript applied for full-stack running with NodeJS.

All the services are created under the same structure, using differents libraries like:
- Express: web application framework
- Socket.io: websocket server for realtime stream, used at front, dc-socket-manager and edge-ws-connector
- Socket.io-client: client library of Socket-io
- Johnny-five: Control the robotics parts (Arduino) with Firmata from edge-controller service

Also for cam streaming edge-cam is using ffmpeg package.

## Services

All services run on the RHEL Universal Base Image 9 + NodeJS 16 image: registry.access.redhat.com/ubi9/nodejs-16:latest

Its repositories contain two folders:
    - code: where we have the Javascript code of the application and the Dockerfile for its construction.
    - manifests: where are the objects to be created in the Microshift / Openshift api.

All services are built in quay.io's rciots organization https://quay.io/organization/rciots, except for edge-cam as it needs the ffmpeg package and is built on a RHEL machine with active subscription to enable the repository *"codeready-builder-for-rhel-9-x86_64-rpms"*

![claw diagram](https://raw.githubusercontent.com/rciots/.github/main/profile/images/claw.rciots.png)

### dc-front

### dc-socket-manager

### edge-ws-connector

### edge-cam

### edge-controller