# Brian Julius (55k Followers) about tellingCube

**Post- Text**
Inexpensive, non-subscription programs that PERFECTLY solve a single problem are the ivory-billed woodpeckers of the data world. Rumored to be extinct, I spotted two in 2025 - Typora in August, and now TellingCube...

Typora - https://typora.io/
-----------------------------
Beautiful, intuitive $15 markdown editor that I open 50-100x for every 1x I use Word. An essential component of my AI stack, along with Hashtag#Claude Code in Hashtag#VSCode, Hashtag#Gemini, Supabase, Plotly Studio, and Obsidian.

Non-software equivalent: the Global GS-38 paring knife - modern, streamlined, razor sharp, effortless to use

TellingCube - https://tellingcube.com/
------------------------------------------
Event-based business data generator that produces highly realistic, reconciled financial data across sectors and market caps, specifically engineered for Hashtag#ICBS. One-time €29 + generous free tier

Non-software equivalent: Zippo Classic Windproof lighter - specific use, but one distinctive click instantly, reliably meets the exact need

In the brief video below I walk through TellingCube's current and planned features, and discuss its major advantages over alternative data simulation tools (e.g., AI, R, Python, Excel, etc.)

And in the conclusion tomorrow, I fire up a brand new TellingCube dataset to put my AI-driven, agentic-looping IBCS Claude skill to the test vs the most challenging of the IBCS visuals

A huge thanks to Mario Semper for making this outstanding tool available to the community

 If you have a need for simulated business data that looks, behaves, and can be analyzed meaningfully like real financial data, I definitely recommend you give it a look! 

As always, I have no financial or affiliate relationship with TellingCube, or any of the other products mentioned above. They are all just beautifully engineered tools that perfectly meet a need, and that I enjoy using.



**Video Transcript**

---

Hey everybody, tonight's video is one that I'm doing as a follow-up to the one I did last night on the agentic loop and the creation of IBCS visuals. But before I get into that, I wanted to talk about a utility that I purchased recently that I just love. It just solves exactly the perfect problem that I've been having for years.

And it's been a while since I've recommended software, just because most of the time I just stay in Claude Code these days. But I think the last one I recommended was Typora. And one of the things I loved about Typora as the markdown editor was it's a one-time cost. It's not a subscription. You just pay, I think it's 15 bucks, and you get this great editor that you can use in all different ways in AI. And this is a really similar program in some sense. It's a one-time cost. It just does one thing, but it does it perfectly. And I'm really enthusiastic about it.

So what this is, it's a program called tellingCube. And the developer, Mario Semper, contacted me a week or so ago. I posted something about IBCS. And he said, I've come up with this program that generates realistic data for IBCS practice and visuals and analysis. And it does it from a financial controller perspective.

And what I want to show you in this is one of the things that I've always had trouble with is developing realistic data, companies and businesses, that's consistent. That when you put it in visuals, it doesn't look crazy. The numbers add up. And what you get out of it, you're able to actually extract insights and recommendations from it. It's not just randomized data.

And that's what I love about this is he's actually got startups, mid-caps, large caps in different industrial categories. And these are kind of simulated companies he builds up from basically simulated employees, simulated sales, simulated customers, simulated vendors. And each one is built on a specific transaction. So it's people billing hours or cash transfers to pay for sales. And it all comes out looking like real data.

And it's great for teaching. It's great for simulation. It's great for testing.

And the other thing I really like about it is that you can extract it at the raw level, at the event level. You can extract it at the aggregated level. You can pull it down by CSV. You can pull it down by API. And it just kind of gives you the full experience of having actual data. And he's got a whole roadmap here.

So here's what it already has. One of the things I love about this is building out an HR view and good HR data is just really difficult to get. And so you just get scenario comparison, different types of exports. This multi-year scenario, kind of looking at a three to five year business plan is something I'm really looking forward to.

So it just, for me at least, it really fills a need. It's like 30 bucks for an individual user license, one time. And if you have a need for really, really slick simulated data for teaching or for practice or experimentation, I highly, highly recommend it. I bought my own copy and I'm really pleased with it.

I'll show you one of the things I like is it just integrates beautifully with Plotly Studio. So let me, let's go here and let's generate, I'll show you what generating a scenario looks like. All right. So let's, yeah, this is what I haven't played with yet is the credit union. So let's generate this.

You'll see it takes about a minute and a half and it generates the full suite of company data. It runs consistency checks with regard to revenue and cost of goods, customers and employees. And so it's going to go through all that. And then what we'll do is we'll take the API data and we'll pull that into Plotly Studio and then into Claude Code. And we'll try to run a more difficult one than we did yesterday.

We did a stack bar chart. This one, I want to do an actual versus previous year by month. And then what I want to do is I want to do relative and actual variances. So we've got three subplots. That's probably, I think, one of the more difficult IBCS charts to build. It gives a lot of programs trouble in terms of aligning the subcharts. And so I want to see how that one does with our IBCS skill.

So this should be done sharing pretty soon. I'll show you what this looks like. He has some nice visuals in here. I typically just use them as kind of an initial snapshot. I don't actually do anything with the visuals. I tend to create my own. But as I say, it's a polished program. I think he's going to be adding a lot to it.

So this one has actual previous year in plan. So that's perfect for what we need. And it should be done any minute now. This one, I think, is a bit larger and more complex than the ones I typically run with a small cap. I just haven't needed a lot of rows and a really extensive data set.

The other thing is I've talked to Mario a number of times, and he's really open to feedback. I think he really wants to hear from people who are using the program and what their needs are.

So here's kind of the overview. He's got some nice IBCS type visuals here. And the technical details, so the revenue reconciliation, past payroll alignment, and the cost of goods matching. So it is realistic looking data.

So let's jump up here. So this is going to be the real raw data. And let's just copy this. And we can just jump into Plotly Studio. And one of the cool things Plotly Studio has done recently, instead of waiting for connectors, right here, you can just punch in an API. You can punch in a program, like you use Supabase or Fabric or Databricks. You just punch it in, and it just builds the connector right on the fly.

So let's just drop the API in here. Let's see how this does. Okay, so it recognizes the tellingCube API data. It's loading the data in the preview. And so we didn't have to do anything other than just paste. There's no configuration.

The one thing that Mario has right now is no authentication. Just once you subscribe, your email is in the registry, and you just download without a key. But he is going to be adding authentication, so you can test on that as well.

And so, yeah, this is going to be perfect for what we need to do. So my original intent on this was to run both the tellingCube and the IBCS skill portion together in the same video. But it turned out to be a little bit longer than I thought it would be. And so what I'm going to do is I'm going to divide it up. And tonight, I'm just going to post the tellingCube portion. And then tomorrow, I'll post the IBCS skill where we're going to do the relative and absolute variance subcharts.

And so that one, I think, is a good test. It's one of the more difficult of the IBCS charts to create, I think.

I'll stop here and say I really like the work that Mario did here. It fills the need perfectly for me. As always, I don't have any financial or affiliate arrangement. I purchased it just like I'm recommending some of you to do. But it's one of those things I say it really fills the need. I think it's really well implemented. I'm very much looking forward to the build out that he does of some of the more complex features.

But it really is a big time saver and a good way to test the skill build as well as some of the analytical stuff we've been doing with IBCS. So really, shout out to Mario for that. And I'll put the information in the post for you.

As always, thanks for watching. And I hope to see you tomorrow night when we'll run through the second of the IBCS skills.

---

*Speaker: Brian Julius | Topic: tellingCube Review*
