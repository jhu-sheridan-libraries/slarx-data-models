# Submission

The top level object encapsulating a logical grouping of data and metadata to be deposited to the data archive. A submission will contain one or more files, with associated file metadata, that a user intends to submit to the data archive. The user must also provide metadata describing the submission as a whole. Much of the submission data will be provided by the user throught the UI


| Field | Type | Description |
| ----- | ---- | ----------- |
| user | string | an opaque string that can be used to uniquely identify the user that created the submission. Provided by the user service |
| status | enum | An overview of the current overall status of the submission [Submission Status](#submission-status) |
| metadataStatus | enum | Status describing the current state of the submission metadata. [Metadata Status](#metadata-status) |
| filesStatus | enum | This status summarizes the state of the files associated with this submission. [Files Status](#files-status) |
| metadata | string | A stringified JSON object with the submission/package level metadata. Must validate against the JSON schema (to be provided) |
| files | List[URI] | A list of (Fedora) [File](file.md) objects |
| accessUrl | URI | A public URL where someone can view this submission. The UI should never write to this field |
| requiredActions | List[URI] | A list of references to [SubmissionActions](submissionAction.md) that must be addressed by the user |
| state |  | State for a draft submission that enables the UI to initialize views nicely. _Only_ the UI should read or write this field |

## Submission Status

A status summarizing the current state of the submission.

| Value | Description |    |
| ----- | ----------- | -- |
| draft | The submission is still user editable. This also means that submission has not yet been deposited in the archive | Submissions will always start with this status. The UI can change a submission's status _to_ `draft` only from `requiresAction` if the user has resolved all [SubmissionActions](submissionAction.md)s in `submission.requiredActions` |
| requiresAction | One or more changes were requested from the honest broker and/or the metadata curator. This status will be set if the honest broker flags one or more files to be changed and/or the metadata curator flags one or more metadata fields | Any automated or manual process that scans files or metadata can change this status to `requiresAction` if it determines that the user must perform an action. For example, an honest broker can flag a file that contains sensitive data, requiring the user to remove the sensitive data or remove the file. When the honest broker flags this file, the submission status must be changed to `requiresAction` |
| complete | The data archive now has custody of the submission's data and materials. The submission must now be read only | This status can only occur if `submission.requiredActions.isEmpty() && submission.metadataStatus === 'approved' && submission.filesStatus === 'approved'`. This status should require user interaction only if the described state is true |
| published | The submission is publicly available through the discovery interface. `accessUrl` should be populated only when the submission has this status | The UI must not set submission status as `published`, as this status only appears after a deposit to the archive is completed and the data is publicly viewable in the discovery interface |

## Metadata Status

| Value | Description |  |
| ----- | ----------- | --- |
| draft | Submission metadata are user editable | Metadata should start in this state. If the user resolves all metadata actions, the metadata status should be changed to `draft` |
| requiresAction | One or more metadata fields have been flagged. The user must approve changes to the metadata or make changes to the metadata. This status should take precedence over other metadata statuses | This status is associated with the existence of one or more [SubmissionActions](submissionAction.md) with type `metadata` |
| approved | The metadata curator has approved the metadata. If a user makes a change to the metadata, the status must change to `draft` | This status must not exist if the submission has one or more [SubmissionActions](submissionAction.md) of type `metadata` |

## Files Status

| Value | Description |
| ----- | ----------- |
| processing | One or more files are being uploaded/copied and/or one or more files are being scanned |
| requiresAction | The honest broker has determined that one or more files must be modified or removed from the submission. This status should take precedence over other files statuses |
| approved | All files associated with this submission have been approved. If the user adds or modifies any files at this point, the status must change to `processing` |