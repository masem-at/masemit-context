### Home

#### Problem / Solution

The Solution > "3 years of..." (we have one year now)

#### Product Plans

Supporter S <> Supporter M > only difference is Email Support vs Priority Support
Unlimited for all? or should we consider different volumes > if we change this we have also to change terms & conds
should we handle api access different?
Think in general about this topic

#### FAQ

Do i need an account? > should we mention the limitation?
What company types are available? > out of date, we have a 3x3 matrix now
Can i export data? > "APi docs" should be a link to the page, "What's next" isn't a link, pls make a link to what's next page

---

### What's Next

#### Now

Company Tiers > out of date, we have 3x3 now
SalesView > out of date, we have new charts
FinanceView > out of date, we have new charts
CSV Export > out of date, export each view AND the event source

#### Next

More industries > pls mention, that we want provide more sectors out of GICS, e.g.:... and sub sectors
I want to add here that the next step is to get a SaaS, with dashboard, background generation (Sophie/Sally search for a better wording), ...

#### Vision

tellingDash > don't use "IBCS(c)" standalone, we are only allowed to use "inspired by IBCS" or something like that

---

### My scenarios

"no generation costs" > there are no costs at the moment
pls check company name, if the correct one is fetched from the database
maybe we could place the date somewhere other, because i also had during testing the issue to find the scenario, that i want to find
delete action (cascading deletion on database end)
footer is missing

---

### Privacy Policy

#### Third-Party Services
should we add neon db? (hosted in germany/frankfurt)

#### Contact
for contact pls use contact@masem.at for the other topics support@masem.at is fine.

---

### Terms & Conditions

#### Contact
use contact@masem.at as contact email

---

### Result Page

#### Sales View

##### Company Header
should the badge with the company on the right side stay? because sector and size is already displayed under companyname, maybe sector and sizewe should badge, i don't know, sally it's your turn
Maybe we could move the timeframe and generation date section right aligned instead, because at the moment it feels like a messy block. but it's only a proposal, think about it.

##### Chart 1
"Net sales in k€" > in the chart is the unit "M" e.g. 17.3M, it's different.
Additional, as far as i remember from the ibcs standards, if the unit is in the header, we don't need it in the chart/table.
pls see:
docs\standards\ibcs-cheatsheet.md
docs\standards\ibcs-standards.md
if don't find this "rule", pls ask me, that's just a summary, i have the whole standards.
I'm not sure if it's in the standards, but in most IBC visualizations, head lines and sum lines are not continuous, but per column.

##### Chart 2
same for units
same for lines
Headers are shortened, but there is enough space to write more and shorten on a wider length. should be more dynamically (especially tkaing mobile in mind)

##### Chart 3
units ok, in Header €, in chart/table "M", could be changed to M€ in the header and no unit in the chart/table. Must be flexible, depending on the numbers.

##### Sales Trend Analysis
I would move it to the top of the page

#### Finance View

##### Chart 1
same unit issue/mix as in sales view
subcharts "revenue trend" and "departments active" could be a little bit bigger
on "departments active" i see a a stright horizontal line, is this a time series?

##### Margin Analysis
pls remove this section completely

##### Cost Breakdown
no unit neither in header nor in table. i see it in footer. put it in header as for the other visulisations

##### Costs by department
remove unit from chart and put it in header "k€", depending on numbers

#### API Documentation
could look a little bit more fancy.
more structure. could we use cool looking codeblocks for the examples?