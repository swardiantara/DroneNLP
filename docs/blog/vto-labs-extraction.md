# Extracting Flight Logs from VTO Labs Forensic Images

**Date:** July 23, 2025
**Author:** S. Silalahi

This blog post provides a detailed, step-by-step walkthrough of the process we employed to extract and decrypt human-readable flight log messages from the forensic images of drone controller devices, as sourced from the VTO Labs Drone Forensics Program. This process was a crucial part of building our comprehensive "NLP for Drone Flight Log Analysis" dataset.

---

## 1. Understanding the Source Data

The VTO Labs dataset (accessible via [https://drive.google.com/drive/folders/1-UrxFGpCo54bVujwFmmqNbsZEV28dSNz](https://drive.google.com/drive/folders/1-UrxFGpCo54bVujwFmmqNbsZEV28dSNz)) consists of forensic images from various drone models and components. Our initial analysis revealed that relevant, human-readable flight log messages were predominantly found within data acquired from **controller devices**. Other components often contained encrypted, proprietary, or purely telemetry data not suitable for direct NLP analysis.

## 2. Locating and Extracting Flight Log Files from Controller Artifacts

The VTO Labs collection of drone images includes data acquired from various controller devices, specifically **Android phones**, **Android tablets**, and **iOS phones**. The extraction methodology varies slightly depending on the operating system and artifact file type. Our goal was to identify and extract files containing human-readable log messages.

### 2.1. Android-Based Controllers (Phones & Tablets)

For Android-based controller artifacts, VTO Labs provides two main types of files: `.zip` archives and `.001` forensic images.

#### 2.1.1. For `.zip` Archives:

These archives can often be directly unzipped using standard archival tools. Once unzipped, the flight logs are typically found in the following directories:

* **Encrypted `.txt` Flight Log Files (`DJIFlightRecord_YYYY-MM-DD_[HH-MM-SS].txt`):**
    * `/dji/dji.go.v4/FlightRecord/`
    * `/dji/dji.pilot/FlightRecord/`
* **Unencrypted Human-Readable Error Logs:**
    * `/dji/dji.go.v4/LOG/ERROR_POP_LOG/`
    * `/dji/dji.pilot/LOG/ERROR_POP_LOG/`
    These folders contain simpler, often plain-text, human-readable error messages that did not require decryption.

#### 2.1.2. For `.001` Forensic Images:

For artifacts provided as `.001` forensic images (which require specialized forensic tools to mount and access), we used [**Autopsy**](https://www.autopsy.com/), an open-source digital forensics platform, to navigate and extract relevant files.

* **Tool Used:** Autopsy (open-source digital forensics platform)
* **Extraction Path:** Within Autopsy, we gained access to the file system and extracted the encrypted `.txt` flight log files from paths similar to:
    * `/dji/dji.go.v4/FlightRecord/`
    * `/dji/dji.pilot/FlightRecord/`

### 2.2. iOS-Based Controllers (iPhones)

For iOS-based controller artifacts, all data was provided in `.zip` archives.

* **Extraction Method:**
    * Most `.zip` files from iOS controllers could be **directly unzipped** to reveal their contents.
    * For any `.zip` files that resisted direct unzipping or appeared corrupted, we reverted to using **Autopsy** to access their internal structure and extract the files containing human-readable log messages.

### 2.3. Post-Extraction Processing for All Controller Logs

After collecting both the encrypted `.txt` flight log files and the unencrypted `ERROR_POP_LOG` files (where applicable) from both Android and iOS sources:

* The encrypted `.txt` log files (e.g., `DJIFlightRecord_YYYY-MM-DD_[HH-MM-SS].txt`) were then decrypted using the DJI Phantom Help Log Viewer, as detailed in the next section.
* The unencrypted `ERROR_POP_LOG` files were immediately ready for inclusion in our raw message collection without further decryption.

<!-- ## 2. Locating Encrypted Flight Log Files

Within the controller device artifacts, we identified files typically named in the format `DJIFlightRecord_YYYY-MM-DD_[HH-MM-SS].txt`. These files are encrypted and contain the raw flight data. After going through all the available artifacts, we compile the found flight log files, named them DroNER and made it publicly accessible on [Mendeley Data](https://data.mendeley.com/datasets/fwcjyc754h/1).

**File Path Structure (Conceptual):**
    ```
    Drone_Model/
    └── DatasetID/
        └── YYYY_Month/
            └── controller_device/
                └── DJIFlightRecord_2017-08-29_[14-30-27].txt
    ```
Under the `raw` folder of the `droner`, the extracted log files are arranged in the above structure. For instance, a log file from DJI Matrice 600 model is stored in `DJI_Matrice_600\df034\2018_June\mobile_android_logical` directory, with a sample flight log file name of `DJIFlightRecord_2018-06-20_[10-11-44]`. -->

## 3. Decrypting the Logs with DJI Phantom Help

DJI flight logs are encrypted, necessitating a decryption step. We utilized the online **DJI Phantom Help Log Viewer** for this purpose.

* **Tool Used:** [https://www.phantomhelp.com/logviewer/upload/](https://www.phantomhelp.com/logviewer/upload/)

* **Step-by-Step Decryption:**
    1.  **Access the Tool:** Navigate to the DJI Phantom Help Log Viewer website.
    2.  **Upload File:** Click the "Upload" button and select an encrypted `DJIFlightRecord_*.txt` file from your local `controller_device` directory.
    3.  **Process:** The tool automatically processes and decrypts the file.
    4.  **Download CSV:** Once decrypted, download the resulting human-readable data as a `.csv` file.
    5.  **Repeat:** This manual process was performed for each individual encrypted flight log file.

    !!! warning "Manual Process Note"
        It's important to note that this was a file-by-file manual decryption process due to the nature of the online tool. For very large collections, this step can be quite time-consuming.

## 4. Extracting Relevant Messages for NLP

The downloaded CSV files contain numerous columns, encompassing various types of flight data. For our NLP analysis, we focused on columns specifically containing human-readable messages.

* **Key Columns Extracted:**
    * `APP.message`: Contains general operational messages.
    * `APP.tip`: Provides advisory or tip messages.
    * `APP.warning`: Contains warning or error messages.

* **Constructing the Forensic Timeline:**
    For each extracted log, we then paired these messages with their precise timestamps. This allowed us to reconstruct a chronological "forensic timeline" of events and communications from the drone system during the flight. This timeline became the raw input for our data cleansing procedures.

    * **Timeline Elements:** `Timestamp (Date and Time)` + `Message Content`

---

## Conclusion

This meticulous extraction and decryption process was fundamental in transforming raw, often inaccessible, drone flight data into a rich source of textual information for our NLP research. It highlights the practical challenges involved in acquiring and preparing data from real-world forensic artifacts.