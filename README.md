# OooShiny Mailinglist Migration Plan

This is a living document outlining the inevitable exodus from Google Groups to a self hosted, member-maintained platform.

Google Groups has been used to host Oooshiny since its inception, and while the platform "works", we need to understand that it may not be around forever. Consider this a conceptual community backup plan. However, if an option comes up that allows a premature migration, it should be considered.


## Required features

- Support for a traditional email-based list-serve
- Avoiding any third-party solutions that cause a dependency enmeshment that could be disrupted and we have to move again.
- Ability to send 10k emails a day (1000 people on the list, 10 emails a day), with potential of scaling.


## Current Candidates

### GNU Mailman

[Mailman](https://list.org/)

Mailman is free software for managing electronic mail discussion and e-newsletter lists, first released in 1999. It's the defacto "listserve" software. It is a mature, and maintained FOSS project.

### Sendgrid (Email Service)

[Sendgrid](https://sendgrid.com/)

SendGrid (also known as Twilio SendGrid) is a SaaS for transactional email, aquired by Twilio in 2019. Costs are varying, ranging from 35 to 100 USD per month.

### Funding requirements

Virtual private server with custom domain and Email service cost breakdown

- $4 to $10 a month - Basic VPS "Droplet" from [Digital Ocean](https://www.digitalocean.com/pricing/droplets) starts at $4.
- $20 to $100 per month - Sendgrid Mail Service 100k to 1.5mm+ emails monthly. Used for user-auth, tokens, updates, digests, etc.

Total costs per month:

$45 to $120 per month total, $ 540 to $ 1500 per year.


## Past (depricated) Candidates

### Discourse

Reason for Deprication:

Discourse is web-first. A solution needs to be email-first.

Description:

A self-hosted, private instance of Discourse on a VPS with a mail server

Features:

- Members can create and manage their own new-member invites
- Optional Web front end for searching, browsing, linking
- Passwordless login option (emails one-time login link)
- Extensive admin/mod tools — allows us to expand the mod team
- Extensive API, allowing this group to be used creatively (enables community bots, forum-centric projects, etc)
- Categories, Tags

Migration Strategy (WIP):

- Import old Google Group -- Discourse has a method for this
- Get a few volunteers to jump on a test server and poke around
- Solicit feedback from the group on the tech stack, and setup
- Post an official announcement on this group for the upcoming move and post the new forum info
- "Stage" members on Discourse so they become active when logged in
- Encourage members to join as easily as possible
- Set Google group to read-only
