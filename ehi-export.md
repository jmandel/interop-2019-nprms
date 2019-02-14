## Background on ONC's EHI Export requirement

ONC's proposed rule requires that certified EHRs provide support for the
export of a patient's complete electronic health information (EHI). Under ONC's proposal:

* a patient (or authorized representative) can request that an export be performed
* the scope of data is "all the EHI that the health IT system produces and electronically manages" about the patient, from across the EHR's "entire database, including but not limited to clinical, administrative, and claims/billing data"
* results must be "timely" and must not require human intervention (though they may not be immediate) 
* eventually, an export file will be delivered to the patient (or authorized representative)
* export files may be delivered electronically or on physical media such as a CD-ROM
* export files must contain "the entire electronic health record for a single patient"
* export files must be "computable" (i.e., they can't can't rasterize bake the entire record into a PDF)
* export file formats need not be standardized
* export file formats must be fully and publicly documented by the vendor, including "data dictionar" and everything else needed to use the export "without loss of information or its meaning"

## ONC's proposal is spot-on — but has two important gaps

ONC's proposal comes very close to enabling a vibrant ecosystem where third-party
services can benefit from interpreting complete clinical records. It's a strong
complement to the standardized data payloads required for USCDI export. But
for this "full EHI export" to provide a usable consumer experience, it needs to be:

1) **Accessible directly to a patient** (or authorized representative), rather than
being built as a feature for clinic staff (e.g., in a scenario where a patient makes
a request by phoning the medical records department, and departmental staff hits the
"export" button in response to a request -- a friction-laden process)

2) **Exposed end-to-end through an API**, rather than being implemented exclusively as a button
hidden deep within a patient portal experience, or being crippled by the exchange of
physical media like CD-ROMs. This is critical because:
* Portals experiences vary, making features difficult to find and correctly describe
(e.g., if a third party is trying to guide patients toward the export functoinality
in a variety of portals). This was a clear challenge for anyone trying to identify
the "Transmit to a third party" features of a patient portal in the MU2 timeframe.
* Managing physical media takes access out of the realm of modern convenience
and consumer experience.
* Managing files may be challenging on many patient devices (e.g., mobile phones),
and some files may be best suited for off-device storage (e.g., in the cloud). API
connectivity ensures that patients can have a seamless experience for accessing
all of their health data, not just a core data set.

## Proposal

*ONC should require that certified EHRs support full EHI export via patient-accessible API,
even without standardizing the API or the data payloads.*

## Eventually, industry can then drive toward common practices, like...

With these requirements in place, we can work toward a standardized API for managing export
requests, even as the data payloads themselves remain vendor-specific (non-standardized).
For example, we could define an operation, accessible to patient-authorized apps, as follows.

### `GET /Patient/:id/$cures-ehi-export`

This operation builds on the asynchronous export capabilities currently being defined in FHIR.
The response payload, rather than including standardized FHIR resources with the patient's EHR,
simply returns a list of URLs to files in vendor-defined formats (e.g., vendor-specific JSON
formats or CSVs). The completed export result might look like:

```
{
  "transactionTime": "[instant]",
  "request" : "[base]/Patient/123/$cures-ehi-export", 
  "requiresAccessToken" : true,
  "extension": {
    "ehiFiles": [
      "http://server.example.org/patient_file_1.zip",
      "http://server.example.org/patient_file_2.json",
    ]
  },
  "error" : [{
    "type" : "OperationOutcome",
    "url" : "http://serverpath2/err_file_1.ndjson"
  }]
}
```

Note that we use an extension property because the output files don't follow the FHIR async API convention
(ndjson files containing one resoruce per line; all entries within a given file are of a uniform type).
Alternatively, we might avoid using an extension here and instead push the final URLs into `DocumentReference`
resources, like:

```
{
  "transactionTime": "[instant]",
  "request" : "[base]/Patient/123/$cures-ehi-export", 
  "requiresAccessToken" : true,
  "output" : [{
    "type" : "DocumentReference",
    "url" : "http://serverpath2/document_references.ndjson"
  }],
  "error" : [{
    "type" : "OperationOutcome",
    "url" : "http://serverpath2/err_file_1.ndjson"
  }]

}
```

... where the `document_references.ndjson` file would have content like:

```
{"resourceType": "DocumentReference", "content": [{"attachment": {"url": "http://server.example.org/patient_file_1.zip"}}], "status": "current", }
{"resourceType": "DocumentReference", "content": [{"attachment": {"url": "http://server.example.org/patient_file_2.json"}}]. "status": "current", }
```

While it's more verbose, advantages of this approach are:

* donsn't require extension properties in the FHIR async response payload
* scales up to complete database export across all patients, where the number of files might be large enough that listing them all in a single API payload is impractical (specifically: tihs gives the server a way to keep the async status response small, and split URLs across as many ndjson files as needed)
