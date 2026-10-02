---
title: "Managing Github Notifications with Mellea Skills Composer"
date: 2026-06-12
draft: false
---

{{< rawhtml >}}
<center><img src="../../images/mellea-todo.png" width="200" alt="tasks to do"></center>
{{</ rawhtml >}}

I maintain a number of projects, both on public GitHub and on my internal enterprise GitHub. As a result, at any given time, I'm watching hundreds of different repos and getting notifications from all of them. That results in major cognitive overload. On given days, I'll have hundreds, if not more, notifications coming in. Now, the GitHub notification inbox on the web UI is, for lack of a better term, not great.

Trying to figure out what you want to do, what to look at, what to prioritize isn't super easy. Often times you more or less find yourself looking at things in chronological order. Which is fine except for when you end up missing the critical thing because it's buried in the middle somewhere. Also, I'm really not a fan of trying to use the GitHub web UI, especially as it can get laggy. And email notifications are even worse.

I already use org-mode in Emacs to manage a lot of my day-to-day things. So if there was an easy way to manage my Github tasks in org, that'd be wonderful. In the past I had tried hacking something together, but it never really went anywhere.

Enter the era of LLMs and coding agents.

Now it became rather trivial to pull something together, especially once SKILLS.md rolled out late last year. So I created a skill (and then later a subagent) where I was able to specify how I wanted my job to go out, pull my GitHub notifications, order them in an `.org` doc so that I could use it locally in Emacs and in particular drive everything from the keyboard without having to worry about weird mouse clicks and things like that.

And this worked... pretty good. I even was able to have the LLM summarize and categorize them so that I got kind of a prioritized list which was mostly helpful. Every once in a while we'd hit a few issues, like the LLM would take some PR that I'd requested reviewed on and put it so far down in the FYI category that meant I didn't look at it in time. Or it would forget to record something in the doc and that item would vanish into thin air (we call that "the AI ate my homework"). But by and large I was using this as my daily driver pretty much every day for the past 6, 7, 8 months.

Though I've got to be honest, the whole setup was a little bit overkill. My markdown file got bigger and bigger (and more and more complex), and running it wasn't trivial either: it would cost between 60 and 80 cents per run. And I was running it multiple times throughout the day. Over time those calls started to add up (and tokenmaxxing is shall we say frowned upon here).

When I started working on the [Mellea](https://github.com/generative-computing/mellea) project earlier this year, one of the goals I set for myself was to make sure I eventually used Mellea in all of the agentic type things that I was doing. So this seemed like a good candidate to migrate over. I turned to the [Mellea Skills Compiler](https://github.com/generative-computing/mellea-skills-compiler), and within about five minutes I had a Python program that I can run from my laptop whenever I want.

An example of how this changed the workflow:

I mentioned above that one of the problems I would have is sometimes the agent would mistakenly close the issue before it had done the right and things would kind of slip through the cracks. In this case, we had to kind of tell the agent what to do, but we didn't really have any guarantees that it would actually do what we asked it to do.

```markdown
**Gate — confirm the write first.** Do NOT mark anything Done until you have
verified the doc wrote. Confirm with:

    test -s "$INBOX" && echo "WROTE_OK" || echo "WRITE_FAILED"

If this does not print `WROTE_OK`, STOP. Do not mark anything Done.
```
*([source](https://github.com/psschwei/ghn/blob/6ad2fb291983203520925ba81a87ce9f7ea851ed/.claude/agents/github-notifications.md?plain=1#L386-L394))*

But when we melleafyed it we got an actual function with an `if` block, and if our `if` didn't pass we didn't drop the notification.

```python
def confirm_inbox_written(inbox_path) -> bool:
    p = Path(inbox_path)
    return p.exists() and p.stat().st_size > 0
```
*([source](https://github.com/psschwei/ghn/blob/6ad2fb291983203520925ba81a87ce9f7ea851ed/ghn/pipeline.py#L279-L282))*

```python
# ... after writing the doc:
if confirm_inbox_written(inbox_path):
    for item in folded_threads:
        mark_thread_done(item["id"], host_for(item))
```
*([source](https://github.com/psschwei/ghn/blob/6ad2fb291983203520925ba81a87ce9f7ea851ed/ghn/pipeline.py#L437-L444))*

We don't have to rely on the LLM for the control flow (which they are notoriously [not amazing](https://mellea.ai/blogs/generative-computing/) at); we give our program the control flow.

There's other benefits too. Instead of a frontier model, I use Granite 4.1 3B running on Ollama on my laptop. And the results are, honestly, just as good ([small models rock!](https://mellea.ai/blogs/small-models-rock/)). The one place that things could maybe be a little better is the classification, but I'm toying with using a slightly bigger model to do that, and I expect that should take care of it.

The cost (free) is the obvious benefit but it's also helpful to be able to reason through the code to debug things when things go wrong. Plus, I was able to use `uv` to install it as a tool, and now I can pretty much call it from anywhere (in other words, I don't have to start a session with my coding agent to run it).

I put both the melleafyed program and the original agent on GitHub: https://github.com/psschwei/ghn so you can test out the difference between the two. One day I might write up a side-by-side comparison.

And I'll keep tinkering on it to better fit it into my workflow.

Links:
* Mellea: https://mellea.ai
* Mellea Skills Compiler: https://generative-computing.github.io/mellea-skills-compiler/
* Granite 4.1 3B: https://huggingface.co/ibm-granite/granite-4.1-3b

_(Note: I used [Granite Speech](https://huggingface.co/ibm-granite/granite-speech-4.1-2b) to dictate this blog, so apologies if it rambles a bit.)_
