# Brian Julius: IBCS Visuals with the Agentic Loop

**LinkedIn Post + Video Transcript**

---

## LinkedIn Post

**Brian Julius**
*Experimenting at the edge of AI and data to make you a better analyst | 6x LinkedIn Top Voice | Lifelong Data Geek | IBCS Certified Data Analyst*

---

Forget about AI taking your job. AI has given you all three pieces of a winning Powerball ticket. Here's how after 18 months of trying (and failing), I now use these pieces to generate ≈perfect #IBCS visuals on demand...

I have always been a huge fan of the International Business Communications Standards (#IBCS), a consistent, familiar "semantic notation" for business reporting developed by Dr. Rolf Hichert and Dr. Jürgen Faisst.

While research has shown that visuals built on the IBCS standards decrease user processing errors by 61%, and user processing time by 46%, the problem is that there has been no easy, cross-platform way to create the full range of IBCS visuals.

In 2024, I began working with Jürgen, Rolf and the team at the IBCS Institute to determine whether AI could provide a widely accessible, comprehensive and straightforward solution.

Over the 18 months that followed, we tested a wide range of models and approaches, but none were able to achieve the consistency and accuracy that IBCS demanded.

Until now.

The OODA (Observe, Orient, Decide, Act) loop was originally developed in the 1970s as a way for fighter pilots to quickly process and respond to threats.

Now often referred to as the "agentic loop," it has become an extremely powerful technique used by probabilistic AI models to provide reliable and accurate outputs via iteration.

However, it is only very recently that AI-driven agentic loops became feasible as the rapidly advancing intelligence and multi-modal capabilities of AI models, their ability to use tools via MCP, and their ability to iterate using Claude Agent Skills all converged in late 2025.

Here's how I've implemented the Agentic loop in the context of IBCS:

1. The user specifies the visual they want to create, based on one of the 16 "reference visuals" on the IBCS website

2. The AI model views the chosen reference visual, and through "progressive disclosure" gains sufficient context to code an initial Python version

3. That initial version is rendered to a local web server where via MCP tools, the model checks it against an extensive "verification checklist" (≈800 lines) I've built to determine if it conforms fully to the IBCS requirements

4. It repeats this process until all relevant checklist requirements are met

5. The user then has a chance to review the visual to provide any fine-tuning adjustments (the reference visuals don't cover all possible variants)

6. The model performs a similar agentic loop to ensure that all adjustments are properly implemented

7. Once the user approves the resulting visual, AI updates the verification checklist and a lessons learned document applied to subsequent tasks

8. Final outputs in the specified formats are provided to the user

The expectation is that over time, the memory will minimize/eliminate the need for the second agentic loop.

---

## Video Transcript

A couple of days ago, Adam Schroeder posted a really interesting post about how he used his prompting skills in Plotly Studio to do a natural language prompt of an IBCS-style stacked bar chart. And I'm not nearly as good at prompting as Adam is, and so what I did is I took a different approach. And I used a Claude skill and took advantage of the code-based agentic loop.

And let me show you how I did that. So what I did is I went in and I pulled out from Adam's chart, I pulled out the CSV values. And then what I did is I created a Claude skill, and let's call that up.

So we'll say, using the IBCS Claude skill, create an IBCS stacked bar chart, implementing the full workflow using the adamoriginalsales.csv. Let's just fire that up, and what that should do, that should invoke the skill, and then that skill has a workflow. Okay, so there's the, it says use the skill, and we'll say yes. We'll get that skill running, and it'll pull up a workflow, and it'll ask us a few questions.

Okay, so let's give that a little more room to breathe. There we go. Okay, so it's going through the skill, it's doing some reading.

It's going to ask us some questions about what we want. What we can do here is we can give it, I should have just dragged the data set in here. So let's just give it a copy of that.

It's always best when you can to be specific and to give it all the information that it needs. So it's reading through the data set, and it's reading through the skills. And the way the skills work is by something called progressive disclosure.

And what that is is that it only pulls the information that it needs to at the time. It doesn't read in the full amount of the skill. It just pulls in, there's a brief YAML description that says what it does.

And then it kind of progressively goes through and pulls more information into the context as it needs additional information to complete it. So right now, it's just kind of reviewing the workflow, kind of getting acclimated to the skill. And it should, in a minute or so, it should start asking us a few questions about exactly what it is we want.

And what I've done here is just, I copied here these reference charts. And what these are is, in the IBCS webpage, it's got a series of charts. So I think 13 charts and three tables.

They just go through, and for the different types of IBCS visuals, it just gives kind of a standard reference guide for each of those. And so we'll use that as kind of our target value for the agentic loop. Okay, so it's found the dataset we gave it.

And it sees that we wanted to create a stacked column chart. So it's pulled in the proper asset from the reference files. All right, there we go.

So it's setting up the workflow. And this is what it does. It'll pull together kind of a general plan for how it's going to attack the problem.

And this is what it typically does. It kind of gives you a sort of a questionnaire. And says, yes, this is what we want.

We want AC through actual values through December 2025. And then plan values from January on. We'll say yes to that.

And then it's going to ask us what we want. And I think here what we want, I always like to get a PNG file. Definitely want to get the Python code.

And let's get, yeah, let's just go with that. Actually, we can get one more. Why don't we say a detailed prompt from the Python code that reproduces the same chart as close as possible.

And then we're just going to hit submit. And that should start up that agentic loop. What it's going to do here, it's going to code the IBCS stacked chart as best it can.

And then what it's going to do is it's going to post that chart to a local web server. And then it's going to use what's called the Playwright skill. Which lets it take screenshots and use its visual skills to take a look and make sure that all of the elements of that chart are properly constructed.

And then it will give us the chance to review and say whether we agree that it's done everything right. And in this case, this is early in the development of this skill. So what I've got is it generally gets very close in the agentic loop.

But because these reference charts don't cover every possible detail and variance in how you might want to implement this. There are often one round of changes that you have to make, kind of some fine tuning changes. But what I have is I have a lessons learned loop that when you make those additional changes that it then captures those for the next time.

And so it should theoretically get better over time. And we should come closer and closer to one-shotting just based on the reference visual and the lessons learned. It's going through, it's still doing some coding.

And we're just hitting this yes button. Because what I typically like to do is I like to monitor exactly what it's doing. And I like to have it ask me for permission.

Particularly given the fact that it's modifying local files and posting to a local web server. So I generally like to keep a tight eye on it. Okay, so what it's done here is it's telling us that in the outputs it should have a screenshot here.

Let's just blow this up and take a look at it. That's looking awfully good. I actually don't see anything distinctly wrong with that.

Let's just take one more look. Wow, that's really looking good. Let's say I think that looks spot on.

So I do not see any additional changes to make on that. So the lessons learned loop appears to be working great. So let's slide that back a little bit.

Okay, and we can just tell it looks great. No changes. The only thing I think it hasn't generated for us is the detailed prompt.

Okay, it just ran into some small trouble with some emoji characters that it put in the Python script. Just fixing those. It should just be creating that markdown version of the prompt that we asked for.

I think everything else has been done. And there we go. It'll give us kind of a summary of what it's done, Python script.

Okay, so it didn't give us the prompt. So let's give us the prompt that will create the same chart as the final Python script. If you remember, it didn't ask us for that prompt initially.

So we added that as an additional option. So I'll need to tweak the skill a little bit so that it remembers. If we enter an option that wasn't originally part of the pick list, that it should still provide that to us at the end.

But that's pretty close, I think. Especially given the fact that it seems to have nailed the chart exactly right. All right, so there we go.

So now we've got the markdown prompt. Let me show you something really neat that I found about the code. If we take this code, we copy this, and we go back into Plotly Studio.

What we can do here, this is Adam's chart. Now what we can do is we can say, okay, let's add a new visual. Let's paste this code in.

And then just put a note up here at the very top that says, we're going to use that code as a prompt. Create a chart based on the exact code below. And if we just wait for that to generate, it should, based on past experiments I've done, it should generate that exact chart.

What we can do is we can also create another one that tests how it does with the prompt. So we go back here, and then we just copy the prompt itself in. Let's see how close that comes.

So here's the one created from the code. That looks fantastic. That's exactly what we hoped for.

And then if we go down here, we'll see what the prompted one looks like. This is pretty good. It needs a little bit of work.

The bars aren't centered. It's got a little bit of a problem with the title. You can see the code is more precise, and that's almost always going to be the case.

The code is literal. The prompt is interpreted. So that's how I do it with the agentic loop.

I think it's really a powerful technique, and you can describe anything with no specificity in terms of the final product that you want. You can just have it iterate to a target, and it'll come out exactly as expected, given how specific you've been in the development of that looping mechanism.

So I think I'm going to do another one like this, maybe a slightly more difficult multi-chart with relative and absolute variance. So if you're interested, join me back in a couple of days, and I should have a good example for you.

I've also got a really neat tool that somebody developed in terms of generating realistic data sets for IBCS-type visual work. So I wanted to show you that, and as always, again, thanks for watching, and I'll see you in the next video.

---

*Speaker: Brian Julius | Topic: IBCS Agentic Loop with Claude Skills*
