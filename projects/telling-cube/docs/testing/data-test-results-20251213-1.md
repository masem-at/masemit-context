# Testing the data (also some UX fixes)

## Event Source Export
Currently the export shows prod-001, cust-001, .... can we join the real names into the export`?

## Processing Indicator
Currently we have 4 steps when generating data:

- build world
- ...
- ...
- ...
- validation

The flow is the following:

- build world 10%->30% (huge task)
- next 3 steps are done in 1-2s jumps to 90 or 95%
- validation on 90/95% (huge taask)

Can we somehow do this better for a better user expirience?

## Notable Events
Is it stated in the prompt for Claude that it can also deliver notable events? I haven't seen any yet. If there is one, is this information used anywhere?

## Scenario Id
Pls provide the scenario id as small information in my scenarios and result page.