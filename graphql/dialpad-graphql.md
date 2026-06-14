# Dialpad GraphQL Schema

## Overview

This document describes a conceptual GraphQL schema for the Dialpad API. Dialpad is an AI-powered cloud communications platform offering business phone, contact center, video, and messaging services — notable for Dialpad Ai (real-time transcription, sentiment analysis, and coaching).

The schema is derived from the Dialpad REST API surface documented at https://developers.dialpad.com/reference and models the full communications lifecycle: calls, contacts, users, numbers, SMS, contact centers, AI analysis, conferencing, analytics, and platform configuration.

## Schema Source

- API Documentation: https://developers.dialpad.com/reference
- GitHub Organization: https://github.com/dialpad
- GraphQL Schema File: dialpad-schema.graphql

## Type Categories

### Call Types
Core call lifecycle types covering origination, state, participants, and outcomes.

- **Call** — Top-level call record with direction, status, duration, and participants
- **CallDetails** — Extended metadata for a call including target, external number, and recording info
- **CallDirection** — Enum: INBOUND, OUTBOUND, INTERNAL
- **CallStatus** — Enum: RINGING, CONNECTED, HOLD, VOICEMAIL, MISSED, COMPLETED, CANCELLED
- **CallDuration** — Duration breakdown including total, talk time, and hold time in seconds
- **CallParticipants** — Caller and callee identity objects
- **CallRecording** — Recording metadata: URL, duration, consent status
- **CallTranscript** — Full call transcript associated with a completed call
- **TranscriptSegment** — Individual speaker turn with timestamp, speaker ID, and text

### AI / Conversation Intelligence Types
Dialpad Ai types for real-time and post-call intelligence features.

- **CallAI** — Container for all AI output on a call
- **AISummary** — AI-generated call summary text
- **ActionItem** — Extracted follow-up action with assignee and due date
- **CallMoment** — Detected event during a call (e.g., competitor mention, objection)
- **MomentType** — Enum: COMPETITOR, OBJECTION, PRICING, CUSTOM, SENTIMENT

### Contact Types
Contact management types covering persons, phones, and emails.

- **Contact** — Person or organization record in Dialpad
- **ContactDetails** — Extended contact fields: company, job title, notes
- **ContactPhone** — Phone number entry on a contact with label and type
- **ContactEmail** — Email address entry on a contact
- **ContactType** — Enum: PERSONAL, SHARED, EXTERNAL

### Voicemail Types
Voicemail inbox and transcript types.

- **VoicemailMessage** — Voicemail record with caller, duration, and read status
- **VoicemailDetails** — Extended voicemail metadata including mailbox target
- **VoicemailTranscript** — Text transcript of a voicemail

### SMS / Messaging Types
SMS and MMS conversation and message types.

- **SMS** — Root SMS query/mutation target
- **SMSMessage** — Individual SMS or MMS message with body, media, and status
- **SMSDetails** — Message metadata: sent time, delivery status, channel
- **SMSConversation** — Thread of messages between two parties

### Organization Types
Dialpad organizational hierarchy types.

- **Department** — Organizational department (Office, Group, or Call Center)
- **DepartmentDetails** — Extended department fields: timezone, business hours
- **Operator** — Agent or attendant assigned to handle calls/chats for a department

### User Types
User account and device management types.

- **User** — Dialpad user account record
- **UserDetails** — Extended user profile: email, role, office assignment
- **UserStatus** — Enum: ACTIVE, INACTIVE, SUSPENDED, INVITED
- **UserDevice** — Physical or software device registered to a user

### Line / Number Types
Phone line and number management types.

- **Line** — Logical phone line entity associated with a user or group
- **LineDetails** — Line configuration: forward settings, voicemail, auto-answer
- **LineType** — Enum: PERSONAL, SHARED, MAIN
- **Number** — E.164 phone number record
- **NumberDetails** — Number metadata: country, area code, assignment, type
- **NumberType** — Enum: LOCAL, TOLL_FREE, INTERNATIONAL

### Contact Center Types
Queue and call center management types.

- **CallCenter** — Contact center definition with routing and queue config
- **Queue** — Inbound call queue
- **QueueStatus** — Enum: OPEN, CLOSED, PAUSED
- **QueueStats** — Real-time queue metrics: wait time, calls in queue, agents available
- **HoldMusic** — Hold music configuration attached to a queue or line

### IVR Types
Interactive Voice Response configuration types.

- **IVR** — IVR definition attached to a main line or call center
- **IVRMenu** — Menu node within an IVR tree
- **IVROption** — Key-press option within a menu with destination action

### Conference / Meeting Types
Conference bridge and video meeting types.

- **Conference** — Audio conference bridge definition
- **ConferenceCall** — Active conference call session
- **ConferenceBridge** — Static bridge with dial-in number and PIN
- **Meeting** — Dialpad Meetings (video) session record
- **VideoMeeting** — Video meeting with participant list and recording

### Analytics Types
Reporting and analytics aggregate types.

- **Analytics** — Root analytics query container
- **CallAnalytics** — Call volume and outcome metrics over a date range
- **CallVolume** — Volume breakdown by hour, day, or week
- **CallResolution** — Resolution metrics: handled, abandoned, transferred
- **Disposition** — Post-call disposition code applied by an agent

### Platform / Auth Types
API credentials, OAuth, and webhook management types.

- **DialpadConnect** — Dialpad Connect (embedded dialer) configuration
- **APIKey** — API key credential record
- **OAuthToken** — OAuth 2.0 token with scopes and expiry
- **Webhook** — Webhook subscription with target URL and event filters

## Key Queries

```graphql
call(id: ID!): Call
calls(filter: CallFilter, page: Int, limit: Int): CallConnection
contact(id: ID!): Contact
contacts(search: String, type: ContactType, page: Int, limit: Int): ContactConnection
user(id: ID!): User
users(departmentId: ID, status: UserStatus, page: Int, limit: Int): UserConnection
number(id: ID!): Number
numbers(assigned: Boolean, type: NumberType, page: Int, limit: Int): NumberConnection
smsConversation(id: ID!): SMSConversation
callCenter(id: ID!): CallCenter
callCenters(page: Int, limit: Int): CallCenterConnection
analytics(filter: AnalyticsFilter!): Analytics
webhooks: [Webhook!]!
```

## Key Mutations

```graphql
initiateCall(input: InitiateCallInput!): Call
hangupCall(id: ID!): Call
holdCall(id: ID!, hold: Boolean!): Call
transferCall(input: TransferCallInput!): Call
sendSMS(input: SendSMSInput!): SMSMessage
createContact(input: CreateContactInput!): Contact
updateContact(id: ID!, input: UpdateContactInput!): Contact
deleteContact(id: ID!): Boolean
assignNumber(input: AssignNumberInput!): Number
createWebhook(input: CreateWebhookInput!): Webhook
deleteWebhook(id: ID!): Boolean
```

## Notable Design Decisions

1. All paginated lists use a Connection/Edge pattern (cursor-based pagination).
2. AI types (CallAI, AISummary, ActionItem, CallMoment) are available only on completed calls.
3. Real-time QueueStats are separate from historical CallAnalytics.
4. IVR trees are modeled as recursive IVRMenu nodes linked by IVROption destinations.
5. Conference and VideoMeeting are distinct types reflecting Dialpad's voice/video split.
