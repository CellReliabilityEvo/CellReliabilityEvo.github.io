<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>
<br />
<center style="font-size:45px;color:green;line-height:-10px"> A Five-Year Retrospective of Cellular Reliability Evolution:</center>
<center style="font-size:32px;color:green;line-height:110px;white-space:nowrap;"> The Encouraging, The Disappointing, and Further Enhancements</center>

![license](https://img.shields.io/badge/Platform-Android-green "Android")
![license](https://img.shields.io/badge/Version-Beta-yellow "Version")
![license](https://img.shields.io/badge/Licence-Apache%202.0-blue.svg "Apache")

## Table of Contents
[Introduction](#introduction)

[Codebase Organization](#codebase-organization)
 - [Continous Monitoring Infrastructure](#continous-monitoring-infrastructure)

 - [Stability-Compatible RAT Transition](#stability-compatible-rat-transition)

 - [TIMP-based Flexible Data_Stall Recovery](#timp-based-flexible-data_stall-recovery)

 - [Meticulous Date_Setup_Error Judgement](#timp-based-flexible-data_stall-recovery)

[Platform Requirements](#platform-requirements)

[Data Release](#data-release)

[For Developers](#for-developers)

## Introduction
This repository contains our continous monitoring infrasturcture (based on Android-MOD, a customized Android system that records system-level traces upon the occurrence of suspicious cellular failure events) for capturing cellular failures in the wild, as well as our efforts for improving cellular reliability on Android devices. Our latest Android-MOD system is built upon vanilla Andorid 13/14. Therefore, you'll be able to run codes in this repo by patching these modifications to proper framework components.

### [The entire codebase and sample data are available in our [Github repo](https://github.com/CellReliabilityEvo/CellReliabilityEvo.github.io).]

### Continous Monitoring Infrastructure
Our modifications mainly involve the telephony component in the framework layer (whose location in AOSP tree is `frameworks/opt/telephony/src/java/com/android/internal/telephony`, denoted later as `TELEPHONY_SRC`).
Specifically, we modify the `DataStallRecoveryManager.java`, `DataNetwork.java` and `DefaultPhoneNotifier.java` to instrument concerned failure points (currently we only list those related to the three major cellular failtures--Data_Stall, Data_Setup_Error, and Out_of_Service), as shown in [Monitor](https://github.com/CellReliabilityEvo/CellReliabilityEvo.github.io/tree/main/monitor):

| Class | Failure Point | Purpose| Location in AOSP |
| ---- | ---- | ---- | ---- |
|   `DataStallRecoveryManager`   |   `handleMessage`   |   Tracking  Data_Stall events  | `TELEPHONY_SRC/data/DataStallRecoveryManager.java` |
|   `DataNetwork.DisconnectedState`   |   `enter`   |   Tracking  Data_Setup_Error events  | `TELEPHONY_SRC/data/DataNetwork.java` |
|   `DefaultPhoneNotifier`   |   `notifyServiceState`   |   Tracking  Out_of_Service events  | `TELEPHONY_SRC/DefaultPhoneNotifier.java` |

Upon cellular failures, we then notify our dedicated event logging service [CellularStateProcessor](https://github.com/CellReliabilityEvo/CellReliabilityEvo.github.io/tree/blob/monitor/CellularStateProcessor.java) and [CellularReliability](https://github.com/CellReliabilityEvo/CellReliabilityEvo.github.io/blob/main/monitor/CellularReliability.java) to log critical cellular and device information:

| Information | Description |
| ---- | ---- |
| `UID` | Unique ID generated to identify a user (cannot be related to the user's true indentity) |
| `TIME` | UNIX timestamp |
| `RAT` | Current radio access technology |
| `RSSI` | Signal strength in dBM |
| `CELL`| Base Station ID (MCC+MNC+LAC+CID) |
| `OS` | Android vesion |
| `MODEL` | Device model |
| `CAUSE` | Error code of Data_Setup_Error defined in `DataFailCause` |
| `APN`   | Current access point names |

For event recovery, we provide similar tracing to record recovery events. In particular, for Data_Stall events we probe the network to more accurately monitor event recovery in [DataStallDiagnostics](https://github.com/CellReliabilityEvo/CellReliabilityEvo.github.io/blob/main/monitor/DataStallDiagnostics.java).

Regarding the codes for tracing the data connection establishment for Data_Setup_Error failures, we are still dicussing with the authority to what extend can it be released.

### Stability-Compatible RAT Transition
Upon RAT transitions, our control policy would kick in to check whether current system and network states are suitable for transitions. It currently runs as a daemon thread along side the telephony service, as shown in [RATTransition](https://github.com/CellReliabilityEvo/CellReliabilityEvo.github.io/blob/main/rat_trans/RATTransition.java).

### TIMP-based Flexible Data_Stall Recovery
We currently provide our time-inhomogeneous Markov process (TIMP) that formalizes the Data_Stall recovery process and find proper triggers for entering each recovery stage. 

We implement the TIMP model in Python ([timp_model](https://github.com/CellReliabilityEvo/CellReliabilityEvo.github.io/blob/main/timp/timp_model.py)) which can automatically search in the time trigger space so as to find triggers that can minimize the expected recovery time.

### Meticulous Date_Setup_Error Judgement
We also present our report to the official development team of Android in Google and the response [here](https://github.com/CellReliabilityEvo/CellReliabilityEvo.github.io/tree/main/dse_judgement/report.pdf).

## Platform Requirements
### Android
For Android-related modifications, currently our code is run and tested in Android 13 and Android 14 (AOSP).
Note that despite quite a number of changes have been made in Android 14 since Android 13, our code is applicable to both given that concerned tracing points remain unchanged.
We have also released our modifications compatible with Android 9-12 [here](https://github.com/CellularReliability/CellularReliability.github.io/tree/main/monitor).

### Others
For other code component such as our TIMP model, they can be run on Linux with proper Python supports.

## Data Release
Our dataset contains three parts.

1) an eight-month fine-grained dataset collected by us from 01/01/2020 to 08/30/2020,

2) another eight-month fine-grained dataset collected by us from 10/01/2023 to 06/30/2024,

3) a five-year coarse-grained dataset (collected by Xiaomi operation team). 

The first part was released at [here](https://github.com/CellularReliability/CellularReliability.github.io/tree/main/sample_dataset).
For the second part, We have released a portion of data (with proper anonymization) for references [here](https://github.com/CellReliabilityEvo/CellReliabilityEvo.github.io/tree/main/sample_dataset).
As to the last part, we are still dicussing with the authority to what extend can it be released.

## For Developers
Our code is licensed under Apache 2.0 in accordance with AOSP's license. Please adhere to the corresponding open source policy when applying modifications and commercial uses.
Also, some of our code is currently not available but will be relased soon once we have obatained permissions.




