# RCIOTS - Remote Control IOT Service

- [RCIOTS - Remote Control IOT Service](#rciots---remote-control-iot-service)
  - [1. Introduction](#1-introduction)
  - [2. Getting Started](#2-getting-started)
  - [3. Security and Certificate Implementation](#3-security-and-certificate-implementation)
    - [3.1 Overview](#31-overview)
    - [3.2 Agent Deployment](#32-agent-deployment)
    - [3.3 Enrollment Process](#33-enrollment-process)
  - [4. Managing Manifests Deployment on the Edge](#4-managing-manifests-deployment-on-the-edge)
  - [5. Features](#5-features)
  - [6. Components](#6-components)
    - [6.1. Frontend](#61-frontend)
    - [6.2. Agent-Connector](#62-agent-connector)
    - [6.3. Agent](#63-agent)
    - [6.4. Kustomize](#64-kustomize)
  - [7. Roadmap](#7-roadmap)

---

## 1. Introduction

RCIoTs, short for "Remote Control Internet of Things service," represents a groundbreaking platform for managing edge devices that execute containerized workloads. It was conceived as a proof of concept and is currently in its early stages of development, with this document being authored in the final quarter of 2023. As of now, RCIoTs is a personal endeavor, and I am the sole individual responsible for its design and progress. However, I am wholeheartedly open to collaborations and contributions from the community to propel its evolution.

The overarching objective behind RCIoTs is to offer a versatile solution that can be deployed as a multi-tenant Platform as a Service (PaaS) or hosted within a private infrastructure environment. At the core of this initiative lies a fundamental commitment: simplicity. The primary emphasis is to ensure that RCIoTs is exceptionally user-friendly, requiring minimal time and resources for users to embark on their journey with it.

A secondary but crucial aspect of the project revolves around the establishment of robust communication channels between edge devices and RCIoTs. This communication is facilitated through the use of WebSockets with Mutual TLS (MTLS). The beauty of this approach is that it necessitates only an HTTPS connection to function effectively. This design decision eases the burden of communication across various network conditions, including mobile networks and corporate firewalls.

The communication channel serves a dual purpose. On one hand, it allows for the seamless deployment of workload manifests to edge devices, ensuring that they execute tasks precisely as required. On the other hand, it serves as a conduit for collecting invaluable data from the edge. This includes logs, metrics, and any other data that applications running on the edge may need to export. This section of the project is still in its conceptual stage, waiting in the garage of ideas to be further developed and to address a wide range of use cases.

In essence, RCIoTs aims to be a transformative solution that streamlines the management of edge devices with containerized workloads. With its commitment to user-friendliness and robust communication capabilities, it strives to empower organizations and individuals to leverage the full potential of the Internet of Things in an efficient and scalable manner.

## 2. Getting Started

***Prerequisites***

At present, the RCIoTs agent is designed to run on the following platforms:

- **Red Hat Edge Devices:** This includes the new distribution of RHEL for Edge, which incorporates MicroShift—a minimalistic distribution of OpenShift.

- **Red Hat Single Node OpenShift:** This distribution of OpenShift is tailored to run on a single node, combining the control plane and workload.

- **Red Hat OpenShift:** While not its primary purpose, the agent can also be run for testing on OpenShift clusters since it utilizes the necessary components available within OpenShift.

As of now, it has been primarily tested on these three types of OpenShift. The agent communicates with the `oc` binary (OpenShift CLI). However, it can also be adapted for use with `kubectl`, or the manifests can be applied through scripts, API calls to the edge, or device service management. The agent's source code can be found at [https://github.com/rciots/rciots-agent](https://github.com/rciots/rciots-agent).

**Network Requirements:** Edge devices only need to connect to `https://enroll.rciots.com:443` and `https://edge.rciots.com:443`. The first connection is required for the initial enrollment, during which the agent receives client certificates to establish communication using WebSockets with Mutual TLS (MTLS) via the second connection. Once connected, certificate renewal is handled via WebSockets.

***Registration at www.rciots.com***

If you haven't already registered, click on the "Create an account" link on [www.rciots.com](www.rciots.com). Provide your email, username, and password to complete the registration.

***Creating an Enrollment Token***

To create an enrollment token, follow these steps:

1. Navigate to "Devices" > "Provisioning Tokens."
2. Create a new provisioning token.

***Running the Agent***

Once you have the token, you can run the agent using the generated token. To do this, execute the following command. You can find the command in the token creation template.

```
oc process -f https://raw.githubusercontent.com/rciots/rciots-agent/main/manifest/openshift-template/template.yaml token=[TOKEN] devicename=device001 namespace=rciots-agent -o yaml | oc create -f -
```

***Device Approval***

If the token was created automatically, the device is automatically accepted. If you deselected this option during token creation, the device(s) will appear in the "Pending Devices" view, awaiting approval.

***Check the devices***

Go to the Devices --> current devices view to check your new managed devices.

Congratulations! By following these simple steps, you have successfully connected your device to the platform. The next step is to define the workload to run on the devices, set it individually or create a reusable Template, based on Kustomize, to deploy the workload you wish to run on the edge. This can involve pointing to a GitHub repository or defining YAML directly through the interface.

## 3. Security and Certificate Implementation

### 3.1 Overview

The communication between the Edge and the PaaS is encrypted using certificates, and the WebSocket channel is established with MTLS (Mutual TLS). This section will focus on the enrollment process, a crucial component where the magic happens.

### 3.2 Agent Deployment

The agent is publicly available, and its code and manifests are accessible. The only distinguishing factor between agents deployed by different tenants is the Provisioning Token, linking the device and its associated user.

Upon deployment, the agent is equipped with the CA (Certificate Authority) certificate and an intermediate certificate to authenticate the connection to the PaaS endpoint.

### 3.3 Enrollment Process

- Step 1: Initial Authentication

The agent initiates the enrollment process with an initial call to https://enroll.rciots.com, providing the Provisioning Token as authentication. If the token is valid, the process proceeds to the next step; otherwise, a 404 "Token not found" response is returned.

- Step 2: Token Validation and Device Registration

Once the device is authenticated with the token, a validation check is performed to ensure the token is enabled, belongs to an active user, and the device is registered, assigning it an ID and a device_token. The process also verifies if the enrollment mode is automatic or manual. In the case of automatic enrollment, the process advances to the next step. For manual enrollment, the device is registered but awaits approval in the "Pending Devices" window. Upon approval, it proceeds as if it were an automatic enrollment.

- Step 3: Client Certificate Generation

With the device authenticated and validated, a unique key and certificate are generated for the device using a privateCA managed at the PaaS. These are sent to the client as a response to their enrollment request, along with the device_id and device_token.

The agent is responsible for updating the secrets to store this information for future restarts. Currently, to avoid unnecessary restarts, this information is stored in memory and /tmp directory of the container.

- Step 4: WebSocket MTLS Connection Establishment

With the client's certificate and key in hand, the MTLS connection is established. Additionally, the device_id and device_token serve as authentication parameters, enhancing security beyond relying solely on client certificate revocation. This approach reduces the risk of undesired connections, as the server can take some time to detect and reject the client certificates.

## 4. Managing Manifests Deployment on the Edge

Once the Edge connects to the PaaS, we have the flexibility to interact with it as desired, enabling bidirectional real-time information exchange. In a previous experiment, I utilized the same libraries to create an arcade machine that could be controlled from anywhere in the world, streaming webcam footage with a latency of about 60ms, all facilitated through websockets. However, let's start with the basics, which I believe generate significant interest when it comes to administering Edge devices running containers – the management of the workload they will execute.

The first functionality I have implemented is the ability to manage, from the PaaS, the manifests we want to run on the Edge's MicroShift. I have opted for the use of Kustomize because, from my perspective, it provides a straightforward system for having a base template. If we have 1 or 1000 devices, each with variations in the configuration to be deployed, Kustomize allows us to start with common manifests. Additionally, it supports GitOps by pointing to a code repository, facilitating efficient configuration updates across multiple devices.

## 5. Features

WIP

## 6. Components

WIP

### 6.1. Frontend

WIP

### 6.2. Agent-Connector

WIP

### 6.3. Agent

WIP

### 6.4. Kustomize

WIP

## 7. Roadmap

WIP