---
layout: post
title: Automation with ChatGPT, Github Actions, and Power Automate
tags: [ChatGPT, OpenAI, Github Actions, Github Copilot, Power Automate]
---

# {{ page.title }}

This past week I wanted to get more hands-on experience with the OpenAI APIs and I had a specific use case:  speeding up some of the manual work it takes to verify a release for [Apache AGE.](https://age.apache.org) The bigger benefit is that I also got to play with Github Copilot, Power Automate, and Github Actions too!

At the very high level, I wanted to accomplish the following:

1.  Receive email in O365 to Vote/check Apache AGE release
2.  Use Power Automate to handle email  
    1.  Filter on subject “[VOTE] Apache AGE”  
    2.  Call OpenAI API with body of email to extract some key fields for the release  
    3.  Trigger Github Action to verify release with payload from above step
3.  Github Action is invoked
    1.  Run Job 1 – handling high-level checks (signature verification, sha512 check, file comparison)
    2.  Run Job 2 - smoke tests in Postgres / Apache AGE (create a graph, run test suite)
 
Below is a sequence diagram of how it plays out:

<img src="https://sorrell.github.io/images/202304/sequence_ai.png" width="75%">  

## Power Automate

First of all, I get [emails that look like this](https://lists.apache.org/thread/t4kjb9rhbkyxscvbx4948wrbcz0hj2fq) asking to help verify the release. So, given that ChatGPT excels at handling natural language, I thought it would be great to feed this email to ChatGPT to extract some key fields.

This part was a little tricky to get right, but I used ChatGPT to help guide me through a lot of it. One downside is that you need a Premium Power Automate license to utilize the HTTP Request flow (BOOOOO!).

Below is the high-level flow inside of Power Automate.

<img src="https://sorrell.github.io/images/202304/power_automate_1.png" width="75%">

### PA Flow Step #1 - When a new email arrives V3

**Goal:** Trigger workflows 

This part is straightforward - filtering on emails with a certain verbiage in the subject line.

<img src="https://sorrell.github.io/images/202304/power_automate_2.png" width="75%">

### PA Flow Step #2 - HTTP Request

**Goal:** Call ChatGPT API to extract the fingerprint, PostgreSQL version, Apache AGE version, RC version, and git commit hash as a json object.

Again, straightforward. I'm using the following prompt before passing along the body of the email.

```json
 {
    "role": "system",
    "content": "You are a helpful assistant that is an expert in parsing emails and turning information into JSON objects. Your JSON should be lowercase, snakecase, and well-formed valid JSON without XML tags or extra newlines. "
},
{
    "role": "user",
    "content": "Parse the following email and extract the fingerprint, PostgreSQL version, Apache AGE version, RC version, and git commit hash as a json object: @{triggerOutputs()?['body/body']}"
}
```

Why am I telling it about XML tags? Well, when I was using the `text-davinci-003` model, it was sometimes returning a `</code>` tag! WEIRD!

Below is what the full flow looks like:

<img src="https://sorrell.github.io/images/202304/power_automate_3.png" width="75%">

### PA Flow Step #3 - GitHub

**Goal:** Trigger a GitHub Action to run some automated tests

This part was a little trickier to get _just right_. ChatGPT didn't have the "out-of-the-box" instructions like it does on most other things. For example, ChatGPT gave me this instruction:

> Search for "GitHub" and choose "GitHub - Run a workflow". If you haven't connected your GitHub account to Power Automate yet, you'll be prompted to sign in.

That flow may have existed in 2021 when the training data went through ChatGPT, but that flow no longer exists. You now need the "Create a repository dispatch event." 

One somewhat scary thing about using this native flow is that signing into Github here requires you to give full access to an app owned by `aaptapps.` Some searching showed this to really be owned by Microsoft, but my God is that the worst name ever.

<img src="https://sorrell.github.io/images/202304/scary_app.png" width="75%">

After that, you need to configure a few more things. For example, the Event Payload should reference the name of the previous flow step (HTTPReq in my case) and then parse the response.

`json(outputs('HTTPReq')?['body']?['choices'][0]?['message']?['content'])`

Below is what the flow looks like in my case.

<img src="https://sorrell.github.io/images/202304/power_automate_4.png" width="75%">

## Github Actions

Next up was the actual testing of the release with the data I've collected so far. In the last step of the Power Automate flow, I am calling the Github API with an event payload that contains this data. It invokes an Action that is stored in [this "Apache AGE Release Verification" repository.](https://github.com/sorrell/apache_age_release_verification)

The workflow here is that I want to run high level tests and then a smoketest.

<img src="https://sorrell.github.io/images/202304/Github-action.png" width="75%">

I hadn't used Github Actions to receive or parse a payload before, so it took me more time than I wanted to get this part right. In the end, the real culprit was not have a `json()` function call around the last step in the Power Automate flow - once I did that, the Github side made a lot more sense!

### Job 1 - Handling high-level checks

For the release, I need to check the following things:

- commit hash in email matches commit hash of git tag
- gpg fingerprint is a good signature and matches the one in the email
- sha512 sum matches
- files in the Apache release match the files in the git repo at that hash and there are no unexpected files in the release

I used both ChatGPT and Github Copilot to create the Python script [found here](https://github.com/sorrell/apache_age_release_verification/blob/main/compare.py). I really started to find myself in a different sort of development flow than I had ever experienced before, and it was GOOD! I felt like I was guiding the flow of code instead of getting mired in implementation and syntax.

I would start with ChatGPT and then round out with Copilot. Here was my first prompt to ChatGPT:

> Write a python script that download the gzip file at this link and then decompresses the gzip: https://dist.apache.org/repos/dist/dev/age/PG12/1.3.0.rc0/apache-age-1.3.0-src.tar.gz

This gave me a script that used the `request` library, which felt too bulky. So I followed up with:

> Rewrite the script for downloading a gzip file to use subprocesses and curl

That gave me the base of what I needed. So then pasting that into VSCode, I would simply start to type the next comment, and it would complete code for me. For example, I would start to type "Download" and it would realize I wanted to download the `sha512` and signature. In the image below, ChatGPT gave me lines 13, 14, 19, 20 and Copilot figured out 15, 16, 22-28.

<img src="https://sorrell.github.io/images/202304/compare_ai.png" width="75%">

ChatGPT helped create the Github Actions file as well and when I asked how to iterate on invoking the actions, it nicely suggested to use the `workflow_dispatch` so that I could invoke via the website. The job in the workflow does a few more things:

- This job builds a Docker container (using the Dockerfile in the repo) with the `compare.py` script copied into. 
- The `compare.py` script uses the OpenAI Python binding to call the ChatGPT with the signature response to parse it determine if it is good.
- The pipeline keeps the OpenAI API Key in the repository secrets and injects it into the Docker container as an environment variable
- ChatGPT again saved the day when I couldn't figure out how to actually run a container I had just built - it told me about `load: true`

Here is [some output](https://github.com/sorrell/apache_age_release_verification/actions/runs/4774755320/jobs/8488637226) from a successful action triggered on yesterday's email.

<img src="https://sorrell.github.io/images/202304/gha-job1.png" width="75%">


### Job 2 - Smoketest

This is the second job in the Action I built. As you can see from the [workflow file](https://github.com/sorrell/apache_age_release_verification/blob/main/.github/workflows/trigger_from_power_automate.yml), this job also uses elements from the Power Automate payload. 

This one I had to actually use my own brain to figure out. I needed to clone the Apache AGE repo to build and run the Dockerfile/container. However, I needed to do a bunch of custom things once that container was stood up that the smoketest handles.

I ended up creating [a step in the job](https://github.com/sorrell/apache_age_release_verification/blob/main/.github/workflows/trigger_from_power_automate.yml#L79) where I download a gist of commands to run in the container, and then mount the directory to copy in the file.

You can see [that output here.](https://github.com/sorrell/apache_age_release_verification/actions/runs/4774755320/jobs/8488637263)

### Final Thoughts

There is a lot more I'd like to do to improve this. Emailing or texting myself the results of the tests would be great, as I need to check the Actions manually to see if all the tests passed. There are also a few more manual steps in verifying a release that I'd like to handle. There are also some clunky steps in the workflow and some brittle code (tags, naming) that could be fortified.

But overall, it felt very motivating to work alongside ChatGPT. It was like having a smart person to bounce ideas off of and it kept me engaged and working instead of feeling lost in a see of bad Google searches.

More importantly, it felt like I needed to alter my thinking. Instead of consistently thinking "how should I do <x>," I found myself tossing the initial volley to ChatGPT, who sometimes (ok, often) had better ideas. And having Copilot handle lots of boilerplate was liberating. I enjoyed this project and look forward to the next one with my new teammates.