# V013 Lane — DB_CONTEXT V003

Status: `VERIFIED`  
Task: `PROMPT305D — Catalog Sync + Booking Create`  
Investigator: Cursor  

## Lane

| Field | Value |
|---|---|
| WPF | `obm_pos_dev_v013_pg` |
| API | `obm_api_dev_v013_pg` |
| TenantGuid | `921df16c-a48d-4d0b-b4e7-fcd8ddf71a2c` |
| V012 mutated? | **NO** |

## After catalog + booking

| Metric | Value |
|---|---|
| WPF outbox total / Sent=2 | 276 / 276 |
| API TblServiceCategory / TblService | 17 / 152 |
| API EventLog cat+svc | 17 + 152 |
| SOM Asc/Desc | 13 / 4 |
| Booking create | `f53cd35c-d7bc-4b2e-a94b-ba3bc7c1ad6b` assignType=2 |
| TblWebService* | 0 |

## Pointers

- Local report V001 Prompt305D
- Coordination `report/report147.md`
- Prior V002 = full baseline drain
