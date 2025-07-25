# Data Collection & Initial Scope

This section details the comprehensive process undertaken to collect the raw drone flight log messages that form the foundation of our NLP analysis. Our data sources include publicly available information from AirData UAV and forensic artifacts from VTO Labs.

---

## 1. AirData UAV Message Collection

The primary source for a broad range of drone notification messages was the AirData UAV platform.

* **Source URL:** [https://app.airdata.com/wiki/Notifications](https://app.airdata.com/wiki/Notifications)

* **Collection Procedure:**
    The collection process involved navigating to the provided AirData UAV Wiki URL and manually copying all messages presented in English. These messages represent a diverse set of notifications, warnings, and status updates generated by various drone models and flight scenarios.

* **Observed Discrepancy:**
    The AirData website indicated a total of **508 messages** available on the page. However, upon copying and transferring these messages into an Excel file for initial organization, we consistently found only **499 unique messages**. The exact reason for this discrepancy (e.g., dynamic loading, filtering, or hidden characters not captured during copying) remains unconfirmed. Despite this minor difference, the collected 499 messages provide a substantial and representative sample of AirData notifications.

---

## 2. VTO Labs Flight Log Extraction

In addition to AirData, a significant portion of our raw data was derived from forensic images of drone artifacts provided by VTO Labs. This source offers real-world flight log data, often containing more granular and context-specific messages.

* **VTO Labs Project Report:**
    A comprehensive overview of the VTO Labs Drone Forensics Program, including details on data acquisition and analysis, can be accessed via their official report:
    [https://dfrws.org/wp-content/uploads/2019/06/pres_drone_forensics_program.pdf](https://dfrws.org/wp-content/uploads/2019/06/pres_drone_forensics_program.pdf)

* **Raw Image Collection (Google Drive):**
    The raw forensic images from various drone models are publicly available on Google Drive:
    [https://drive.google.com/drive/folders/1-UrxFGpCo54bVujwFmmqNbsZEV28dSNz](https://drive.google.com/drive/folders/1-UrxFGpCo54bVujwFmmqNbsZEV28dSNz)

* **Message Source Identification:**
    Upon initial examination of the VTO Labs data, it was observed that human-readable flight log messages were consistently found **only within log files acquired from controller devices**. Logs from other drone components (e.g., the aircraft itself) were often in proprietary binary formats or contained only telemetry data without explicit textual messages relevant to our NLP tasks.

    !!! info "Detailed Extraction Guide"
        For a comprehensive, step-by-step guide on how we extracted and decrypted these flight log files from the VTO Labs forensic images, please refer to our dedicated blog post:
        [:octicons-arrow-right-24: Extracting Flight Logs from VTO Labs Forensic Images](../blog/vto-labs-extraction.md)


* **Extraction and Decryption Process:**
    The extraction process for VTO Labs data involved the following steps:

    1.  **Controller Log Identification:** For each drone model and dataset ID, all flight log files originating from the `controller_device` artifact were identified.
    <!-- 2.  **Local Storage Structure:** These raw, encrypted flight logs were organized and stored locally using the following hierarchical folder structure to maintain provenance:
        ```
        Drone_Model/
        ├── DatasetID/
        │   └── YYYY_Month/
        │       └── controller_device/
        │           └── DJIFlightRecord_YYYY-MM-DD_[HH-MM-SS].txt
        ```
        *Example Filename:* `DJIFlightRecord_2017-08-29_[14-30-27].txt` -->
    3.  **Decryption:** The raw `.txt` flight log files from DJI drones are typically encrypted. To obtain human-readable messages, each file was individually decrypted using the **DJI Phantom Help Log Viewer** online tool.
        * **Tool Used:** [https://www.phantomhelp.com/logviewer/upload/](https://www.phantomhelp.com/logviewer/upload/)
        * **Procedure:** Each encrypted `.txt` log file was uploaded to the Phantom Help Log Viewer, decrypted, and then the resulting human-readable data was downloaded as a `.csv` file. This was a manual, file-by-file process.
    4.  **Message and Timestamp Extraction:** From the decrypted `.csv` flight logs, specific columns containing textual messages and their corresponding timestamps were extracted.
        * **Extracted Columns:** `APP.message`, `APP.tip`, and `APP.warning`.
        * **Forensic Timeline Construction:** For each log, a simplified forensic timeline was constructed by pairing the extracted messages with their precise timestamps (date and time). This timeline forms the basis for subsequent cleansing and annotation.

!!! note "Data Sensitivity"
    While the VTO Labs raw images are publicly available, the decryption and extraction process yields more detailed information. Our focus is on the *messages* themselves, and any sensitive contextual data not directly part of the message content is omitted from the final dataset.

---

## 3. Combined Raw Message Collection

The messages collected from both AirData UAV and the VTO Labs decrypted flight logs constitute the initial raw message collection for our NLP analysis. This diverse collection provides a robust foundation for identifying common patterns of noise and developing effective cleansing and annotation strategies.

---