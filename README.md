# Oooshiny Google Groups Migration

Migrating from Google Groups to a self hosted, member-maintained platform.

Required features:

- Support for a traditional email-based list-serve
- Member’s can invite other members
- Avoiding any third-party solutions that eventually become disrupted and we have to move again.

Candidate:

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

See main [#1 Discourse Installation Progress](https://github.com/OooShiny-Community/migration/issues/1) for more details and other [Issues](https://github.com/OooShiny-Community/migration/issues) to track progress.
