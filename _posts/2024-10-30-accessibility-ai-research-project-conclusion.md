---
title: LLMs and Accessibility Testing
date: 2024-10-31 08:00:00 Z
categories:
- Artificial Intelligence
tags:
- accessibility
- Artificial Intelligence
summary: This blog reports on our R&D project exploring the potential of LLMs
  to enhance accessibility testing.
author: crobb
image: "/uploads/llms%20accessibility.png"
---

## Summary of our Accessibility AI Research Project
This project began with the idea of using an LLM to aid user testing - would it be possible for it to navigate a webpage accurately and determine how easy it is to find specific information?

## How we did it
This research could be broken down into two distinct parts:

- **Part One - The Live User Test**: Create a series of tree tests that we can have real users complete for analysis
- **Part Two - The LLM Test**: Take the same tree tests and have an LLM attempt to navigate them.

## Something to know - Tree Testing
These tests revolve around tree testing, a way of determining the findability of something in an organised hierarchy - i.e. a tree. Whenever you are trying to find something in a menu, such as the "contact us" section of a website, it should be under relevant headers to make it straightforward to locate. 

Tree testing differs from traditional user testing, in that it is not performed on the website itself, rather a simplified textual structure of the menu is created and given to participants who are to find the correct place to find a given topic. Following this the results are analysed - could the user find the correct location? Did they have to backtrack a lot? Was there a lot of thinking required to understand where the correct location would be?

## The tools we used
So looking at the first part of the experiment, we needed a tool that could support running a tree test across numerous users easily. We considered whether we wanted to run a moderated (with supervision) test, or and unmoderated (without supervision) test, and decided that due to the geographical constraints of the participants that we would run an unmoderated test. With this in mind, we found Kort, a UX Research tool that is open source and could be modified for our needs. We would run this in AWS for the duration of the User Test.

For the LLM test, we went with OpenAI's GPT models, and by feeding the tree tests into the models asking for it to select the most relevant nodes for the prompt, we could have it effectively navigate a tree test.

## The process - The POC
First of all, we had to take some time to learn how we could work with the AI - most of us had little to no experience in AI so we created a POC, aiming to gain an understanding. This was a cashier application in which we created an agent, gave it some products to work with, and would request it to give us the total cost of the products we asked it for. Within this, we began to get an understanding of how the LLM would work, how we could test it, and how reliable it was. Could it handle an uncertain customer changing their order mid sentence? Would it stick to it's prompt if we asked it something unrelated? Overall the answer was yes to all these questions, and through this the team gained a lot of insight into dealing with an LLM agent.

## The Process - Identifying metrics for the tests
Next up, we needed to identify the metrics that we wanted to use for both the User Test and the LLM Test, things that were easily measurable and that we could compare. We settled on three key metrics that we wanted to use

**The Success Rate**: Quite simply, was the answer correct?  
**The Directness**: Did the user take the optimal path to the answer? Or did they have to navigate through unrelated nodes before settling on the answer?  
**The Time Taken**: Was it faster using the LLM or were humans quicker?

We also wanted to gather data around how the user managed with the questions:

**Understanding the Question**: A yes or no as to whether the user understood the question.
**Difficulty**: A scale of 1 to 5, on how difficult it was to find the answer within the tree. 

## The Process - Getting Kort ready
Next up, we looked towards the Live User Test. And for that, we needed to test Kort to understand it's usability, and modify it to extract the above metrics. We added timestamps to each event generated by the user, we recorded events for each node a user navigated to, and we did a general tidy up and bug fixing throughout the application.

We ran into some additional bugs along the way - getting Kort using HTTPS, we found that it throttled requests and made changes to that (maximum requests accross all users was 120 in ten seconds, and we were asking almost that many people to complete the test). Simple things such as deleting a test were failing, which hampered our automation tests on the suite. However one of the big issues we discovered under load testing was that it had a race condition, where if concurrent users submitted an answer at the same time as each other, the database state would change and the saving of the response would fail.

Once we had all of these issues resolved, we tested it heavily, running through hundreds of iterations of tests in short amounts of time to ensure that we would not lose any data gathered from the User Test.

## The Process - The LLM Test Set Up
We needed to prove that we could run a LLM against a tree, so we created a project that would create an agent, who would be given an objective and a tree of nodes to navigate. The agent could either NAVIGATE (to find it's way through the tree), SUBMIT (when it found the node it thought was the correct answer), or GIVE_UP (when it decided that there was no node that could reasonably answer the objective). This project could be configured by the OpenAI model that we wanted to use, and the temperature that we wanted to run against it, and would take a JSON file specifying the trees we wanted to test. This JSON was made to be the same format that KORT would use for ease of feeding the finalised trees in. Every time the LLM would make a decision on a tree, we would log it with a timestamp to get the same metrics that we were taking within Kort. On completion of each test, the LLM provides whether it understood the question, and how difficult it found it. 

## The User Test
Once we were happy with Kort, managed to get it deployed and tested to AWS, we were ready to run our live test. We deployed the live test to AWS on a Monday morning, ran a script to create our Google Surveys (which would contain the links to Kort and ask the qualatitive questions after each test) and sent out our emails to the participants accross the company that we had identified. We ran the test from Monday to Friday to avoid any costs for running it over the weekend, and got a good number of responses - over 60 on some of the trees! Some of the trees were lower as not all of the volunteers finished every single one, but we still got some excellent data that we could work from.

## The LLM Test
Now that we had the data from the User Test, we could look to run the LLM Test. Our plan was straightforward, we would feed the same trees and questions that had been used in the user test to the LLM project that we created earlier, and configure it across models and temperatures. We used gpt-3.5-turbo and gpt-4-turbo, and had iterations with temperature set to 0, 0.5 and 1. The higher the temperature used, the more unpredictable and diverse the output can be, and the lower the more predictable and conservative the output.

## Analysis
Now that we had our two sets of data, we could now clean the data to get it into a state that we could work with, and then compare the LLM and Human outputs.

We used Google Colab notebooks to do this, which comes with Googles AI Gemini to help with creating the code blocks, which seemed appropriate for this project!

From here we manipulated the data to make it easy to understand, and be able to display the data in clear, easy to read charts.

## The Insights
Lets take a look at some comparisons between the human experiment and the LLM.

![The percentage of correct answers between the human test and the llm test]({{site.github.url }}/crobb/assets/PercentageCorrectHumanVsLLM.png)

Ah okay, the LLM does not seem to have managed to do as well as the humans overall has it? Lets take a look at our directness metric (the most optimal route to the right answer) in the correct answers.

![The percentage of correct answers with Directness at 100%]({{site.github.url }}/crobb/assets/PercentageOfDirectness100andCorrectHumanVsLLM.png)

Okay, so we can see that if it is right, it is pretty good at finding the most optimal route. How does it stack up in terms of average time taken? If the LLM was more correct, is it at least faster?

![Average time taken LLM vs Humans]({{site.github.url }}/crobb/assets/TimeHumanVsLLM.png)

Okay, we can see that the LLM is significantly faster than humans. What about the difference in GPT models?

![Percentage correct answers by LLM model]({{site.github.url }}/crobb/assets/PercentageCorrectAnswersByLLMModel.png)

So we can see that GPT4Turbo, is generally as good if not better, than 3.5.

## Some considerations to note
We ran into some issues that are important to consider.

**The Turnover in the team**
As is the nature of being a consultancy, new clients came in, and members of the team rolled off the project to go work on them. New members joined the team, but we would lose some time as they did their discovery phase to upskill for the project.

**Kort wasn't quite where we wanted it**
Kort is a fantastic tool, and incredible for being open source, however we were limited by the number of participants available to us. If something went wrong then we would have effectively burned those participants as they had already been exposed to the trees. This meant that things such as the concurrent user bug, which may have not become an issue, had to be resolved to ensure we didn't lose or corrupt our dataset.

**Prompt engineering didn't come to pass**
Ideally, we would have wanted to engineer the prompt that was being provided to the LLM, to ensure that we could make it the most accurate it could be. Unfortunately the deadline for the close of the project, coupled with the last remaining members all needing to upskill for client work, we didn't manage to achieve this. This was likely reflected with the LLM choosing to give up in a number of tests. Fpr example, the LLM was tasked to find the most relevant place to find a rucksack, would navigate to bags, and then give up as there wasn't a specific enough node under bags for a rucksack.

![LLM Give Up / Failures Vs Human Failures]({{site.github.url }}/crobb/assets/FailuresHumansVLLM.png)

## Despite the considerations...
We learned a lot. The entire team was able to work with the OpenAI GPT models and learn from them - how to test them, how to create an agent for a specific role, to step out of our comfort zone and work with exciting new technology. We ran a successful User Test, across the company, gathering valuable data that could potentially be used in future to expand on this study. We began experimenting with dataframes, and how analyse the data that we had extracted from all of this.