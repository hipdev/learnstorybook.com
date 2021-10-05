---
title: 'How to actually test UIs'
tocTitle: 'Introduction'
description: 'Testing techniques used by leading engineering teams'
---

<div class="aside">This guide is made for <b>professional developers</b> learning how to test UI components. Intermediate experience in JavaScript and React is recommended. You should also know Storybook basics, such as writing a story and editing config files (<a href="/intro-to-storybook">Intro to Storybook</a> covers all the basics).
</div>

<br/>

Testing UIs is awkward. Users expect frequent releases packed with features. But every new feature introduces more UI and new states that you then have to test. Every testing tool promises “easy, not flaky, fast”, but has trade-offs in the fine print.

How do leading front-end teams keep up? What's their testing strategy, and what methods do they use? I researched ten teams from the Storybook community to learn what works — Twilio, Adobe, Peloton, Shopify and more.

This post highlights UI testing techniques used by scaled engineering teams. That way, you can create a pragmatic testing strategy that balances coverage, setup, and maintenance. Along the way, we'll point out pitfalls to avoid.

## What are we testing?

All major JavaScript frameworks are component-driven. That means UIs are built from the “bottom-up”, starting with atomic components and progressively composed into pages.

Remember, every piece of UI is now a component. Yup, that includes pages. The only difference between a page and a button is how they consume data.

Therefore, testing UI is now synonymous with testing components.

![Component hierarchy: atomic, compositions, Pages and Apps](/ui-testing-handbook/component-testing.gif)

When it comes to components, the distinction between different testing methods can be blurry. Instead of focusing on terminology, let’s consider what characteristics of UIs warrant testing.

- **Visual:** does a component render correctly given a set of props or state?
- **Composition:** do multiple components work together?
- **Interaction:** are events handled as intended?
- **Accessibility:** is the UI accessible?
- **User flows:** do complex interactions across various components work?

## Where should you focus?

A comprehensive UI testing strategy balances effort and value. But there are so many ways to test that it can be overwhelming to figure out what’s right for any given situation. That’s why many teams evaluate different testing techniques using the criteria below.

- 💰 **Maintenance cost:** time and effort required to write and maintain the tests.
- ⏱️ **Iteration speed:** how fast is the feedback loop between making a change and seeing the result.
- 🖼 **Realistic environment:** where the tests are executed—in a real browser or a simulated environment like JSDOM.
- 🔍 **Isolate failures:** a test fails, how quickly can you identify the source of the failure.
- 🤒 **Test Flake:** false positives/negatives defeat the purpose of testing.

For example, end-to-end testing simulates “real” user flows but isn’t practical to apply everywhere. The key advantage of testing in a web browser is also a disadvantage. Tests take longer to run, and there are more points of failure (flake!).

Now that we’ve covered the UI characteristics to test and the criteria to evaluate each testing method, let’s see how teams design their test strategy.

> "Testing gives me full confidence for automated dependency updates. If tests pass, we merge them in."
> — Simon Taggart, Principal Engineer at Twilio

## Visual testing: does this look right?

Modern interfaces have countless variations. The more you have, the tougher it is to confirm that they all render correctly in users' devices and browsers.

In the past, you’d have to spin up the app, navigate to a page, and do all kinds of contortions to get the UI into the right state.
