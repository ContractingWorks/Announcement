# Contracting Works Integration status

The table below is updated if we have major disruptions in operations.  
If this occurs, a more detailed description is added below.  

| System                      | Short name | Status |
|-----------------------------|------------|--------|
| SpeedyCraft                 | SC         | OK     |
| PowerOffice GO              | POG        | OK     |
| UNI Economy                 | UNI        | OK     |
| Visma Business NXT          | BNXT       | OK     |
| Visma Finacials / Visma.Net | VNET       | OK     |
| Visma Payroll               | VPAY       | OK     |

## 2025-03-24: Increased integration parallelism
As the number of of clients handled by the integrations have increased, we noticed resource usage on our integration services was starting
to be high. We have doubled the number of integration services running, as well as scaled up available memory for the integration services.
This was a planned scaling operation, and we had prepared for it well in advance.

We are seeing reduced queue times for integrations, as well as a reduction in some recoverable error types related to network resources.
In effect, our internal integrations are now faster, in particular in periods with a lot of client activity.


## 2024-12-19: CW API performance improvement
We have deployed a dedicated API node for use by our internal integrations, similar to what SpeedyCraft did earlier.
This gives us better CW API performance for our integrations, improving average sync times. 

The most important effect is however that we see a reduction in the occasional slow API responses,
which we observed in periods with significant load on the API. 

Integrations syncs complete faster, and the CW frontend is a bit more snappy now.

## 2024-12-17: SpeedyCraft performance improvement
__Updated 22:30__: SpeedyCraft has deployed a dedicated API node for CW usage, which should improve SC sync times and improve stability. 
We are monitoring the integration closely to evaluate the effect of the update. It looks really promising so far!

## 2024-11-27: Incorrectly re-opened assignments caused sync delays

> Nightly maintenance processed service lines migrated from Visma Contracting in the beginning of 2024.
> This processing caused a lot of migrated assignments to incorrectly be re-opened.

### Details
We added automated nightly fixes to postprocessing of service lines: If for some reason 
a new service line was not properly postprocessed as part of the initial save operation,
the postprocessing will be performed during the nightly maintenance.

Unfortunately, for Visma Contracting clients migrated in the period 2024-01-02 - 2024-02-29 the flag indicating
postprocessing of a service line is completed was not set as part of the migration. The migrated service lines
were therefore picked up and "fixed" by the new mechanism. In all, 17 clients were affected by this issue.

When a new service lines is postprocessed, this is triggers an automatic opening of the assignment the service
belongs to. A lot of old assignments were therefore re-opened by this.

We restored the affected assignments and service lines to their correct state using the history tracking on the
tables to identify affected rows, re-closing the affected assignments.

Because some of the affected rows had been synced to other systems after the assignments were re-opened,
it was necessary to let the integrations sync corrections as well, causing a lot of correction syncs to 
be performed during normal working hours. For the end users, this effectively caused syncs to be slower than usual.

All related syncs are now completed, and sync times are back to normal.
 


## 2024-11-23: Address structure updates caused sync delays

> For Norwegian addresses, `PostalPlace` is now set according to the registered `PostalCode`.
> This caused an unexpected amount of sync operations to SC, POG, UNI, BNXT and VNET.

### Details
Contracting Works now looks up `PostalPlace` as well as municipality and county information
when registering an address based on the registered `PostalCode`.  

All existing registered addresses were processed to add missing information and correct spelling
mistakes on the postal place where needed.

Full address syncs to SpeedyCraft were needed, but could be skipped when sycing to the ERP systems.

There were unfortunately three different issues working together which caused this change to
adversely impact integration performance:
* The SpeedyCraft integration performed a single batch of address updates only, and did not proceed to the next batch before another sync operation was performed. Syncs did therefore not complete within the expected time during the weekend.
* POG, UNI, BNXT and VNET integrations performed an update-sync on all assignments and customers related to the updated addresses. These syncs could have been skipped.
* A bug in the CW frontend caused the "spinning wheel" indicating syncs are being performed to keep spinning even if syncs were completed. This bug is now fixed.

The cumulative effect was that address syncs expected to complete during the weekend were being performed on Monday / Tuesday the follwing week, causing max sync times to go from a few seconds up to (at most tracked in logs) 20 minutes. The frontend bug made this appear to be even slower.

All address related syncs are now completed, and sync times are back to normal.



## 2024-11-21: BNXT invoice reference changed

> We have switched from using the field `InvoiceReference` to `CustomerOrSupplierOrderNo` in BNXT

### Details
The field `ExternalReference` in CW used to be mapped to BNXT `InvoiceReference`.

It turns out that if there exists an invoice in BNXT with an invoice number matching the `InvoiceReference`,
BXNT will copy rows from the matching invoice into the newly created invoice, which is not intended.  
This behaviour in BNXT started causing issues for several clients recently. 

The field `CustomerOrSupplierOrderNo` has no automation in BNXT, and is actually the field intended for this
reference in BNXT according to the BNXT development team.

### Impacts of the Change
No functional change in CW, but invoice views in BNXT might need to be configured to display the `CustomerOrSupplierOrderNo` field.

### Recommended Action
Ensure that `CustomerOrSupplierOrderNo` is visible when working on invoices from CW in BNXT.

