---
title: Taking back ownership of my email
date: "2024-11-24"
description: "Taking back control of my email by moving to Thunderbird and Migadu"
---

Email is an amazing standard, but it has been almost completely taken over by big tech (as so many other corners of the internet). But because it is an open standard, you can move elsewhere without forcing all your friends to use a different app, as is the case with messaging apps. Needless to say, I _love_ standards.

But even when I had the option, I still used the email inbox "given" to me by Apple and Google. Not anymore.

## A new email client

The first thing that I wanted to change was my email client. I've heard great things about [Thunderbird](https://www.thunderbird.net/en-US/) – while Mozilla has been receiving its fair share of criticism and Thunderbird _is_ under the Mozilla foundation, it's run independently and, inlike Firefox, it's funded almost entirely by donations. And of course, it's [fully open source](https://github.com/thunderbird).

Turns out, Thunderbird is truly amazing with its infinite number of shortcuts and clean interface. What pleasantly surprised me is that it's not just an email client, but also a calendar, contacts, TODOs, and chat client. If that doesn't convince you, you can disable any of these features completely. For me, the calendar was actually a nice surprise, and it turned out to be quite convenient over having an additional calendar app on my Mac.

I've been using Thunderbird for a few weeks now, and I don't see myself going back to the default Mail.app I used before.

Thunderbird is now [available on Android](https://www.thunderbird.net/en-US/mobile/) as well. The team should start working on the iOS version sometime next year, so fingers crossed. For now, I'm sticking with the default Mail.app on my iOS devices - but I don't check my email that much on them anyway, so it doesn't bother me that much. But if you know of a good, ideally open source, iOS email client, let me know.

## Email provider

Apart from moving away from big tech, I also wanted to start using my own email domain. Mainly because this will allow me to be flexible with my email provider in the future - I'll never have to change my login information, my contacts won't have to get used to a new email domain, etc.

I already have a domain for this personal website, so what it had come to down was which email provider to use? I did consider self-hosting, but I also wanted my setup to be as low-maintenance as possible, so that was a no-go.

### Proton

It was hard to miss [Proton Mail](https://proton.me/mail). For many, it's _the_ alternative to providers like Gmail that is very privacy-focused. I did give Proton Mail a try – setting my own domain was straightforward and in a couple of minutes, I had everything working when using their web UI.

The first point of friction was when I wanted to connect my Proton email with Thunderbird. While possible, you have to install [Proton Bridge](https://proton.me/mail/bridge) that you have to keep continually running. Ok, not great, but I could live with that.

But there is no Proton Bridge for iOS (not that surprising), so I'd be forced to use the Proton mail client on my iOS devices. One of the reasons I was doing this whole migration was to become more flexible in my tech choices – but now instead of locked into Apple, I'd be locked into Proton.

Of course, Proton doesn't impose these restrictions for nothing – it's all in the name of privacy, primarily so they can encrypt emails end-to-end. But the big caveat there is that the emails are only end-to-end encrypted when _both_ sides have it set up – be it with Proton or with any other provider supporting [OpenPGP](https://www.openpgp.org/).

Given my email usage, that would be almost no-one – so, I'd be getting vendor lock-in for almost no privacy gain.

### Migadu

I then looked for alternatives until I found [Migadu](https://www.migadu.com/). Migadu is a no-frills, small, independent email provider based in Switzerland that does one thing and does it well.

The interface is simple and gets the job done, even if it's not the prettiest thing you'll ever see, the pricing is tiered based on your usage, and they're very open about the [pros and cons of their service](https://www.migadu.com/procon/).

I'd like to highlight these two blog posts that supported me in the decision of moving to Migadu:
- [Review of Reputable, Functional, and Secure Email Service](https://changelog.complete.org/archives/10711-review-of-reputable-functional-and-secure-email-service) – comparison of some of of the popular providers
- [Migadu > ProtonMail](https://timharek.no/blog/migrate-pm-migadu) – a quick post about migration from ProtonMail with some reasons behind the _why_ that are very similar to mine.

## Summary

With Thunderbird on macOS and Migadu as my email provider, I feel back in the control seat of my email. If either Thunderbird or Migadu doesn't meet my needs anymore, it should be much easier to switch to another service - and no one who interacts with me via email will notice.

Do you have any feedback or want to get in touch? You can drop me an email at <a href="mailto:mail@marekfort.me">mail@marekfort.me</a> 😉
