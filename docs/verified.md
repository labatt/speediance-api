# Verified routes

Routes confirmed against a live account. Base URL `https://api2.speediance.com`,
prefix `/api/`. Every route here is a `GET` unless stated otherwise. Field names
are the API's own; example values are omitted.

A response is wrapped: `{"code": 0, "message": ..., "data": ...}`. `code` 0 means
success; the table at the end lists the codes seen so far.

## Rowing and ski telemetry

### `app/boatingSkiDataGraph/{uuid}`

Per-point telemetry for a rowing or ski erg session — the only known source of
anything finer than session totals.

**Keyed on the session `uuid`, not the numeric training id.** Passing the numeric
id returns `null` with `code` 0. Fetch the session summary first
(`app/trainingInfo/courseTrainingInfo/{trainingId}`), which carries both `uuid`
and `existBoatingSkiDataGraph`; only call this route when that flag is `true`.

```
data.id                      the uuid echoed back
data.type                    integer
data.pointDataList[]         one sample roughly every 3 seconds
```

Each point:

| Field | Meaning |
|---|---|
| `time` | seconds from the session start |
| `spm` | strokes per minute |
| `resistance` | resistance setting |
| `power` | watts |
| `pace` | seconds per 500 m |
| `minSpm`, `maxSpm` | the target stroke-rate band at this moment |
| `minResistance`, `maxResistance` | the target resistance band at this moment |

The `min*`/`max*` fields describe the programmed target for that part of the
workout, so a change in them marks an interval boundary. Two observations worth
handling: `pace` in the opening seconds reflects the flywheel spinning up rather
than real pace, and `spm` can read `0` on individual samples.

Session totals (`totalDistance`, `trainingTime`, `calorie`, `totalEnergy`) come
from the session summary, not from here.

## Training stats

### `mobile/userHealth/trainingLoad/dateRange`

Parameters: `startDate`, `endDate` (`YYYY-MM-DD`). Returns one row per day:

```
dateStr, trainingLoadValue, trainingLoadUnit ("TRIMP"), recommendMin, recommendMax
```

### `app/userDataStat/trainingPartFatigueInfo`

No parameters. One row per muscle group: `trainingPartId2`, `fatigue`,
`muscleLoadStatus`. Group names are not included; `trainingPartId2` maps to the
ids used by `app/actionLibraryGroup/trainingPartGroup`.

### `app/strengthScore/summary`

No parameters. `totalScore`, `scoreUpdateTime`, and a per-region object
(`chest`, `legs`, `back`, `shoulder`). Regions are empty until enough assessed
work exists.

## Body and profile

### `app/userBody/current`

No parameters. `appUserId`, `weight`, `height`, `bmi`, `bodyFatPercentage`.
Values are in the account's display unit.

### `app/userPreference`

No parameters. Device toggles: `autoOffWeight`, `autoInfinityMode`,
`autoPlayTrainMusic`, `voiceprintSwitch`.

### `mobile/userHealth/newIndex/healthScore`

No parameters. One object per card (`walking`, `nutrition`, `bodyAge`,
`wellnessMonitor`), each with `value`, `createTimestamp` and sometimes a JSON
string in `extData`.

### `mobile/userHealth/mockReport/hasRealRecoveryData`, `.../hasRealSleepData`

No parameters; each returns a bare boolean. Useful before rendering a recovery or
sleep screen, since those screens fall back to sample data.

## Health and readiness

The date parameter for these is `dateStr`, formatted `YYYY-MM-DD`.

### `mobile/userHealth/physicalCondition/detailByDate`

Parameter: `dateStr`. The training-status model, and the one health route that returns
real data without a wearable:

```
physicalConditionResp.physicalConditionValue    training status (fitness / fatigue)
physicalConditionResp.physicalConditionScore    0-100
physicalConditionResp.fitness                   long-term load
physicalConditionResp.fatigue                   short-term load
physicalConditionResp.trainingStatus
physicalConditionParam.trainingStatusScore      and trainingStatusScoreAvg
```

### `mobile/userHealth/recovery/detailByDate`

Parameter: `dateStr`. Returns `recoveryScoreResp`, `nightHrvResp`,
`nightRestingHeartRateResp`, `sleepResp`, `muscleLoadResp`. All are empty without a
device recording overnight HRV and resting heart rate — check
`mobile/userHealth/mockReport/hasRealRecoveryData` first.

### `mobile/userHealth/sleep`

Parameter: `dateStr`. Returns `sleep` and `targetSleepMin`.
`mobile/userHealth/sleep/rhythm` is a POST, not a GET.

## Routes whose parameters are still unknown

Each returns `code` 10 (parameter error) when called bare, so the route exists but
the parameter names have not been confirmed. Listed so nobody re-derives them:

```
app/userDataStat/boatingSki, boatingSkiStatByDateType, boatingSkiStatDetail
app/userBody/stat
mobile/userHealth/physicalCondition/listDailyDateRange
mobile/userHealth/calendar/scores
mobile/userHealth/newIndex/healthDataIndicators
```

None of them accept `dateStr`, `startDate`/`endDate`, `startDateStr`/`endDateStr`,
`beginDateStr`, `monthStr` or `dateType`.

Response field names recovered from the app, which may help identify them:

- recovery: `recoveryScoreResp`, `nightHrvResp`, `nightRestingHeartRateResp`,
  `nightRestHeartRate`, `sleepScoreAvg`
- sleep: `sleepQualityScore`, `sleepEfficiencyScore`, `sleepRegularityScore`,
  `sleepDurationScorePercent`, `deepSleepDuration`, `sleepOnsetLatency`,
  `longestSleepStreak`, `scheduleDeviation`, `bedDownSleep`

Recovery appears to be derived from overnight HRV and resting heart rate, so it
stays empty without a paired device recording overnight.

Other observations: `app/userinfo/setting` rejects `GET` with `code` 12, so it is
a write route. `mobile/muscleLoad/detail` returns `code` 500 when called bare.

## Response codes seen

| Code | Meaning |
|---|---|
| 0 | success |
| 10 | parameter error |
| 12 | request type not supported (wrong HTTP method) |
| 90 | session displaced — another login took this client-type slot |
| 91 | token expired |
| 98 | client version too low for this content |
| 500 | server error |
| 1002 | invalid appid (unknown `App_type`) |
| 1003 / 1004 / 1005 | missing or invalid version code / nonce / signature |
