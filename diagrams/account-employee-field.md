# Account employee eligibility

Account View displays **Employee Y/N**; Account Update lets an operator
change it using the existing Enter → validate → F5 save flow. Lowercase
input is normalized; blank and non-Y/N input are rejected. F12 cancels.
This records eligibility only; it does not alter discount calculations.

`CVACT01Y.cpy` uses the first byte of the former filler (zero-based offset
122) for `ACCT-EMPLOYEE-FLAG`, retaining the 300-byte record length and all
existing field offsets. Legacy blank/low-value flags display as N. Only Y
is treated as employee eligibility.

The account update record now includes the ZIP field present in the shared
copybook and starts from the locked original record, preserving ZIP and
unused bytes. Blank account groups remain spaces rather than being saved
as low-values. Employee changes participate in change detection and the
existing optimistic concurrency check.

## Verification

Both affected COBOL programs compile with GnuCOBOL CICS and both BMS maps
generate with the development environment's bms2json tool.

Runtime terminal checks with bundled demo data:

- Legacy account flag displays N.
- Lowercase y becomes Y, requires F5 confirmation, persists, and appears
  as Y after returning through Account View.
- X displays “Employee must be Y or N” and does not change the saved flag.
- Lowercase n becomes N and saves successfully.
- Final byte-for-byte check confirms an employee-only save changes byte
  122 and preserves all other 299 bytes, including ZIP, group, and filler.

The bundled customer 000000001 has existing validation failures unrelated
to this field. For runtime testing only, its sandbox data was corrected to
FICO 650, phone-2 area code 212, and state NY (matching ZIP 12546).
Bundled source data files are unchanged.

The preview uses the native GnuCOBOL CICS compatibility runtime, not an IBM
mainframe deployment. Mainframe compilation and deployment remain untested.
