# Data Cleansing Procedure

The raw flight log messages obtained from sources like AirData UAV, while rich in information, often contain various forms of noise that can significantly hinder subsequent Natural Language Processing (NLP) tasks. These noisy logs can manifest as inconsistent formatting, missing punctuation, over-generalized statements, or irrelevant system messages, making automated extraction and understanding challenging.

Our data cleansing procedure is a critical initial step designed to transform these raw, noisy messages into a structured, consistent, and semantically clearer format, suitable for robust NLP analysis. This process ensures the quality and reliability of our annotated dataset.

---

## 1. Accessing Raw Logs

The raw, un-cleansed flight logs from which our dataset is derived are available as part of the complete dataset download on Zenodo. This allows for full transparency and reproducibility of our cleansing process.

* **Access Raw Logs on Zenodo:** [Link to your Zenodo dataset DOI]
    * *Note: These raw logs, while representing the original source, might have undergone minimal initial parsing by the AirData platform itself. Our cleansing focuses on refining the textual content.*

---

## 2. Motivating Examples for Cleansing

To illustrate the necessity of data cleansing, consider the following examples of raw messages and their challenges for NLP:

### Example 1: Missing Punctuation & Concatenation

<div class="grid cards" markdown>

-   __Original Message__
    ```text
    165 - Ambient Light too Weak, Vision Positioningobstacle sensing is unavailable. please ensure safety during the flight
    ```
    * **Challenge for NLP:** The concatenation of "Positioning" and "obstacle" (e.g., "Positioningobstacle") makes tokenization difficult and impedes entity recognition or parsing of sensor-related information. The missing period after "unavailable" and capitalization issues further complicate sentence boundary detection and readability.

-   __Cleansed Message__
    ```text
    165 - Ambient Light too Weak. Vision Positioning/Obstacle Sensing is unavailable. Please ensure safety during the flight.
    ```
    * **Impact:** This corrected version allows for accurate tokenization, clearer identification of the system affected ("Vision Positioning/Obstacle Sensing"), and proper sentence segmentation, which is crucial for many NLP tasks.
</div>

### Example 2: Ambiguous or Over-generalized Statements

<div class="grid cards" markdown>

-   __Original Message__
    ```text
    34 - Detected side/backward/forward shock / possible collision, aircraft is pitching/rolling sharply forward/backwards/left/right
    ```
    * **Challenge for NLP:** This message is highly generalized. "Possible collision" is vague, and "pitching/rolling sharply" combines multiple distinct flight maneuvers or anomalous behaviors. Without disambiguation, it's difficult for an NLP model to infer the precise event or its cause, limiting its utility for incident analysis or specific problem identification.

-   __Analysis Goal (for cleansed data)__
    * For such messages, our cleansing aims to either refine them based on available context (if a definitive, specific interpretation is possible) or to flag them as multi-faceted warnings that require deeper human review and contextual analysis, which our subsequent annotation tasks address. This ensures that the dataset either provides precise ground truth or highlights areas of inherent ambiguity.
</div>

---

## 3. Correction Types and Procedures

Our cleansing process primarily addresses two major categories of noise:

### 3.1. Type 1: Missing Punctuation and Space Correction

These corrections focus on improving the syntactic structure and readability of messages by addressing errors commonly arising from inconsistent logging practices or data ingestion.

* **Motivation:** Incorrect punctuation and missing spaces create malformed tokens, break sentence boundaries, and impede standard NLP preprocessing steps (e.g., tokenization, part-of-speech tagging, dependency parsing). Correcting these issues ensures messages are parseable and conform to expected linguistic patterns.
* **Procedure:**
    1.  **Automated Rules:** Regular expressions and rule-based heuristics were applied to identify common patterns of missing spaces (e.g., lowercase followed by uppercase, number followed by alphabet) or missing punctuation (e.g., sentence ending without a period).
    2.  **Contextual Insertion:** Punctuation (e.g., periods, commas, slashes) was inserted based on common linguistic rules and observed patterns in well-formed messages, particularly at logical phrase or clause boundaries.
    3.  **Manual Review & Refinement:** A significant portion of these corrections involved manual review by human annotators to ensure semantic correctness and avoid introducing new errors. This iterative process involved developing new rules based on observed edge cases.

* **Examples of Type 1 Corrections:**

    | Message ID | Original Message (Excerpt)                                                     | Corrected Message (Excerpt)                                                          | Justification                                                                                                                                                                                                                                                         |
    | :--------- | :----------------------------------------------------------------------------- | :----------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | 165        | `Ambient Light too Weak, Vision Positioningobstacle sensing is unavailable.`   | `Ambient Light too Weak. Vision Positioning/Obstacle Sensing is unavailable.`        | Inserted period after "Weak" to separate independent clauses. Added "/" to clarify "Vision Positioning/Obstacle Sensing". Fixed "Positioningobstacle" concatenation.                                                                                                |
    | 204        | `Ambient Light too Weak, Forward/Backwardobstacle sensing is unavailable.`     | `Ambient Light too Weak. Forward/Backward Obstacle Sensing is unavailable.`          | Similar to 165, separated clauses with a period. Added space between "Backward" and "Obstacle".                                                                                                                                                                      |
    | 78         | `Battery voltage lowplease land`                                               | `Battery voltage low. Please land.`                                                  | Separated two distinct imperative clauses with a period. Capitalized "Please" for sentence start.                                                                                                                                                                     |
    | 112        | `GPS signal weakcannot find home point.`                                       | `GPS signal weak. Cannot find home point.`                                           | Separated two distinct clauses with a period. Capitalized "Cannot".                                                                                                                                                                                                   |
    | 312        | `Aircraft disconnectedfrom remote controller.`                                 | `Aircraft disconnected from remote controller.`                                      | Inserted missing space between "disconnected" and "from".                                                                                                                                                                                                             |
    | 405        | `Propeller errorcheck propellers`                                              | `Propeller error. Check propellers.`                                                 | Separated statement from instruction with a period. Capitalized "Check".                                                                                                                                                                                              |
    | ...        | *[More examples will be added here or linked to]* | *[More examples]* | *[More justifications]* |

### 3.2. Type 2: Over-generalized Message Analysis & Refinement

This category deals with messages that, while syntactically correct, are semantically vague, combine multiple potential issues, or lack the specificity required for granular NLP analysis (e.g., pinpointing a single root cause).

* **Motivation:** Over-generalized messages reduce the actionable insights derivable from flight logs. For tasks like problem identification or event recognition, it's crucial to either refine these messages into more specific statements or to acknowledge their inherent ambiguity and document potential interpretations. This process aims to enhance the utility of the dataset for models seeking precise classifications.
* **Procedure:**
    1.  **Expert Review & Contextual Analysis:** Messages identified as over-generalized underwent rigorous review by domain experts. This involved consulting official drone documentation, flight manuals, and aviation safety reports (FAA, NTSB, INTERPOL) to understand the full spectrum of conditions that might trigger such messages.
    2.  **Disambiguation Strategies:**
        * **Contextual Refinement:** When surrounding log messages, telemetry data (not part of this NLP dataset), or known flight conditions provided sufficient evidence, the message was rephrased to reflect a more specific event or issue.
        * **Documentation of Possibilities:** For cases where a definitive single interpretation was not possible without external data, the message was documented with a detailed explanation of its potential meanings or the various scenarios it might cover. This helps annotators and future model developers understand the inherent ambiguity.
        * **Decomposition (Future Work):** In some complex cases, an over-generalized message might be decomposed into multiple, distinct events or issues. This is a more advanced form of refinement and is considered for future iterations of the dataset.

* **Case Studies: Discussion of Over-Generalized Messages**

    Here, we delve into specific examples of over-generalized messages, discussing their challenges and our approach to handling them in the dataset.

    #### **Case 1: Message ID 34**

    * **Original Message:** `Detected side/backward/forward shock / possible collision, aircraft is pitching/rolling sharply forward/backwards/left/right`

    * **Analysis & Approach:**
        This message is a prime example of an alert designed to cover a broad range of potential impact or stability issues. The use of multiple slash-separated terms (`side/backward/forward shock`, `pitching/rolling sharply forward/backwards/left/right`) indicates a generic warning that could be triggered by various events, from a minor bump to a significant collision, or even extreme turbulence.

        * **Possibilities of Actual Events:**
            * **Minor Collision/Contact:** The drone might have briefly contacted a tree branch, power line, or other obstacle without severe damage.
            * **Hard Landing/Rough Touchdown:** A less-than-smooth landing could trigger "shock" or "pitching/rolling sharply."
            * **Extreme Turbulence/Wind Shear:** Strong, sudden gusts of wind could cause the aircraft to "pitch/roll sharply" or experience a "shock" without any physical contact.
            * **Gimbal/Camera Impact:** If the camera or gimbal hit something, it might be interpreted as a "shock" to the aircraft.
            * **Internal Component Shift/Malfunction:** Less likely, but a sudden shift of an internal battery or a mechanical failure could produce similar readings.
        * **Dataset Handling:** For annotation purposes, such messages are typically tagged as a `Warning` type with `Collision/Impact Risk` as a broad problem category. However, in cases where context from surrounding messages or flight telemetry *definitively* indicated a specific type of collision (e.g., immediate follow-up messages about sensor damage), a more specific annotation might be applied. Where specific disambiguation is not possible, the annotation reflects the generalized nature of the warning.

    #### **Case 2: Message ID 204**

    * **Original Message (after Type 1 cleansing):** `Ambient Light too Weak. Forward/Backward Obstacle Sensing is unavailable. Please ensure safety during the flight.`

    * **Analysis & Approach:**
        While the punctuation is corrected, the core issue of "Forward/Backward Obstacle Sensing is unavailable" remains somewhat generalized. It implies a functional failure, but the underlying *reason* for unavailability might vary beyond just "Ambient Light too Weak."

        * **Possibilities of Actual Causes for Sensing Unavailability (beyond just light):**
            * **Sensor Obstruction:** The obstacle sensing cameras/sensors could be physically blocked (e.g., by dirt, moisture, sticker).
            * **Software Glitch/Bug:** A temporary software issue might render the sensors inoperative.
            * **Hardware Malfunction:** A sensor component itself could have failed.
            * **Environmental Factors:** While "Ambient Light too Weak" is given, other environmental factors like heavy fog, smoke, or excessive glare could also contribute.
            * **Flight Mode Limitation:** Certain flight modes (e.g., Sport mode on some DJI drones) intentionally disable obstacle sensing.
        * **Dataset Handling:** In our annotated dataset, the primary problem identified is `Sensor Unavailability` or `Environmental Limitation` (due to ambient light). While the message itself is generalized, the immediate preceding or following messages often provide more context (e.g., specific sensor error codes) that allow for a more precise `Problem Identification` annotation. If no further context is available, the annotation reflects the directly stated warning.

### 3.3. Other Cleansing Considerations (Briefly Mention)

* **Removal of Redundant/Repetitive Messages:** Consecutive identical messages often provide no new information and are consolidated or removed to streamline the dataset.
* **Anonymization:** Any potentially personally identifiable information or highly sensitive flight details are carefully anonymized or removed to ensure privacy.
* **Normalization:** Consistent formatting of numerical values (e.g., temperatures, voltages) and units of measurement.

---

## 4. Documentation of All Corrected Messages

A comprehensive log of all messages that underwent cleansing, showing their original and corrected forms, along with the specific type of correction applied and a brief justification, is provided for full transparency.

* **Download the Full Cleansing Log:**
    This detailed log is available as a CSV or JSON file on our Zenodo dataset. It serves as an exhaustive record of all modifications.

    ```bash
    # Example CLI command to download the cleansing log (replace DOI)
    zenodo_get --file "cleansing_log_v1.0.csv" [YOUR_ZENODO_DOI_NUMBER_HERE]
    ```
    * **File Format:** The log is typically in `CSV` or `JSONL` format, with columns such as `MessageID`, `OriginalMessage`, `CorrectedMessage`, `CorrectionType`, `Justification`, etc.

* **Sample from the Cleansing Log:**
    Here's a small illustrative sample of what you'll find in the full cleansing log:

    | Message ID | Original Message                                                                     | Corrected Message                                                                          | Correction Type        | Justification                                     |
    | :--------- | :----------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------- | :--------------------- | :------------------------------------------------ |
    | 165        | `Ambient Light too Weak, Vision Positioningobstacle sensing is unavailable. please ensure safety during the flight` | `Ambient Light too Weak. Vision Positioning/Obstacle Sensing is unavailable. Please ensure safety during the flight.` | Type 1                 | Punctuation, capitalization, space, slash added.  |
    | 204        | `Ambient Light too Weak, Forward/Backwardobstacle sensing is unavailable. please ensure safety during the flight` | `Ambient Light too Weak. Forward/Backward Obstacle Sensing is unavailable. Please ensure safety during the flight.` | Type 1                 | Punctuation, capitalization, space added.         |
    | 34         | `Detected side/backward/forward shock / possible collision, aircraft is pitching/rolling sharply forward/backwards/left/right` | *(No direct "corrected" form for this type of over-generalized message; analysis provided in section 3.2)* | Type 2 (Analysis)      | Documented ambiguity and potential interpretations.|
    | 78         | `Battery voltage lowplease land`                                                     | `Battery voltage low. Please land.`                                                        | Type 1                 | Punctuation, capitalization.                      |
    | 121        | `NoGPS signalweak.`                                                                  | `No GPS signal. Weak.`                                                                     | Type 1                 | Space, punctuation, capitalization.               |
    | ...        | ...                                                                                  | ...                                                                                        | ...                    | ...                                               |

---

This structure for your `cleansing.md` file will make your data cleansing process incredibly transparent and easy for other researchers to understand and replicate. Remember to replace all placeholder text (`[YOUR_ZENODO_DOI_HERE]`, etc.) with your actual project details once they are available.