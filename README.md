# healthdata-analytics
## Introduction

The project aims to analyze patient data to gain insights into healthcare service utilization, patient satisfaction, and operational efficiency. By examining the dataset, healthcare administrators and stakeholders can identify trends, patterns, and potential areas for improvement within the healthcare system.

### Column Name and Meaning:

- date: Date and time of patient record. It indicates when the patient record was created.
- patient_id: Unique identifier for each patient. It is a unique identifier assigned to each patient.
- patient_gender: Gender of the patient. It represents the gender of the patient (M for male, F for female).
- patient_age: Age of the patient. It indicates the age of the patient in years.
- patient_sat_score: Patient satisfaction score. It represents a score indicating patient satisfaction (null if not available).
- patient_first_initial: First initial of the patient's first name. It denotes the first initial of the patient's first name.
- patient_last_name: Last name of the patient. It represents the last name of the patient.
- patient_race: Race of the patient. It indicates the ethnicity or race of the patient.
- patient_admin_flag: Flag indicating if the patient has administrative privileges. It represents TRUE if the patient has administrative privileges, FALSE otherwise.
- patient_waittime: Time (in minutes) patient waited. It denotes the time in minutes that the patient waited before being attended to.
- department_referral: Department referred for medical assistance. It represents the medical department to which the patient is referred (None if not applicable).

## Data Cleaning

Imported the data from the get data menu then select the type of file like text/csv here in this case and transform to check the data quality. Now to check the column quality go to views and tick column quality to ensure the valid is 100%, error is 0 %, and empty is 0%.

Or replace any column having N/A with 0 to have an error-free data set.

[In case any column data not understood well, can be uploaded to chat GPT for assistance. Like shorten the row length to 10 for that.
From file, go to keep rows, and write 10 in numbers of rows box, and click on OK.

Prompt for that: Create a tabular format data dictionary for the dataset below: copy and paste the shortened dataset.

And that will return the meaning of each column to provide the explanation.]

## Transformation

### A new column was added

The Date column contains both date and time. So, another column called 'Moments' was created where the time zone was indicated, such as AM or PM.

To create the new column, the following steps were taken:

- Go to the add column, then custom column.
- Give a name for the new column.
- Write the formula for this case:

If DateTime.ToText([date],”tt”) = “AM” THEN “AM” else “PM”

[A new column, if AM/PM was already present in the date column, was created by going to add column, then extract, and selecting the last character. The count of characters (e.g., 2) was given, and then the column was renamed by double-clicking on it.]

### A new column for full name was generated

Two columns, Patient first initial and Patient last initial, were combined to create the full name. To do this, the two columns were selected, and from the transform menu, merge columns were chosen based on the separator space. A name was given for the new column.

After making all the changes, the process was completed by going to the home tab and selecting close and apply.

### New table added

Date table added additional columns derived from the date information. It uses the CALENDARAUTO function to automatically generate a calendar table. Then, it adds calculated columns such as "Year" (extracted from the date), "Month" (formatted as a three-letter abbreviation), "WeekType" (categorized as either "Weekend" or "Weekday" based on the day of the week), "Weekday" (formatted as a three-letter abbreviation), and "MonthNum" (extracted numeric value representing the month).

```
Date =
ADDCOLUMNS(
  CALENDARAUTO(),
  "Year", YEAR([Date]),
  "Month", FORMAT([Date],"mmm"),
  "WeekType", IF(WEEKDAY([Date])=1, "Weekend", IF(WEEKDAY([Date])=7, "Weekend", "Weekday")),
  "Weekday", FORMAT([Date], "ddd"),
  "MonthNum", MONTH([Date])
)
```
This table was added to create a new table that contains distinct months from the 'date' table along with the total number of patients for each month.
```
Table =
VAR _PatientTable =
    CALCULATETABLE(
        ADDCOLUMNS(
            VALUES('date'[Month]),  -- Use VALUES to get distinct months directly
            "@Total_Patients", [Total Patients]
        ),
        ALL('date')  -- Use ALL to remove all filters on the 'date' table
    )
RETURN
    _PatientTable
```
This table was created to define two parameters along with their corresponding measures and identifiers (0 for "Avg. Satisfaction" and 1 for "Avg. Wait Time") for use in analysis or reporting tools.
```
Parameter = {
    ("Avg. Satisfaction", NAMEOF('Calculation'[Average Satisfaction]), 0),
    ("Avg. Wait Time", NAMEOF('Calculation'[Average Wait Time]), 1)
}
```
This table created for utilization for parameter selection or configuration within a Power BI report or similar tool, where users can choose between "Average Satisfaction" and "Average Wait Time" measures based on the provided identifiers (0 for "Average Satisfaction" and 1 for "Average Wait Time").
```
Parameter 2 = {
    ("Average Satisfaction", NAMEOF('Calculation'[Average Satisfaction]), 0),
    ("Average Wait Time", NAMEOF('Calculation'[Average Wait Time]), 1)
}
```
### Calculation
Now we will do the calculation part needed for the visualization of the project.

From the report view go to enter data and give name Calculation and click on OK.

New measure added to find the total patient
```
Total Patients = COUNTROWS('HEALTHCARE DATA')
```

New measure added to find the percentage of administrative schedule.

```
% Administrative Schedule = 
FORMAT(
    DIVIDE(
        COUNTROWS(
            FILTER(
                'HEALTHCARE DATA',
                'HEALTHCARE DATA'[patient_admin_flag] = TRUE()
            )
        ),
        [Total Patients]
    ),
    "0.00%"
)
```

New measure added to find the percentage of none - administrative schedule.
```
% None - Administrative Schedule = 
FORMAT(
    DIVIDE(
        COUNTROWS(
            FILTER(
                'HEALTHCARE DATA',
                'HEALTHCARE DATA'[patient_admin_flag] = FALSE()
            )
        ),
        [Total Patients]
    ),
    "0.00%"
)
```

New measure was added to find the average satisfaction score.
```
Average Satisfaction = 
CALCULATE(
    AVERAGE('HEALTHCARE DATA'[patient_sat_score]),
    'HEALTHCARE DATA'[patient_sat_score] <> BLANK()
)

```



### New measure was added to find the percentage of people who didn't give any ratings.To generate % calculation, add the symbol % after writing dax.
```
% No Rating =
VAR No_Ratings =
CALCULATE(
    [Total Patients],
    'HEALTHCARE DATA'[patient_sat_score] = BLANK()
)
RETURN
DIVIDE(
    No_Ratings,
    [Total Patients]
)
```
### New measure was added to find the average wait time 
```
Average Wait Time = AVERAGE('HEALTHCARE DATA'[patient_waittime])
```

### New measure was added to find the percentage of patients who were referred.To generate % calculation, add the symbol % after writing dax.
```
% Referred Patients =
 VAR _FilterPatients =
 CALCULATE(
    [Total Patients],
    'HEALTHCARE DATA'[department_referral] <> "None"
 )
 RETURN
 DIVIDE(
    _FilterPatients,
    [Total Patients]
 )
```

### New measure was added to find the percentage of patients who were referred.To generate % calculation, add the symbol % after writing dax.
```
% Un Referred Patients =
 VAR _FilterPatients =
 CALCULATE(
    [Total Patients],
    'HEALTHCARE DATA'[department_referral] = "None"
 )
 RETURN
 DIVIDE(
    _FilterPatients,
    [Total Patients]
 )
```



### New measure was added to find the percentage of female visitors.To generate % calculation, add the symbol % after writing dax.

```
% Female Visit =
DIVIDE(
    CALCULATE(
        [Total Patients],
        'HEALTHCARE DATA'[patient_gender]= "F"
    ),
    [Total Patients]
)
```


### New measure was added to find the percentage of female visitors.To generate % calculation, add the symbol % after writing dax.
```
% Male Visit =
DIVIDE(
    CALCULATE(
        [Total Patients],
        'HEALTHCARE DATA'[patient_gender]= "M"
    ),
    [Total Patients]
)
```

### New measure was added to find the percentage of unknown visitors.To generate % calculation, add the symbol % after writing dax.

```
Unknown Visit % =
DIVIDE(
    CALCULATE(
        [Total Patients],
        'HEALTHCARE DATA'[patient_gender]= "NC"
    ),
    [Total Patients]
)
```
### New measure was added to identify whether the total number of patients for a given month is the minimum or maximum value across all months, returning 0 for minimum and 1 for maximum.
```
CF Max Point (Month) =
  VAR _PatientTable =
    CALCULATETABLE(
        ADDCOLUMNS(
        SUMMARIZE('Date','Date'[Month]),
        "@Total_Patients",[Total Patients]
        ),
        ALLSELECTED()
  )
  VAR _MinValu = MINX(_PatientTable,[@Total_Patients])
  VAR _MaxValu = MAXX(_PatientTable,[@Total_Patients])
  VAR _TotalPatients = [Total Patients]
  RETURN
  SWITCH(
    TRUE(),
    _TotalPatients = _MinValu, 0,
    _TotalPatients = _MaxValu, 1
  )
```
### New measure added to generate a caption for a heat map visualisation based on the selected measure. If the selected measure is 0, it indicates low wait time on the age-group, and the caption reflects this. Otherwise, it suggests that patients are most satisfied when the scale shows the darkest green on the age-group.
```
HeatMap Caption =
VAR _SelectedMeasure =
SELECTEDVALUE(Parameter[Parameter Order])
RETURN
IF( _SelectedMeasure=0,
"The darkest GREEN on the Scale denotes LOW wait TIME on the Age-Group",
"Patients are most SATISFIED when the SCALE shows the darkest GREEN on the Age-Group"
)
```
### New measure "Age Buckets," is added to categorise patient ages into different age groups (0-10, 11-20, ..., 61-70, 70+) based on the 'HEALTHCARE DATA' table's 'patient_age' column.
```
Age Buckets =
SWITCH(
    TRUE(),
    'HEALTHCARE DATA'[patient_age] <= 10, "0-10",
     'HEALTHCARE DATA'[patient_age] <= 20, "11-20",
      'HEALTHCARE DATA'[patient_age] <= 30, "21-30",
       'HEALTHCARE DATA'[patient_age] <= 40, "31-40",
        'HEALTHCARE DATA'[patient_age] <= 50, "41-50",
         'HEALTHCARE DATA'[patient_age] <= 60, "51-60",
          'HEALTHCARE DATA'[patient_age] <= 70, "61-70",
          "70+"
)
```

### Visualisation 

#### Cards

Cards were created to generate visuals for Average Satisfaction, % No Rating, Average Wait Time, Referred Patients, Walking Patients, Administrative Appointment, None - Administrative Appointment, Heatmap caption, Total Patient, % Male Visit, % Female Visit, and Unknown Visit %.

#### Clustered Bar Chart

Two clustered bar charts were added to visualize the total visits by department referral (putting department_referral on the y-axis and Total Patient on the x-axis) and Total Patient by Age group (putting Age group on the y-axis and Total Patient on the x-axis).

#### Clustered Column Chart

A Clustered Column Chart was added to generate visuals for Patients by Week Type, where WeekType was added to the x-axis and Total Patient was added to the y-axis.

#### Matrix

A Matrix was added to generate heatmap visualization for different patient race-wise age distribution, where patient_race was added to rows and Age Buckets were added to columns. In the values part, Parameter was added. In the series part of cell element, Avg. Satisfaction was added and the background and font color customization were done to dark green and light green from the highest and middle value. The format style was set to gradient and the field was changed to Average Satisfaction. This process was repeated for slicer option Avg. Wait Time.

#### Line Chart

Two line charts were added to generate visuals for Total Patient visit month wise where date and month were added to the x-axis, and Total Patients and CF Max Point(Month) were added to the y-axis. Another line chart was added for generating visuals for Total Patient visit by Year where date and year were added to the x-axis and Total Patients were added to the y-axis.

#### Slicer

In the slicer, Parameter was added and the slicer setting was set in dropdown style for user interaction.

## Insight

The analysis of the healthcare data reveals several key insights. Firstly, administrative appointments substantially outnumber non-administrative ones, suggesting a significant administrative workload within the healthcare system. Moreover, a considerable percentage of patients refrain from providing ratings, indicating potential areas for improving patient feedback mechanisms.

Furthermore, walk-in patients constitute a larger portion compared to referred patients, highlighting the importance of accommodating unplanned visits efficiently. Notably, the majority of patients are adults over 18 years old, emphasizing the need for tailored healthcare services for this demographic.

The analysis also indicates a seasonal trend in patient visits, with the highest activity observed between April and November. Interestingly, there has been a significant increase in patient visits in 2020 compared to the previous year, reflecting evolving healthcare needs or external factors such as the COVID-19 pandemic.

Additionally, patients visiting without department referrals significantly contribute to overall patient visits, suggesting a robust primary care system or accessibility to healthcare services without specialized referrals.

Finally, Native American patients aged 41 to 50 years express the highest average satisfaction levels, indicating areas of success in patient care for this demographic.

In conclusion, these insights underscore the importance of efficient administrative processes, enhancing patient feedback mechanisms, optimizing healthcare access for both walk-in and referred patients, and tailoring services to meet the needs of different demographics for improving overall healthcare service delivery and patient satisfaction.

![AM-avg-satisfaction](https://github.com/roy-deblina/healthcare-data-analytics/assets/164593876/47c3884b-9e03-41db-b29e-a20c5c4f993b)
- Average Satisfaction during AM hours.

![am-wait-time](https://github.com/roy-deblina/healthcare-data-analytics/assets/164593876/7d15a2f3-9d5a-484d-b3bf-1a56ca2488d7)
- Wait Time during AM hours.

![PM-avg-satisfaction](https://github.com/roy-deblina/healthcare-data-analytics/assets/164593876/a6f9f014-0714-4e0e-8e1e-04bc40de2a7e)
- Average Satisfaction during PM hours.

![pm-wait-time](https://github.com/roy-deblina/healthcare-data-analytics/assets/164593876/de0e62ec-b88e-4967-b4bf-d128b41f1153)
- Wait Time during PM hours.




