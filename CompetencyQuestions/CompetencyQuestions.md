### Competency Questions and SPARQL Queries
 #### Q1: If a student is studying 4 AS level learning aims over one year starting in September and  withdraws from one of them after 5 weeks, may funding be recorded all year for the withdrawn learning aim?
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student
(COUNT(DISTINCT ?allASLevels) as ?totalASLevels)
?withdrawnAim ?withdrawalDate ?qualifyingDuration
?originalHours ?adjustedHours ?replacementActivity
?fundingBand
(IF(?qualifyingDuration > 5,
"Not_Eligible_Withdrawn_before_qualifying_period",
"Eligible with adjusted hours") as ?fundingStatus)
WHERE {
  # Get student and program
  ?student rdf:type :Student ;
           :enrolledIn ?programme ;
           :hasWithdrawalProcess ?withdrawal .
  
  # Count all AS levels
  ?programme :hasLearningAim ?allASLevels .
  ?allASLevels rdf:type :ASLevel .
  
  # Get withdrawal info for single AS level
  ?withdrawal :withdrawnFrom ?withdrawnAim ;
              :hasWithdrawalDate ?withdrawalDate .
  
  # Get qualifying period and hours
  ?programme :hasQualifyingPeriod ?qualifyingPeriod ;
             :hasOriginalPlannedHours ?originalHours ;
             :hasAdjustedPlannedHours ?adjustedHours ;
             :hasFundingBand ?fundingBand .
  ?qualifyingPeriod :hasQualifyingPeriodDuration ?qualifyingDuration .
  
  # Get replacement activity
  OPTIONAL {
    ?withdrawal :hasReplacementActivity ?replacementActivity
  }
  
  # Filter for specific student
  FILTER(?student = :Student_AS_Example)
}
GROUP BY ?student ?withdrawnAim ?withdrawalDate ?qualifyingDuration
?originalHours ?adjustedHours ?replacementActivity
?fundingBand
```
 #### Q2: When a full time student reduces their programme, at what point do they become a part time student?
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student ?originalHours ?adjustedHours
?qualifyingPeriodDuration ?studentStatus
WHERE {
  # Get student and programme
  ?student rdf:type :Student ;
           :enrolledIn ?programme .
  
  # Get hours to determine full-time status
  ?programme :hasOriginalPlannedHours ?originalHours ;
             :hasAdjustedPlannedHours ?adjustedHours ;
             :hasQualifyingPeriod ?qp .
  
  # Get qualifying period duration if available
  OPTIONAL {
    ?qp :hasQualifyingPeriodDuration ?qualifyingPeriodDuration .
  }
  
  # Determine status based on both qualifying period and hours
  BIND(
    IF(?originalHours >= 540,
       IF(BOUND(?qualifyingPeriodDuration) && ?qualifyingPeriodDuration >= 6,
          "Remains_full_time_for_whole_year_despite_reduction",
          "Status_may_change_before_qualifying_period"),
       "Not_initially_full_time")
    AS ?studentStatus
  )
  
  # Filter for full-time students only
  FILTER(?originalHours >= 540)
}
```

 #### Q3: What dates are used to determine a student’s study programme qualifying period for funding purposes?
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student ?studyProgramme
(MIN(?startDate) AS ?earliestLearningAimStart)
(MAX(?endDate) AS ?latestLearningAimEnd)
?qualifyingPeriodDuration
WHERE {
  # Get student and programme info
  ?student rdf:type :Student ;
           :enrolledIn ?studyProgramme .
  
  # Get learning aims and their dates
  ?studyProgramme :hasLearningAim ?learningAim ;
                  :hasQualifyingPeriod ?qp .
  ?qp :hasQualifyingPeriodDuration ?qualifyingPeriodDuration .
  
  # Get start and end dates
  ?learningAim :hasEarliestStartDate ?startDate ;
               :hasPlannedEndDate ?endDate .
}
GROUP BY ?student ?studyProgramme ?qualifyingPeriodDuration
```
 #### Q4: If a student stops attending class with no notification to the institution, when is the date of withdrawal?

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?student ?learningAim ?lastAttendanceDate ?classRegister
WHERE {
  ?student rdf:type :Student ;
           :enrolledIn ?programme ;
           :hasLastAttendance ?lastAttendance .
  ?programme :hasLearningAim ?learningAim .
  ?lastAttendance rdf:type :LastAttendanceActivity ;
                  :hasDate ?lastAttendanceDate ;
                  :hasClassRegister ?classRegister .
  ?classRegister :correspondsToLearningAim ?learningAim .
  
  FILTER(?student = :Student_AS_Example)
  FILTER NOT EXISTS {
    ?student :hasLastAttendance ?otherAttendance .
    ?otherAttendance :hasDate ?otherDate .
    FILTER(?otherDate > ?lastAttendanceDate)
  }
}
```
 #### Q5: If a student stops attending classes and a member of college staff telephones the student to discuss his or her learning progress, can this be counted as guided learning and be deemed the date of withdrawal?

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?telephoneContact ?isGuidedLearning
?canBeWithdrawalDate ?assistanceType ?description
(EXISTS {?telephoneContact :isCountedAsPlannedHours ?hours} as
?countedForPlannedHours)
WHERE {
  # Binding to specific example
  BIND(:TelephoneContact_Example AS ?telephoneContact)
  
  # Core properties of telephone contact
  ?telephoneContact rdf:type :TelephoneContact ;
                    :isCountedAsGuidedLearning ?isGuidedLearning ;
                    :canBeUsedAsWithdrawalDate ?canBeWithdrawalDate ;
                    :assistanceType ?assistanceType ;
                    :hasDescription ?description .
}
```
 #### Q6: If a student stops attending classes and some time later the student is persuaded to attend the institution to discuss his or her learning attendance, can this be counted as planned hours learning and be deemed the date of withdrawal?

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student ?meeting ?isPlannedHours ?canBeWithdrawalDate
?assistanceType
WHERE {
  # Get student with administrative meeting
  ?student rdf:type :Student ;
           :hasAttendanceRecord ?meeting .
  ?meeting rdf:type :AdministrativeMeeting .
  
  # Check properties of the meeting
  ?meeting :isCountedAsPlannedHours ?isPlannedHours ;
           :canBeUsedAsWithdrawalDate ?canBeWithdrawalDate ;
           :assistanceType ?assistanceType .
  
  # Filter to show it's not counted as planned hours
  FILTER(?isPlannedHours = false)
}
```

 #### Q7: A student on a one-year learning aim stops attending at Easter to revise at home yet turns up and sits the examination in early June. When is the date of withdrawal?

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student ?examActivity ?examDate
(IF(BOUND(?examDate), ?examDate, ?withdrawalDate) AS ?finalWithdrawalDate)
WHERE {
  # Get student with withdrawal process
  ?student rdf:type :Student ;
           :hasWithdrawalProcess ?withdrawal .
  
  # Get recorded withdrawal date
  ?withdrawal :hasWithdrawalDate ?withdrawalDate .
  
  # Get examination details if they exist
  OPTIONAL {
    ?student :hasExaminationResult ?examActivity .
    ?examActivity :hasDate ?examDate .
  }
}
ORDER BY ?student
```
 #### Q8: Is the date of withdrawal for open learning or distance learning provision worked out in the same way as for traditional provision?
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student ?learningMode (MAX(?lastParticipationDate) AS
?correctWithdrawalDate)
(IF(MAX(?lastParticipationDate) = ?withdrawalDate,
"Correct: Withdrawal matches last participation",
"Error: Withdrawal date should be last participation date") AS ?validation)
WHERE {
  # Get student data
  ?student rdf:type :Student ;
           :enrolledIn ?programme ;
           :hasWithdrawalProcess ?withdrawal ;
           :hasLastAttendance ?lastAttendance .
  
  # Get learning mode
  ?programme :hasLearningMode ?mode .
  ?mode rdf:type ?learningMode .
  
  # Get both dates
  ?withdrawal :hasWithdrawalDate ?withdrawalDate .
  ?lastAttendance :hasDate ?lastParticipationDate .
  
  # Filter for learning mode types
  FILTER(?learningMode IN (:OpenLearningProvision, :DistanceLearningProvision,
:TraditionalProvision))
}
GROUP BY ?student ?learningMode ?withdrawalDate
ORDER BY ?student
```
 #### Q9: If a student completes the first year of a 2-year programme and then fails to return for the second year, can the institution record a funding value for the second year?
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student ?programme ?yearOne ?yearTwo ?fundingStatus
(IF(?fundingStatus = :NonFundedYear2Status,
"No, the student did not meet Year 2 start criteria, so funding is not allowed.",
"Further review needed") AS ?fundingValidation)
WHERE {
  # Get student and programme
  ?student rdf:type :Student ;
           :enrolledIn ?programme ;
           :hasStartDate ?yearOne ;
           :hasEndDate ?yearTwo ;
           :relatedToFundingStatus ?fundingStatus ;
           :hasSuccessfulCompletion ?completion .
  
  # Filter for our specific student
  FILTER(?student = :Student_TwoYear)
  
  # Filter for funding status type
  FILTER(?fundingStatus = :NonFundedYear2Status)
}
```

 #### Q10: For traineeships where the student achieves an early progression either into sustainable employment, full time education, other training or an apprenticeship, how should this be treated for funding retention purposes?

  ##### QUERY 1: Action recommendations for early progression
This query provides users with advice about necessary data recording actions related to early progression to adhere to the regulation

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT 
    ?student ?completionStatusAction ?destinationTypeAction
    ?plannedHoursAction
WHERE {
    # Get student enrolled in a traineeship
    ?student rdf:type :Student ;
             :enrolledIn ?traineeship ;
             :hasProgressionActivity ?progression .
    # Traineeship details
    ?traineeship rdf:type :Traineeship ;
                 :hasQualifyingPeriod ?qualifyingPeriod .
    # Qualifying period duration
    ?qualifyingPeriod :hasQualifyingPeriodDuration ?duration .

    # Check status for the traineeship is completed
    OPTIONAL {
        ?student :hasCompletionStatus ?status ;
                 :forProgramme ?traineeship .
        ?status rdf:type :CompletedStatus .
    }

    # Determine if student should be marked as completed
    BIND(IF(BOUND(?status),
           "Already has completion status",
           "Must be marked as COMPLETED")
        AS ?completionStatusAction)

    # Check if progression represents early progression to sustainable outcome
    ?progression rdf:type ?progressionType ;
                 :progressedInWeek ?week ;
                 :progressedFrom ?traineeship ;
                 :progressedTo ?destination .
    FILTER(?week <= ?duration)

    # Check if the progression is not a suitable type
    FILTER(?progressionType NOT IN (
            :ApprenticeshipProgression,
            :FullTimeEducationProgression,
            :VocationalTrainingProgression,
            :SustainableEmploymentProgression
        )
    )

    # Funding eligibility based on destination type
    BIND(IF(BOUND(?progressionType),
           "MAINTAIN student for funding",
           "Student did not progress to a suitable destination for funding.") 
        AS ?destinationTypeAction)

    # Planned hours adjustment based on qualifying threshold
    BIND(IF(?week <= ?duration,
           "REVISE planned hours to actual attendance period",
           "MAINTAIN original planned hours")
        AS ?plannedHoursAction)
}
```

   ##### QUERY 2: Find students complying with the example rule

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?student ?traineeship ?duration ?week ?progressionType ?status 
WHERE {
    # Get student enrolled in a traineeship
    ?student rdf:type :Student ;
             :enrolledIn ?traineeship ;
             :hasProgressionActivity ?progression .

    # Traineeship details
    ?traineeship rdf:type :Traineeship ;
                 :hasQualifyingPeriod ?qualifyingPeriod .

    # Qualifying period duration
    ?qualifyingPeriod :hasQualifyingPeriodDuration ?duration .

    # Check status for the traineeship is completed
    ?student :hasCompletionStatus ?status ;
             :forProgramme ?traineeship .
    ?status rdf:type :CompletedStatus .

    # Check if progression represents early progression to sustainable outcome
    ?progression rdf:type ?progressionType ;
                 :progressedInWeek ?week ;
                 :progressedFrom ?traineeship .

    FILTER(?week <= ?duration)

    # Check if the progression is a suitable type
    FILTER(
        ?progressionType IN (
            :ApprenticeshipProgression, 
            :FullTimeEducationProgression, 
            :VocationalTrainingProgression, 
            :SustainableEmploymentProgression
        )
    )
}
```

#### Q11: If a young person completes their programme during but before the official end of the summer term do the institution still have a legal duty to inform the local authority that the student has left learning?

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student ?learningAim ?completionType ?completionDate
?termEndDate ?localAuthority ?reportingStatus
WHERE {
  # Get student and learning aim
  ?student rdf:type :Student ;
           :enrolledIn ?programme .
  ?programme :hasLearningAim ?learningAim .
  
  # Get summer term dates
  ?term rdf:type :SummerTerm ;
        :hasEndDate ?termEndDate .
  
  # Get completion information through exam or assignment
  {
    # Check exam completion
    ?student :hasExaminationResult ?exam .
    ?exam :hasDate ?completionDate .
    BIND("Exam completion" AS ?completionType)
  }
  UNION
  {
    # Check assignment completion
    ?student :hasAssignmentCompletion ?assignment .
    ?assignment :hasDate ?completionDate .
    BIND("Assignment submission" AS ?completionType)
  }
  
  # Get local authority info if exists
  OPTIONAL {
    ?student :belongsTo ?localAuthority .
    ?localAuthority rdf:type :LocalAuthority .
  }
  
  # Determine reporting requirements
  BIND(
    IF(?completionDate < ?termEndDate,
       IF(?completionType IN ("Exam completion", "Assignment submission"),
          "Normal summer completion - no additional reporting required",
          "Check if reporting needed"),
       "Standard end of term - normal completion")
    AS ?reportingStatus
  )
}
```

#### Q12: If a young person withdraws from just one learning aim in their study programme within the first 6 weeks of their programme during the funding year, does the institution have to change the planned hours?

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student ?learningAim ?originalHours ?adjustedHours
?qualifyingPeriod
WHERE {
  # Get student with withdrawal
  ?student rdf:type :Student ;
           :enrolledIn ?programme ;
           :hasWithdrawalProcess ?withdrawal .
  
  # Get programme details
  ?programme :hasLearningAim ?learningAim ;
             :hasOriginalPlannedHours ?originalHours ;
             :hasAdjustedPlannedHours ?adjustedHours ;
             :hasQualifyingPeriod ?qp .
  
  # Get qualifying period duration
  ?qp :hasQualifyingPeriodDuration ?qualifyingPeriod .
}
```

#### Q13: When does the first six weeks for setting and adjusting planned hours start for students who started their programme in the previous funding year?

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?programme ?fundingYear ?startDate ?endDate ?plannedHours
WHERE {
  # Get programme and funding years
  ?programme rdf:type :StudyProgramme ;
             :hasFundingElement ?fundingYear .
  
  # Get funding year details
  ?fundingYear rdf:type :FundingYear ;
              :hasStartDate ?startDate ;
              :hasEndDate ?endDate .
  
  # Get planned hours for each funding year
  ?fundingYear :hasPlannedHours ?hours .
  ?hours :hasOriginalPlannedHours ?plannedHours .
}
ORDER BY ?startDate
```

#### Q14: When a student continues a programme from the previous year and the first date of attendance is after 1 August (for example, due to summer holiday arrangements), what date is used for calculating the planned hours?

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student ?firstAttendanceDate ?weeklyLimit ?fiveWeekTotal
WHERE {
  # Get student with attendance
  ?student rdf:type :Student ;
           :enrolledIn ?programme .
  
  # Get first attendance date
  ?programme :hasActualAttendance ?attendance .
  ?attendance :hasDate ?firstAttendanceDate .
  
  # Get weekly limit and period total
  ?programme :hasPlannedHours ?hours .
  ?hours :hasWeeklyLimit ?weeklyLimitHours .
  ?hours :hasPeriodTotal ?periodTotalHours .
  ?weeklyLimitHours :hasHoursValue ?weeklyLimit .
  ?periodTotalHours :hasHoursValue ?fiveWeekTotal .
}
```

#### Q15: When a student completes a study programme earlier than planned but after the initial 6 week period to start an Apprenticeship with the same institution are there any circumstances in which the planned hours should be amended?

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student ?studentType ?originalHours ?adjustedHours
?completionDate ?qualifyingPeriod ?institution
WHERE {
  # Get student and study programme
  ?student rdf:type :Student ;
           :enrolledIn ?programme .
  
  # Get programme details including hours and qualifying period
  ?programme :hasOriginalPlannedHours ?originalHours ;
             :hasAdjustedPlannedHours ?adjustedHours ;
             :hasQualifyingPeriod ?qp ;
             :belongsTo ?institution .
  ?qp :hasQualifyingPeriodDuration ?qualifyingPeriod .
  
  # Get completion details
  ?student :hasSuccessfulCompletion ?completion .
  ?completion :hasDate ?completionDate .
  
  # Early completion check
  FILTER(?student = :Student_AS_Example)
  
  # Determine student type based on hours
  BIND(IF(?originalHours >= 540, "FullTime", "PartTime") as ?studentType)
}
```
#### Q16: When a student withdraws from their entire study programme before they meet the funding start criteria do their planned hours need adjustment?
```sparql

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX : <http://www.w3id.org/efro/efro#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?student ?totalAims ?withdrawalDate ?report
?hasWithdrawnFromEntireProgramme ?hasMetFundingCriteria
?hoursAdjustmentStatus
WHERE {
  # Base pattern that works
  ?student rdf:type :Student ;
           :enrolledIn ?studyProgramme .
  
  # Count total aims
  {
    SELECT ?student (COUNT(?aim) as ?totalAims)
    WHERE {
      ?student :enrolledIn ?p .
      ?p :hasLearningAim ?aim
    }
    GROUP BY ?student
  }
  
  # Get withdrawal info
  ?student :hasWithdrawalProcess ?withdrawal .
  ?withdrawal :hasWithdrawalDate ?withdrawalDate .
  
  # Get programme withdrawal and funding status
  ?student :hasWithdrawnFromEntireProgramme
           ?hasWithdrawnFromEntireProgramme ;
           :hasMetFundingCriteria ?hasMetFundingCriteria .
  
  # Get funding report
  ?report rdf:type :FundingClaimReport .
  
  # Determine hours adjustment
  BIND(
    IF(?hasWithdrawnFromEntireProgramme = true,
       IF(?hasMetFundingCriteria = false,
          "No hours adjustment needed - withdrawn before funding criteria met",
          "Hours_adjustment_needed_funding_criteria_met"),
       "Check_individual_withdrawals")
    AS ?hoursAdjustmentStatus
  )
}
```
