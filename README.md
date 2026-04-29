# AIoT Course - Project 2+ (Human Activity Recognition Project)

**An End-to-end Artificial Intelligence of Things Project**

## About this Project

* A team of a max of **3 students** is mandatory for each project
* Announcement date: **28 April, 2026** 
* Delivery Date: **25 May, 2026, 23:59**
* Grade: **40%**

## Project Description

In this project, you are prompted to create an end-to-end Artificial 
Intelligence of Things (AIoT) procedure in order to recognize a set of human 
movements automatically. This problem is identified as Human Activity 
Activity (HAR), which is actually the technology that uses sensors to read 
and interpret body movements as commands or labeled actions.

### The [PAMAP2 Dataset](https://archive.ics.uci.edu/dataset/231/pamap2+physical+activity+monitoring)

**The wearable hardware is not available for this offering of the course.** 
Instead of collecting your own data, you will work with the publicly 
available **PAMAP2 Physical Activity Monitoring** dataset [1], which 
contains IMU recordings (accelerometer, gyroscope, magnetometer) [2] sampled at 
**100 Hz** from 9 subjects performing 18 physical activities. The dataset 
provides three IMUs per subject (placed on the **hand/wrist**, **chest**, 
and **ankle**).

You should treat PAMAP2 as if you had collected it yourselves: parse it, 
ingest it into MongoDB, fetch it back for analysis, and run the full ML 
pipeline on it.

After ingesting the data, you will proceed with data engineering and 
preparation methodologies in order to transform the data into a suitable 
format capable of training the AI models. Then, you will select between a set 
of supervised-learning Machine Learning models for the learning procedure, 
and finally you will evaluate the AI models with respect to their performance.

You will train the ML models using two approaches:
1. Feed the ML algorithms with segmented, windowed data.
2. Perform feature engineering by generating a set of features, selecting (and justifying) the most suitable ones, and then training the ML models using these features.

Finally, compare the results obtained by training on the full set of extracted features versus the selected subset.

It is suggested to read the whole papers that are provided in the "References" 
section in order to better understand the identification scenario.

## Activities

**For the learning scenario, you will use the activities recorded in PAMAP2.** 
The dataset defines the following classes (the integer is the activity ID 
stored in the `.dat` files):

**Protocol activities** (all 9 subjects performed these — `PAMAP2_Dataset/Protocol/`):

| ID | Activity            | ID | Activity            |
|----|---------------------|----|---------------------|
| 1  | lying               | 7  | Nordic walking      |
| 2  | sitting             | 12 | ascending stairs    |
| 3  | standing            | 13 | descending stairs   |
| 4  | walking             | 16 | vacuum cleaning     |
| 5  | running             | 17 | ironing             |
| 6  | cycling             | 24 | rope jumping        |

**Optional activities** (a subset of subjects — `PAMAP2_Dataset/Optional/`):

| ID | Activity        | ID | Activity        |
|----|-----------------|----|-----------------|
| 9  | watching TV     | 18 | folding laundry |
| 10 | computer work   | 19 | house cleaning  |
| 11 | car driving     | 20 | playing soccer  |

Activity ID `0` corresponds to *transient* periods between activities and 
must be **filtered out** before any analysis.

**Use every activity available** to you in the chosen split (Protocol, and 
Optional where it is present). Inspect how many instances each class 
contributes after windowing — class imbalance is part of the analysis you 
will need to interpret in your report.

## Sensor Selection Strategy

Each PAMAP2 subject wears three IMUs (hand/wrist, chest, ankle). Each IMU 
exposes a 3-axis accelerometer (use the ±16g full-scale readings), a 3-axis 
gyroscope, a 3-axis magnetometer, and a temperature reading. There is also a 
heart-rate channel — **do not use biometric channels (e.g. heart rate); this 
project is restricted to inertial sensing.**

You will explore sensor configurations incrementally:

1. **Start with the hand/wrist IMU only**, using just the accelerometer and 
   gyroscope (6 axes total). This mirrors the original wrist-worn scenario.
2. **If the wrist-only configuration is not sufficient** to reach acceptable 
   classification performance, expand to the chest and/or ankle IMUs and 
   **compare** the results. Report the per-configuration metrics and 
   discuss what each additional sensor contributes.

The hand IMU is sampled at 100 Hz. If you choose to add the magnetometer or any channel sampled at 
a different rate, **downsample** to 100 Hz so the per-window feature 
dimensionality stays consistent.

## Configuration

As the majority of software projects work with configuration files to set up 
the instantiation of their components or experiments, we follow the same 
principle in this project too.

Thus, we provide you with a configuration file as an indicator of how to set up
your project. This file should contain all the parameters set up that are 
utilized for the project. To use this file, please copy and rename the 
`config.yml.template`, which is located in the root directory, to `config.yml`.

Additionally, the file should comprise all parameters regarding the MongoDB 
host, the database name, the collection name, as well as parameters about the 
data engineering and the learning processes.

## Dataset Ingestion

The PAMAP2 dataset is shipped with this repository under 
`PAMAP2_Dataset/`. The data flow you must implement is:

```
.dat files  ──parse──▶  MongoDB documents  ──load──▶  MongoDB  ──fetch──▶  analysis notebooks
```

1. Follow the folder and files structure guideline, which can be found 
   in `data/README.md`.
2. Read the dataset documentation (`PAMAP2_Dataset/readme.pdf`, 
   `DataCollectionProtocol.pdf`, `DescriptionOfActivities.pdf`, 
   `subjectInformation.pdf`, `PerformedActivitiesSummary.pdf`) so you 
   understand the column layout and the recording protocol.
3. Instantiate a `mongod` primary daemon process for the MongoDB system.
4. Based on the instructions in `aiot_dataset_creation_sample.ipynb`:
   * Parse each `.dat` file into per-activity, per-subject segments. 
     Discard any rows whose activity ID is `0` (transient).
   * Replace `NaN` values produced by the dataset's missing-sample marker 
     according to a strategy you justify (interpolation, drop, etc.).
   * Transform the segments into the MongoDB document schema described in 
     `data/README.md`.
   * Create the database and the collection where you will save your data.
   * Upload the documents to your collection.
5. From your analysis notebooks, **fetch** the documents back from MongoDB 
   rather than re-reading the raw `.dat` files — the rest of the pipeline 
   should treat MongoDB as the single source of truth.

* The sampling frequency (sampling rate) for the working dataset must be 
  **100 Hz**. Downsample any channel that is recorded at a different rate.

Note: Use the MongoDB Compass GUI to check your database and collections:

* https://www.mongodb.com/products/tools/compass
    
## Exploratory Data Analysis and Data Processing

As it was described in the "Project Description" section, two different ML
solutions will be provided:
* feed the ML algorithms with segmented-windowed data. To help you with the 
process, you can study the publication [7] in the "References" section. 
* generate a set of features (feature engineering), and then feed the
ML models. Accordingly, you can study publication [4] as a guide.

Thus, you will create two different Jupyter Notebooks that correspond to each 
ML solution:

1. `aiot_project_time_series.ipynb`
2. `aiot_project_feature_engineering.ipynb`

Check, for example, the `aiot_project_sample.ipynb`.

### Data Exploration

For the Exploratory Data Analysis (EDA) and the Data Engineering process for 
both a single instance and the whole dataset, please follow the instructions 
below. In particular, according to this step, you are prompted to implement 
and run the following steps (the order is indicative):

* Provide a barplot that contains the time-length of the collected instances 
for each class.
* Time-series segmentation: Split the data into fixed windows of X seconds 
with X% overlap (in samples). 
* Provide a barplot with the count of instances that occurred after this 
process, for each class. 
* Filter the data with a low-pass filter at a frequency of *X* Hz.
* Visualize a time-series instance of the transformed dataset to see the 
effect of the filter on the signal. 
* Detect outliers and/or find missing values (PAMAP2 includes `NaN` markers 
  for dropped wireless packets — handle these explicitly).

### Data Transformation

* Provide the input vector of the segmented windows.
* Provide the input vector after feature engineering. Which features are 
discriminative and useful for the model? 
* For the extracted feature set, determine which features to keep (feature selection).
* Perform dimensionality reduction (if needed).


### Data Preparation

1. Split the data **by subject** into train and test sets, using a 
   **6–7 subjects for training and 2–3 subjects for testing** scheme. The 
   split must be subject-disjoint — no subject may appear in both sets — so 
   the evaluation reflects generalization to new users.
2. Use a scaling algorithm to scale the data into a standard value range 
(for instance, Standardization, Min Max Normalization). Exploit Python's 
`scikit-learn` library. Fit the scaler **only on the training subjects** 
and apply it to the test subjects.

## Learning Process (ML Modeling)

1. Select a supervised Machine Learning approach to perform the activity 
recognition scenario, by exploiting Python's `scikit-learn` library. For 
instance, you can use the Support Vector Machine (SVM) algorithm, Random Forests, 
or other models.
2. Fit the data into the model. 
3. Evaluate the ML model performance in the form of Confusion Matrix and 
Classification Report by using the evaluation metrics that arise from the 
True Positives, False Positives, True Negatives, False Negatives 
classification results. 
4. Explore and understand the data adequately and provide your insights.
5. Fine-tune the ML model (e.g., use Exhaustive Grid Search algorithm). Exploit 
Python's `scikit-learn` library.
6. **Repeat the evaluation across the sensor configurations described in 
   "Sensor Selection Strategy"** (wrist-only first; expand to chest/ankle if 
   wrist-only is insufficient) and discuss the trade-offs.

## Report

**You will provide us with a link that leads to a downloadable `.zip` file
(e.g., Google Drive or OneDrive) that will contain**:

* The **code** of the project.
* The **MongoDB export** of your processed dataset (e.g., a `mongodump` 
  archive) so we can reproduce the analysis. The raw PAMAP2 `.dat` files do 
  **not** need to be re-included — they are public and already available in 
  this repository.
* A **2-page report** related to the dataset and its activities, the 
  preprocessing decisions you made (handling of `NaN` values, downsampling, 
  filtering, windowing), the sensor configurations you compared, how you 
  trained the AI model, and an interpretation of the evaluation results.

Some additional useful information about the report is provided below.

Code Information:
* The code used to parse PAMAP2 `.dat` files, ingest them into MongoDB, and 
  convert the documents into a format capable of training the models.
* The code used to train and evaluate the models. 
* The high-level code could be in notebooks, while the functions could be in scripting 
format.
* `README.md` file with instructions on how to automatically run the code 
should be delivered: data ingestion, data processing, model training, and 
model evaluation process.
* The `README.md` should describe clearly which subjects, activities, and 
  sensor configurations were used.
* The code should be well-documented and commented.
* Provide your results of the training and data exploration using `.png` files.

Documentation:
* A short documentation, max of 2 pages, in `.docx` format.
* For **all** the members of your team, you should include the following: First Name, 
Last Name, email, and phone number.
* Describe the process that you took to convert the data and generate the model. 
Include your observations on the accuracy of the model regarding the classes’ 
identification. 
* **Interpret the results!!**: 
   * You should include the main information in the 
Notebooks and at a more abstract level in the report document (`.docx` file - 
see the instructions below).
  * Compare and explain the differences in the performance metrics between the 
  different ML solutions **and between the sensor configurations**.
* If the `.docx` file is not possible, you can use a `PDF` file.
* The documentation should be in English and should be written in a 
professional manner.
* The documentation should be in a single file, not in multiple files.
* If the `.docx` file is more than 1 page, you will lose points.

## Technologies

The following list provides all the necessary Python packages that can be 
exploited for the project's needs:

* Data Engineering: NumPy, SciPy, pandas 
* Data preparation: scikit-learn
* Visualization: Matplotlib, seaborn, pandas
* AI modeling: scikit-learn 

An indicative Python environment setup can be found in the 
`requirements.txt` file in the root directory.

If you need some extra knowledge of how to utilize Pandas, NumPy, MatplotLib, 
seaborn, scikit-learn, and other Data Science stuff, you can read and 
experiment with the tutorials that can be found here:

* [Python Data Science and Machine Learning Tutorials](https://github.com/tzamalisp/data-science-and-machine-learning-tutorials)
* [Human Activity / Gesture Recognition Tutorials](https://machinelearningmastery.com/how-to-load-and-explore-a-standard-human-activity-recognition-problem/)

## Important Notes (READ THIS CAREFULLY)

* You can write your own functions in the `utils.py` or `utils_visual.py` 
or modify the existing ones. The code in these files is indicative. 
* If any changes take place in these two files, keep in mind to write down the 
proper documentation, as well as the docstrings in the same Python Style 
Format (Google) that is included in the project announcement.
* The high-level project's code (function imports) should run in the Jupyter 
Notebook.
* You can select and deploy any of the ML models that are presented 
in the literature section.
* The project should be implemented in Python 3.11 or higher.
* **Modify** the `requirements.txt` file to contain the latest versions of the 
libraries you use and disable compatibility issues between them.

## References

[1] Reiss, Attila, and Didier Stricker. "Introducing a new benchmarked dataset for activity monitoring." In 2012 16th 
International Symposium on Wearable Computers, pp. 108-109. IEEE, 2012. Dataset: 
https://archive.ics.uci.edu/dataset/231/pamap2+physical+activity+monitoring

[2] Colibri, "A Wireless Micro Inertial Measurement Unit (IMU)", [Online]. Available: [https://mbientlab.com/store/wristband-sensor-research-kit/.](https://www.scribd.com/document/362775494/Colibri-Wireless-IMU)

[3] Tzamalis, Pantelis, Sotiris Nikoletseas, and Paul G. Spirakis. "Gestureset: Public domain dataset for human gesture 
recognition using wrist-worn devices: A preliminary version." In 2023 19th International Conference on Distributed 
Computing in Smart Systems and the Internet of Things (DCOSS-IoT), pp. 404-413. IEEE, 2023.

[4] Aggelides, Xenophon, Andreas Bardoutsos, Sotiris Nikoletseas, Nikos Papadopoulos, Christoforos Raptopoulos, and 
Pantelis Tzamalis. "A gesture recognition approach to classifying allergic rhinitis gestures using wrist-worn devices: 
a multidisciplinary case study." In 2020 16th International Conference on Distributed Computing in Sensor Systems (DCOSS), 
pp. 1-10. IEEE, 2020.

[5] Iyer, Darshan, Fahim Mohammad, Yuan Guo, Ebrahim Al Safadi, Benjamin J. Smiley, Zhiqiang Liang, and Nilesh K. 
Jain., "Generalized hand gesture recognition for wearable devices in IoT: Application and implementation challenges.," 
in Machine Learning and Data Mining in Pattern Recognition: 12th International Conference, MLDM 2016, New York, NY, 
USA, 2016. 

[6] Xu, Chao, Parth H. Pathak, and Prasant Mohapatra., "Finger-writing with smartwatch: A case for finger and hand 
gesture recognition using smartwatch," in 16th International Workshop on Mobile Computing Systems and Applications, 2015. 

[7] Liu, Fang-Ting, Yong-Ting Wang, and Hsi-Pin Ma., "Gesture recognition with wearable 9-axis sensors.," in 
International Conference on Communications (ICC), 2017.

[8] Zhu, Peide, Hao Zhou, Shumin Cao, Panlong Yang, and Shuangshuang Xue., "Control with gestures: A hand gesture 
recognition system using off-the-shelf smartwatch.," in IEEE, 4th International Conference on Big Data Computing and 
Communications (BIGCOM), 2018. 

[9] Tzamalis, Pantelis, Andreas Bardoutsos, Dimitris Markantonatos, Christoforos Raptopoulos, Sotiris Nikoletseas, 
Xenophon Aggelides, and Nikos Papadopoulos., "End-to-end Gesture Recognition Framework for the Identification of 
Allergic Rhinitis Symptoms.," in 2022 18th International Conference on Distributed Computing in Sensor Systems (DCOSS), 
Marina del Rey, Los Angeles, CA, USA, 2022. 

[10] Tzamalis, Pantelis, "Python Data Science and Machine Learning Tutorials", [Online]. 
Available: https://github.com/tzamalisp/data-science-and-machine-learning-tutorials


## Contact

Dr. Pantelis Tzamalis, *Engineering Manager*
* email: [tzamalis@ceid.upatras.gr](mailto:tzamalis@ceid.upatras.gr)
* GitHub: [https://github.com/tzamalisp](https://github.com/tzamalisp)
* Website: [https://tzamalisp.github.io)](https://tzamalisp.github.io)
* Social: [https://www.linkedin.com/in/pantelis-tzamalis/](https://www.linkedin.com/in/pantelis-tzamalis/)

George Kontogiannis, *Ph.D. Candidate*
* email: [george.k.kontogiannis@gmail.com](mailto:george.k.kontogiannis@gmail.com)
* GitHub: [https://github.com/kontogiannisg](https://github.com/kontogiannisg)
* Social: [https://www.linkedin.com/in/george-kontogiannis/](https://www.linkedin.com/in/george-kontogiannis/)
