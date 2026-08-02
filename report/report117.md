# Report 117

## Verdict

OBM_WPF_INSTALLATION_STARTUP_CRASH_CLOSED_LOCAL_FIRST_STABLE_READY_TO_RESUME_POS_DEVICE_WRITER_DESIGN

## Public Coordination Summary

Prompt116 updated artifact SHA verified: yes.

Latest WPF label observed: prompt117.

WPF executable/hash proof: yes.

Current startup state classification: S3_PHASE2_INCOMPLETE_SHOULD_STAY_INSTALLATION.

Installation UI shown: yes.

Measured seconds to original error: process terminated before the 12-second inspection window; Windows event timestamps place the unhandled exception about three seconds after process start.

Exact crash trigger category: API/bootstrap call exception escaping the InstallationV0 async Loaded resume path.

Exact throwing class/method: Phase1InstallationService.CallProtectedHelloAsync.

Unhandled exception type: System.Net.Http.HttpRequestException.

Process exited before correction: yes, exit code -532462766.

Process exited after correction: no during the physical observation window.

Current-state physical startup stable: yes.

Current-state observed duration: 20 seconds.

Valid installed-local state proof: not-available.

MainWindow physical proof: not-available.

Incomplete installation recoverable proof: yes.

API/cloud outage local-first proof: yes for InstallationV0 recovery; not-applicable for MainWindow in the current schema-incomplete state.

WPF canonical runtime DB proof: yes.

WPF pending migrations count: not-reached; current assessment reports schema migration required.

Focused test totals: 56 passed, 0 failed, 0 skipped.

WPF build totals: 0 errors, 172 warnings.

TblTenantPosDevice changed: no.

API DB mutated: no.

WPF DB reset performed: no.

POS1-POS10 UI/Pairing Code changed: no.

Category Weight changed: no.

Booking Weight changed: no.

Manual POS1 test ready: false.

Production/customer/reference DB mutated: no.

Private artifact: yes.

Aggregate SHA-256: 257afc02890a0b560e0c50d60d8f51a715b8203400f31d6f13e7e08653973395.
