## Cures envisions APIs for "all data"; ONC proposes "a limited set"

The 21st Century Cures Act aims to achieve interoperability with APIs that provide
"access to all data elements of a patient’s electronic health record to the extent
permissible under applicable privacy laws" -- but ONC's proposed APIs are limited
only to a "core" data set, rather than enabling "all data elements":

> strictly for the scope of the API Condition of Certification, we propose to interpret the meaning of the phrase 
> “**all data elements** of a patient’s electronic health record” as follows...
>
> [T]he proposed § 170.315(g)(10) certification criterion...
> would facilitate API access to **a limited set of data elements**...
> 
> Accordingly, for the 
> purposes of meeting this portion of the Cures Act’s API Condition of Certification, we interpret 
> the scope of: the ARCH; its associated implementation specifications; and the policy expressed 
> around the data elements that must be supported by (g)(10)-certified APIs (i.e., FHIR servers) to 
> constitute “all data elements.”

Somehow, ONC has explicitly interpreted "all data elements" to mean "a limited set of data elements".

**ONC's interpretation misses the mark established by Cures, and fortunately there's a way to
remedy the situation with a small tweak to ONC's proposed rule.**

## Background on ONC's (non-API-oriented) EHI Export requirement

Entirely separate from API access, ONC has proposed a certification requirement for the
export of a patient's complete electronic health information (EHI).
Under ONC's proposal:

* a patient (or authorized representative) can request that an export be performed
* the scope of data is "all the EHI that the health IT system produces and electronically manages" about the patient, from across the EHR's "entire database, including but not limited to clinical, administrative, and claims/billing data"
* results must be "timely" and must not require manual intervention (though they may not be immediate) 
* eventually, export file(s) will be delivered to the patient (or authorized representative)
* export files may be delivered to the patient electronically or on physical media such as a CD-ROM (? I think)
* export files must contain "the entire electronic health record for a single patient"
* export files must be "computable" (i.e., they can't can't rasterize the entire record into a PDF)
* export file formats need not be standardized, but must be fully and publicly documented by the vendor, including "data dictionary" and everything else needed to use the export "without loss of information or its meaning"

## To meet the Cures API intent, ONC's EHI Export just needs an API.

ONC's proposal comes very close to enabling a vibrant ecosystem where third-party
services can benefit from interpreting complete clinical records. It's a strong
complement to the standardized data payloads required for USCDI export. But
for this "full EHI export" to provide a usable consumer experience, it must be:

1) **Accessible directly to a patient** (or authorized representative), rather than
being built as a feature for clinic staff (e.g., in a scenario where a patient makes
a request by phoning the medical records department, and departmental staff hits the
"export" button in response to a request -- a friction-laden process)

2) **Exposed end-to-end through an API**, rather than being implemented exclusively as a button
hidden deep within a patient portal experience, or being crippled by the exchange of
physical media like CD-ROMs. This is critical because:
* Portals experiences vary, making features difficult to find and correctly describe
(e.g., if a third party is trying to guide patients toward the export functionality
in a variety of portals). This was a clear challenge for anyone trying to identify
the "Transmit to a third party" features of a patient portal in the MU2 timeframe.
* Managing physical media would take access outside the realm of modern, convenient
consumer experiences
* Managing files may be challenging on many patient devices (e.g., mobile phones),
and some files may be best suited for off-device storage (e.g., in the cloud). API
connectivity ensures that patients can have a seamless experience for accessing
all of their health data, not just a core data set.

## Key Recommendation

*ONC should require that certified EHRs support full EHI export via patient-accessible API,
even without standardizing the API or the data payloads. This will meet the Cures intent
for API access.*

Note that this EHI export API is in addition to the other proposed patient-access API requirements
(i.e., USCDI access using SMART on FHIR).

## Eventually, industry can then drive toward common practices, like...

With these requirements in place, we can work toward a standardized API for managing export
requests, even as the data payloads themselves remain vendor-specific (non-standardized).
For example, we could define an operation, accessible to patient-authorized apps, as follows.

### `GET /Patient/:id/$cures-ehi-export`

This operation builds on the FHIR Bulk Data asynchronous export capabilities currently
being defined (often referred to as "Flat FHIR"). In this use case, the response payload,
in addition to including FHIR resources with the patient's data in the structured FHIR
format when available, would also include FHIR `DocumentReference` resources that embed or
provide urls pointing to files in vendor-defined formats (e.g., vendor-specific JSON
formats or CSV database table snapshots). The `DocumentReference` resources can also
incorporate metadata about each file, such as a link to the corresponding data dictionary
and documentation in the `description` element, a vendor defined identifier for the data
being provided in the `type` element, and the file's data format in the `attachment`
element's `contentType` element. 

The completed export result might look like:

```
{
  "transactionTime": "[instant]",
  "request" : "[base]/Patient/123/$cures-ehi-export", 
  "requiresAccessToken" : true,
  "output" : [{
    "type" : "Patient",
    "url" : "http://serverpath2/patient.ndjson"
  },{
    "type" : "Observation",
    "url" : "http://serverpath2/observations.ndjson"
  },{
    "type" : "DocumentReference",
    "url" : "http://serverpath2/csv_export.ndjson"
  }],
  "error" : [{
    "type" : "OperationOutcome",
    "url" : "http://serverpath2/err_file_1.ndjson"
  }]
}
```

... where the `document_references.ndjson` file would contain items like:

```
{
  "resourceType": "DocumentReference",
  "description": "Demographic information not included in Patient resource, described at http://vendor.com/docs/cures-ehi-demographics.html",
  "type": {
    "coding": [{"system": "http://vendor.com/", "code": "demo-1", "display": "Demographics table export"}]
  },
  "content": [{
    "attachment": {"url": "http://server.example.org/patient_file_1.csv", "contentType": "text/csv"}
  }],
  "status": "current"
}
```
