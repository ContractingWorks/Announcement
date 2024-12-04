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
