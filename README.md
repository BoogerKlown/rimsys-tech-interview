# rimsys-tech-interview
Technical Specs for Document Management Service.


# Overview
Rimsys requires a document management system that allows users to upload documents and create tasks for each document.
We also need a path to integrate with external systems. This will be accomplished by use of an asynchronous messaging system and by allowing other document services to integrate with us via api.


# Basic Design
The basic design of this application is to build this service using Laravel Backend and a NuXTJS front end.  Application is to be deployed into AWS Environment and will be backed by AWS Services.  Application will need to scale properly.  API Application will be deployed into a kubernetes cluster.  Application will need to be deployed into a multiple availability zones.

# Definitions
- Rimsys Document Management Service (RDMS): Is the collection of applications and infrastructure that is used for client users to upload, download and manage their documents. 
- External Document Management Service (EDMS): Is a client operated document management service that is external to RDMS.
- rdms-web: the handle for the SPA web application.
- rdms-api: the handle for the API backend application.

# User Stories
(Must Haves)
- As an authenticated client user, I can upload a document to the RDMS.
- As an authenticated client user, I can download documents I have uploaded to the RDMS.
- As an authenticated client user, I can view a list of documents I have uploaded to the RDMS
- As an authenticated client user, I can create tasks for individual documents and keep track of their different statuses (Open, Closed)
- As an authenticated client user, I can store multiple versions of documents I uploaded to RDMS.
- As an authenticated client user, I can view any current or previous versions of documents I have uploaded to RDMS.

(Nice to haves)
- As a client user, I can authenticate my EDMS to RDMS using oauth 2.0 client credentials flow.
- As a connected EDMS Application, I can receive events for when a new version of a document is uploaded to RDMS.
- As a connected EDMS Application, I can receive events for when a task is created for a document in RDMS.
- As a connected EDMS Application, I can receive events for when a task status is changed in RDMS.


# Use Cases
Actor – anyone or anything that performs a behavior (who is using the system)
Stakeholder – someone or something with vested interests in the behavior of the system under discussion (SUD)
Primary Actor – stakeholder who initiates an interaction with the system to achieve a goal
Preconditions – what must be true or happen before and after the use case runs.
Triggers – this is the event that causes the use case to be initiated.
Main success scenarios [Basic Flow] – use case in which nothing goes wrong.
Alternative paths [Alternative Flow] – these paths are a variation on the main theme. These exceptions are what happen when things go wrong at the system level.
```
Use Case: 1 (Display Document List)
Actor: Client User
Precondition: User is authenticated
Trigger: User navigates to the Document List view
Basic Flow:
  - rdms-web sends request to rdms-api to retrieve list of documents.
  - rdms-api authorizes users access token and retrieves list of users document meta data from database.
  - rmds-web recieves the list and renders the document using a list component of document items.
Alternative Flow: 
  - (No uploaded documents)
    - rdms-web receives an empty list and renders No document upload and presents a link to navigate to the upload document view.
 
Use Case:2
Actor: Client User
Precondition: User is authenticated
Trigger: User navigates to the upload document view
Basic Flow:
  - rdms-web sends request to rdms-api to 
  - rdms-api authorizes users access token and generates a presigned-url using a uuid as the key
```

# Assumptions
- Users are managed centrally using AWS Cognito as Identity Provider
  - Reason for assumption is available aws tech stack and declared developer skillsets.
- Client document management system can connect to API's via M2M Token and oauth 2.0 protocal.
- Client document management system is able to receive events via webhooks.
- Applications are already being deployed into existing kubernetes cluster into multiple availability zones for reliability and resiliance.
- Database infrastructure is already running in multi-az aurora cluster for high performance and scalability.


#Infrastructure
New Domain Registration: documents.rimsys.io (Route 53)
SPA Application : Deployed using AmazonS3 and CloudFront
API Application : Deployed into existing kubernetes cluster.




# Business Models
Documents
 - Internal ID
 - External ID
 - Link
 - Name
 - Description
Tasks
 - id
 - Document ID
Users
