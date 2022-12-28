# Rimsys Technical Interview
Technical Specs for Document Management Service. (Draft)


## Product Requirements
The business problem/roadmap item is the need for a document and task manager. The following is what we know so far:
-The system shall allow a user to add documents
-The system shall version documents
-The system shall allow a user to add tasks to a document
-The system shall allow a user to download a document

## Considerations:
-UI currently with design
-Product started user stories but needs to ensure technical solution before continuing
-Solution needs to consider how integrations with clients document management system will be affected/ handled.
-This roadmap item is critical to a new client launch, which is expected to be completed by the end of January.

## Existing technology/infrastructure stack/skill of developers:
- BE API (PHP / Laravel)
- FE (Nuxt / Vue.js)
- DB (MYSQL, REDIS)
- CLOUD (AWS, Kubernetes)
- CI/CD (Github actions / ArgoCD)

Please document what questions you have, assumptions you would make when coding, technical dependencies and draft a handful of technical stories. Along with the stories, provide a magnitude of effort so we can understand what we can do as a minimum for the January release. And lastly, a diagram if applicable to assist in explaining the above.


# Overview
Rimsys requires a document management system that allows users to upload documents and create tasks for each document.
We also need a path to integrate with external systems. The system will be storing sensative information thus it needs to have built in redundency and adequate security from resources for each client.  Isolation will be important as the nature of the documents are confidential, thus the highest degree of separation will be enforced.

## Definitions
- Rimsys Document Management Service (RDMS): Is the collection of applications and infrastructure that is used for client users to upload, download and manage their documents. 
- External Document Management Service (EDMS): Is a client operated document management service that is external to RDMS.
- rdms-web: the handle for the SPA web application.
- rdms-api: the handle for the API backend application.
- MUST: The specification is mission critical and has to be included no matter what.
- MAY: the specification is not mission critical and is presented as a guideline.


## Assumptions
- Applications are already being deployed into existing kubernetes cluster into multiple availability zones for reliability and resiliance.
- Database infrastructure is already running in multi-az aurora cluster for high performance and scalability.
- Infrastrucutre is in place already to execute the CI/CD with flexible infrastructure as code.
- RimSys already has a support application that can receive notifications from client installs.
  -  This assumption is based on that we want this project to be completed tested before EOM in january and we don't yet have requirements.
-  AWS as a skill is vague, so I'll assume that the team does not have experience with advanced aws features.
  -  If this turns out to be a false assumption, I would reccomend the stateless option.

## User Stories
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


## Use Cases
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
  - rdms-web sends request to rdms-api for a presigned-url
  - rdms-api authorizes users access token and generates a presigned-url using a uuid as the key.
  - rdms-api saves the s3 key in a record in the database, along with upload information.
  - rdms-web gets presigned url and breaks up file into 2mb chunks and starts uploading to S3.

```


# Design Description
Basic design of this system is to have two different service groups, file management service and the document management service.  The file management service is used for uploading, downloading and helping the ui manage multi-part uploads to S3.   The document management service is responsible for keeping track of document versions, managing tasks on documents.  The RDMS will contain sensative information so security on access needs to be tight and controlled.

Diagram: Contains both design considerations: https://lucid.app/documents/view/eb7ffc33-130d-48d4-bd1a-5c060f8a53bc

There are two design considerations: 
- Stateless
- Stateful

## Stateless (Not Reccomended)
  This option has more moving parts and utilizes aws cognito and oauth 2.0. specification.
  Authentication is handled via cognito hosted ui.
  Cloudfront will be used to complete the oauth flow and set the cookie for the access token.
  
Stateless design would be a large effort.  Mainly because of the advaned cloudfront and cognito setup.  It falls outside the teams core compentency, thus it poses much greater risk of missing our deadline.
  
## Stateful (reccomended)
  This option has less moving parts, but leaves more the user authentication to the applications.
  Each pod will need access to the same user databases and redis cache to stay in sync with one another concerning sessionId's and the logged in user.
  This option can eventually be upgraded to a stateless system.

Stateful Design would be a medium effort, but would focus on teams core attributes within the already existing stacks.
  
### Security

#### AWS Cognito (Stateless)
- User authentication MUST be handled via AWS Cognito.
- A new AWS Cognito user pool MUST BE created.
- AWS Cognito configuration MAY include customers existing identity provider.
- WAF MUST be implemented by configuring an Amazon WAF with the Cognito User pool.
- All traffic must be on TLS (HTTPS)

#### AWS WAF
- WAF MUST be configured to protect cloudfront distribution that is serving the ui application.

#### CORS
- API Gateway must be configured with proper CORS. (Stateless)
- Backend applications need to have CORS configured. (Stateful)
 
### UI Application
- SPA application MUST be deployed to the shared S3 bucket.
- Cloudfront is to be configured to servce the website from the clients application directory.
- Cloudfront functions MUST be used to redirect users without a token to the configured cognito hosted ui. (Stateless)
- Cloudfront functions MUST be used to exchange auth code for access token and store it as a cookie. (Stateless)
- Cloudfront MUST be configured to use strict CORS settings.
- Route 53 will be configured to direct clients custom url to the cloudfront distrobution.

## API
- Fargate ALB will be configured to route API Calls.
- API MUST be served using API Gateway region optimized endpoints (Stateless)
- API MUST be configured with private link and assigned VPC ENI. (Stateless)
- API needs a new group Security Service Group. (Stateful)

## Databases
- Database MUST be hosted in AWS Aurora.
- Each Service Group MUST access to the database.

## Redis (Stateful)
- Each Service Group MUST have access to Redis, so they can stay in sync concerning user authentication.


## Notifications
- S3 Bucket rimsys-document-management-[client-id] MUST send S3 Events to sns.
- Document Management Servie Group MUST be configured to receive S3 event messages.
- SNS MUST have dead letter queue configured and Rimsys Support MUST be a consumer.


# Questions
- Is doing this single tenant a contractual requirement with the client?
- How can we connect to the client's document management system?
- What kind of integration do we need for the client's document management system?
- What kind of scale are we looking at?
- Do we have expectation on usage?
- Is there an existing SLA we need to adhere to?
- What level of isolation is the client expecting?
- Do we have a centralized SSO system we need to integrate with?
- What is the depth of the teams skills concerning AWS?  It is a broad and could be interpreted many different ways.


# Infrastructure
New Domain Registration: custom domain or customer domain (Route 53)
SPA Application : Deployed using AmazonS3 and CloudFront
Applications : 
  - Deployed into existing kubernetes cluster managed by Fargate. 
  - API is exposed via API Gateway. (Stateless)
  - SNS and SQS setup to recieve S3 Events
  - S3 to store documents and trigger events.
  - Fargate ALB - Multiple Service Groups required.
    - File Management Service Group
      - Provides secure pre-signed urls
      - Provides upload chunk data.
      - Allows downloading of files.
      - lost file recovery.
    - Operational Service Group
      - Add task to documents.
      - Update tasks status and description.
      - Manage document versions.
    - Security Service Group (Stateful)
      -  Manages user authentication, forgot password, etc...
