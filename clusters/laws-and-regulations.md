## Laws and regulation

### DPA with Google Cloud Platform
/to be filled in

### ROS and PVK

The team needs to update their risk assessment (ROS) and data protection impact assessment (PVK analysis) to migrate to GCP.
Refer to the ROS and PVK section under [Google Cloud Platform clusters](gcp.md).

The following subsystems are compliant and do not need to be analysed by teams, below are the links to their ROS analyses:

* [GCP Lagring av data (Buckets og Postgres)](https://apps.powerapps.com/play/f8517640-ea01-46e2-9c09-be6b05013566?ID=219)
* [GCP Tilgangskontrolloppsett](https://apps.powerapps.com/play/f8517640-ea01-46e2-9c09-be6b05013566?ID=218)
* [Tilgang til Kafka fra GCP](https://apps.powerapps.com/play/f8517640-ea01-46e2-9c09-be6b05013566?ID=229)
* [Google Compute Platform - GCP, og Google Kubenetes Engine - GKE](https://apps.powerapps.com/play/f8517640-ea01-46e2-9c09-be6b05013566?ID=95)
* [Bruk av GCP](https://apps.powerapps.com/play/f8517640-ea01-46e2-9c09-be6b05013566?ID=222)

### Archiving

#### TLDR

Evaluate whether your application´s information storage needs are affected by the paragraphs in The Archival Act, and whether you might need to operate with a separate database physically placed on Norwegian soil (for example NAV´s on-prem data center). If it has archival value, the documents must be stored in systems in Norway for now.

As long as the documents in and out of your applications end up in archives that are stored in JOARK on-premise, a migration to GCP should not pose any problems. There is also an ongoing issue as to whether an in-built archive will be sufficient for our purpose. I.e. the obligatory archive can be sustained in the domain applications, since much of the information there is necessary to archive as well. 

There is a new proposal regarding public cloud infrastructure which hopefully will be ready this autumn as well. This proposal will allow storage on public cloud systems, to a degree. See [New public archive proposal].

#### Background

When moving to GCP or other public cloud providers not offering data centers in Norway, an evaluation of the documentation requirements for the application must be done. There are several important regulations that apply to this documentation, as stated by The Archival Act For Public Archives of 2018 (referred to as The Archival Act in the following).

The requirements are set to ensure documents that contain judicial or important administrative information are stored and made available for future inspection (§1)
NAV is, as a public body and central government agency, obligated to keep archival records to make sure the documents are stored safely as information sources for present and future (§2).

The Public Information Act states that everyone shall have access to case documents to facilitate transparency and as such strenghtens:
- Freedom of expression and information
- Democratic participation
- Security under the law
- Public trust and control

There are three requirements that need to be in place for a document to be considered obligatory for archiving:
- There must exist a case document for the public body
- The document must have been sent from or received by the public body
- The document must be case handled and have value as documentation

Public bodies and agencys are obligated to archive all documents that are created during its handling of business: 
- The documents have value as documentation or are being case handled
- The documents must be archived whether they have been used externally or are created for internal use
- The documents that are created electronically must be made available for The National Archives

The Archival Act states:
A document is defined, by The Archival Act, as a logically limited source of information stored no a media for subsequent reading, listening, viewing or transfer.
An archive is defined as documents that are created during business handling.
Obligatory archiving is defined as storage for documenting the handling.

#### General regulations when migrating to public cloud providers

Without a motion or expressed consent from the National Archivist the records cannot:
- Be moved out of country
- Be deleted
- Be redacted or edited (if it applies to the obligatory documentation)

Documents that requires archival storage must be stored on a media and in a format that fulfills the necessary requirements for durability and accessability (§6). I.e. these documents must be stored 
in a way that ensures authenticity, reliability, integrity and usability.

Rules must be made for deletion of all documentation (§16).

Public bodies are to deliver older and finished archives to an archival depot (§18).

Sourced in large part from  documentation for SalesForce migration to cloud:
https://confluence.adeo.no/display/PTC/Arkivering+og+dokumentasjon+i+Salesforce

All paragraphs of law are from The Archival Act:
[The Archival Act For Public Archives](https://lovdata.no/dokument/SF/forskrift/2017-12-15-2105?q=arkiv)

Archival proposal for public cloud solutions:
[New public archive proposal](https://www.regjeringen.no/no/dokumenter/hoyring--ny-forskrift-om-offentlege-arkiv/id2515364/)