# Planning

## Purpose

This project will resolve a variety of problems and have more than a single use. I will be using it on the family's primary vehicle that is typically driven by my wife, occasionally driven by myself, and all too soon will be driven by my son. I will attempt to address the needs of each driver, as well as vehicle maintenance in general.

## Problems to Solve

### Regular Drivers

My wife is the usual driver. She is a very capable driver, but like the majority of drivers, is not alway able to recognize, understand, troubleshoot, or communicate technical issues (particularly those that arise in a 160K mile vehicle).

*   A diagnostic code is triggered, but it is difficult to communicate back to me exactly what it is.
*   A diagnostic code is triggered, but the fact that there is a code is forgotten.
*   The vehicle is broken down or I otherwise must go to the location, but direction are difficult to communicate.
*   The vehicle has been parked and exited, but left unlocked.
*   Useful information, such as weather activity and traffic issues is not available while driving.

### New Drivers

*   The driver operates the vehicle in an unacceptable manner, including hard acceleration, exceeding the speed limit, or traveling outside of expected bounds.
*   The new driver is lost or otherwise unable to convey their current location.
*   The parent's of the new driver would like to review cumulative data, such as total miles driven, gas used, number of times driven in an unacceptable manner, etc.

### Vehicle Maintenance

*   Diagnostics codes are not noticed.
*   Diagnostics codes are vague
*   Fuel level is unknown
*   Time to next service is unknown

## Basic Architecture

In order to support the current desired usage and feature set, the project will consist of an Arduino controller that interfaces with or includes a wifi module to send MQTT messages to a broker, a module to interpret can bus traffic on the OBD2 port, a GPS module to report current location, a wifi hotspot to provide internet access, a circuit to step down the voltage to the Arduino, and possibly a circuit to manage the power state of the hotspot and/or the Arduino to prevent power drain. 

The MQTT broker will manage the data received from the car, allowing for subscription by an application in charge of monitoring and sending notifications to smart devices, smart home systems for home integration, and an application that tracks and stores the current status information to be rendered via a mobile or web app.

It may make more sense to create a public facing REST server to interface directly with the Arduino instead. It would then manage the historical and current state data, then only send more refined messages to the MQTT broker.

## Todo Before Project Start

*   Investigate current solutions and existing features. I'll probably still build my own anyway.
*   Determine the best controller to use. ESP32 is likely powerful enough and has built in wifi. The bluetooth could be useful as well.
*   Pick a module for interpreting can bus traffic over OBD2. Ideally, it should be capable of two way communication in case I find a good use case for sending can bus messages to control things like door locks.
*   Pick a GPS module. Price aside, it should be reliable and, if possible, not require an external antenna. 
*   Design a power management circuit to handle stepping down the 12v signal from the OBD2 port to 5v for the arduino, and disconnecting the arduino/hotspot from the power if needed.
*   Select a cheap hotspot and cheap/free provider for a small data plan.
*   Determine what form messages should be sent to MQTT (or if that is even the correct solution)
*   Finalize architecture
    *   Configurable based on PID for use in other vehicles.
    *   Solution for communication can bus, GPS, and network availability to the controller.
    *   Solution for communicating data to a public endpoint for collection into a historical and current state model.
    *   Solution for triggering smart device notifications based on collected data.
    *   Solution for displaying data in a user interface.

## Desired Features

*   Utility Features
    *   Power management to prevent power drain when the ignition is off.
    *   Interface with a can bus module for can bus traffic monitoring.
    *   Listen for specified can bus messages that should be sent to the MQTT broker.
    *   Translate can bus messages to useful data to be sent to the MQTT broker.
    *   Regularly poll for diagnostic codes and send to the broker.
    *   Connect to a wifi hotspot on power up.
    *   Attempt to reconnect to wifi at a regular interval when the connection is lost.
    *   Buffer data that fails to send or is collected while not connected to wifi or the internet so that it can be sent later
        *   May require an SD card module.
    *   Interface with a GPS module to make location data available.
    *   Regularly poll and send GPS coordinates.
    *   Stretch Goals
        *   Subscribe to the MQTT broker to receive messages to the car.
        *   Create a security model to handle issuing can bus messages to control the vehicle.
        *   Lock/unlock doors when message received.
        *   Disable starting or ignition when message received
*   Status Report
    *   Create a service to record all data to a model representing historical report data.
        *   GPS driven route data
        *   MPG over time
        *   Diagnostics codes over time
        *   Miles driven per average day, week, month
    *   Create a service (or reuse above) to aggregate current and historical data into a current status model:
        *   Fuel level.
        *   GPS coordinates
        *   Door lock state
        *   Ignition state
        *   Last known speed
        *   Average speed over t time
        *   Diagnostics codes
        *   Time to next service
        *   Miles remaining (fuel)
        *   Tire inflation status
    *   Create an application to display the current and historical data to the end user.
*   Notifications
    *   New diagnostic code triggered.
        *   Include a description, possible causes, and estimated repair cost.
    *   Tire pressure low.
    *   Speed limit exceeded by a defined threshold.
    *   Acceleration rate exceeded by a defined threshold.
    *   Geofence left/entered.
    *   Doors left unlocked.
    *   Approaching dangerous weather
    *   Approaching road closure
    *   Approaching accident
