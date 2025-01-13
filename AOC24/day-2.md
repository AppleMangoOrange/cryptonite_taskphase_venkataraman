# Day 2

- The website in the challenge links to Elastic SIEM which is hosted. The username and password are: `elastic`.
- The suspicious activity took place between 09:00-09:30 am on December 1st, so this is the time window.
- After adding the necessary fields, it is clear that someone successfully ran some powershell commands.
- Checking for authentication events for the past few days reveals that a certain IP tried brute-force attacks untill succeeding and logging in with the IP `10.0.255.1`.
- The challenge reveals that this log in was from 'Glitch' to fix outdated Windows credentials, thus being a false-positive (in a sense).