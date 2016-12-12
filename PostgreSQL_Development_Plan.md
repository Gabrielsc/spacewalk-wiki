[Main Project Page](PostgreSqlProject)
# PostgreSQL Project Plan



[Plan Mile Stones and Tracking](PostgreSQL_Tasks_Track)
## Development Process



The identified process to develop the ability for SapceWalk to work on PostgreSQL database as well will be iterative.

The identified iterations are as follows:

 1.  Kick-off
 2.  Planning/Analysis 
 3.  SetUp of Environments
 4.  Development Iterations 1 to 5.
 5. Final
## Kick off

### Objective

 


To create the project plan and mile stones for the project.
### Identified Tasks



  1. Kick off  meeting
  2.  Identify the objectives.
  3. Identify tools and process to be used for communication and tracking of the project.
  4. Create a Team.
### Time Line



December 2008/January 2009
## Planning/Analysis

### Objective




To create the detailed project plan, technical approaches for development and testing of the migration/alternate/additional db source.
### Identified Tasks



 1. Project Plan.
 2. Development and Testing Technical Approach.
 3. Test plan.
 4. Verification of the recommended Technical Approach.
 5. Sign off on the project Plan and approaches.
### Time Line



January 2009
## Iteration 1

### Objective


 

To share and train the Team with knowledge transfer setup of environments and create a sample database to work as a reference.
### Identified Tasks



 1. Knowledge Transfer of the application.
 2. Recommendation for the Development and Test Environments.
 3. Identification of the modules and priority of these modules.
 4. Access to source code repositories of the Team.
 5. Access to the Test cases already existing to the Team - manual and automated.
 6. Creation of the schema -- for Pg
 7. Setup Development Environments.
 8. Setup of Test Environment.
 9. Creation of the sample database for Pg.
 10. Validation of the orafce package to be leveraged.
 11. Creation of the make scripts for the database.
 12. Database access layer in the different application layers
 10. Further refinement of the Project and Test plan.
 11. Verification of sign offs.
### Time Line



February 2009
## Iteration 2

### Objective




To develop and test the identified modules.
### Task Lists



[breakdown](PgIter2Breakdown)

 1. Identified modules (here the modules are placed as the priority is not set as yet)
       1. Web-UI
       2. API
       3. Release Engineering
       4. E-mail regression
       5. Sanity
       6. Proxy
       7. Monitoring
       8. Quick Search
 2. Knowledge Transfer -- Detailed for the identified modules (including Q&A)
 3. Development:
      1. Identify the queries.
      2. Identify the stored procedures/functions/packages
      3. Make the appropriate changes.
      4. Unit and regression Test the same
 4. Testing:
     1. Review existing test cases.
     2. Identify gaps.
     3. Fill the gaps -- and review -> refill -> review and signoff.
     4. Automate and run the test cases.
 5. Identify/Establish Oracle Benchmarks
 6. Create Oracle to Postgres -- migration scripts.
 7. Create Postgres to Oracle -- migration scripts.
 8. Review and sign-off of the above tasks
### Time Line



March/April 2009
## Iteration 4

### Objective




To develop and test the identified modules.
### Task Lists



 1. Identified modules (here the modules are placed as the priority is not set as yet)
       1.  SDC Software
       2. Activation Key
       3. Reactivation Key
       4. Multi Org - RHN
       5. Multi Org - II
       6. Authentication
       7. Virtualization
       8. Kickstart
 2. Knowledge Transfer -- Detailed for the identified modules (including Q&A)
 3. Development:
      1. Identify the queries.
      2. Identify the stored procedures/functions/packages
      3. Make the appropriate changes.
      4. Unit and regression Test the same
 4. Testing:
     1. Review existing test cases.
     2. Identify gaps.
     3. Fill the gaps -- and review -> refill -> review and signoff.
     4. Automate and run the test cases.
 5. Identify/Establish Pg Benchmarks for developed modules
 6. Create Upgrade Scripts between various Spacewalk versions.
 7. Review and sign-off of the above tasks
### Time Line



May 2009
## Iteration 5

### Objective




To develop and test the identified modules.
### Task Lists



 1. Identified modules (here the modules are placed as the priority is not set as yet)
       1.  Solaris
       2. Users
       3. SSM
       4. Satellite Sync & Export
 2. Knowledge Transfer -- Detailed for the identified modules (including Q&A)
 3. Development:
      1. Identify the queries.
      2. Identify the stored procedures/functions/packages
      3. Make the appropriate changes.
      4. Unit and regression Test the same
 4. Testing:
     1. Review existing test cases.
     2. Identify gaps.
     3. Fill the gaps -- and review -> refill -> review and signoff.
     4. Automate and run the test cases.
 5. Identify/Establish Pg Benchmarks for developed modules
 6. Identify performance issues and other outstanding items.
 7. Review and sign-off of the above tasks
### Time Line



June 2009
## Final Iteration

### Objective




To validate all the modules - functionality, performance and total project sign-off.
### Task Lists



 1. Identify and list the list of issues to fix.
 2. Packaging
 2. Complete UAT.
 3. Complete performance Test along with fixes.
 4. Provide guidelines for future projects enhancements - not to break the Pg database option 
 5. Packaging of the application
 5. Review and sign-off of the project
### Time Line



July 2009


