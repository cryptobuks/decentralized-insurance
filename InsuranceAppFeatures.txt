- Real time data tracking to adjust risk. How fast does the person drive. What is their average income etc.
- MVP should be the simplest thing ever. Decentralized bitcoin life insurance pools.

## Decentralized / captive life assurance pools 
  - Manager / Underwriter uploads historical life data 
    - All data is validated before being stored in DB 
    - Manual cleanup would be required to fit the format of our DB Schema
  - Underwriter creates his own questions and answer options
    1) Underwriter uploads actual customer profiles with their actual answers 
    2) Underwriter uploads historical claims data
    3) The app calculates probability of death from actual historical data instead of standard mortality tables. 
    *And I had a lightbulb moment now if I was an underwriter, my questions would be the same as the ACE study!!! now The greater your ACE score, the higher your premium now discount available if you choose one of our panel counsellors*
  - Life logs on and inputs information 
  - App calculates probability of death based on DB data 
  - App spits out premium including profit margin and costs 
  - App uses standard life insurance policy terms for MVP 
    - Policy if customized must be signed on Bitcoin 
  - Money is pooled in a multi sig address 
  - When a claim occurs, a predetermined adjuster investigates validity 
    - All signatories to claim sign off 
    - Claim amount is disbursed. 
    - Life data is updated

## Telematics
* App to measure HRV and pass data on to DB 
* DB measures average HRV over the course of the year to adjust premiums? 
* Maybe even adjust monthly 

## Claims Tracking
* For a future phase once we are income generating 
* Leave most claims processes to be handled off-site 
* For now, underwriter will have to update claims payments on app 

## Types of Users / Roles

### Underwriter 
- Captive Client Corporation
  - Is represented by users tagged as managers 
  - Can manage lives in his pool. 
  - Has edit rights on all lives in his pool. 
- Corporate Insurer / Group of underwriters 
- Individual underwriter 
  - Can be a member of
    - Captive client corporation
    - Insurer 
  - Or can be an independent underwriter

### Authorizer
- Authorizes an underwriter to administer access / funds

### Broker
- Can have relationships with all types of insurers 

### End user 
- Can be 
  - Direct customer of individually underwritten pool 
  - Customer belonging to captive pool 
  - Customer belonging to specific corporate insurer 

### Administrator 
- Can modify everything 

## Life Estimation Variables
 
 - Q&A - Can be added by underwriters (all 1 table)
 	- Age - 0,1
 	- Gender - 0,1
 	- Occupation class - id from occupations table
 	- Location of residence - id from locations table
 	- Predominant Ethnicity - id from ethnicities table
 	- Smoking/Non-Smoking - 0,1 or id from smoking table
 - Special - Needs custom programming (separate tables)
	- ACE Score
	- HRV
## Unit Tests

- User should only be able to access his own pool
- User should not have access to other pools
- Addition of new life at beginning of policy period
- Addition of new life at middle of policy period
- Deletion of life at middle of policy period



