# Have A Break!

## URL: https://tryhackme.com/room/haveabreak

This is an OSINT room. Usually, these suck on THM because people dont think far enough into the future: sites get scrubbed or deleted, data falls off and gets stale. Eventually, some rooms cant be solved...so you go looking for a sketchy Medium.com walkthrough...

## Description:

Disclaimer: This challenge is inspired by a real cargo theft that occurred in March 2026, in which a shipment of KitKat products was stolen in transit between Italy and Poland. All companies, agencies, individuals, documents, and investigative findings presented in this challenge are entirely fictional. No real employees, law enforcement personnel, or organisations are implicated. The real theft remains under investigation by the relevant authorities.

Background:

On 26 March 2026, a refrigerated truck carrying over 400,000 units of KitKat product vanished somewhere between Central Italy and Poland. Nestlé confirmed the theft two days later. The vehicle has not been found.

The European Cargo Threat Assessment (ECTA) does not believe this was opportunistic. A shipment of this size, on a contracted route, does not disappear without someone helping it along.

An anonymous tip reached a journalist the following evening. ECTA obtained it under judicial authority. That is where your investigation begins.

Your Assignment
You are a CZ Node investigator on Project HAVEABREAK. Your goal is to identify the culprit behind the heist by using the following files:

File |	Description
ecta_memo.pdf |	Your briefing. Start here.
exhibit_a.eml |	Exhibit A — referenced in the memo
exhibit_b.jpg |	Exhibit B — referenced in the memo
transeuro_data/employees.csv |	Subpoenaed from TransEuro Logistics IT
transeuro_data/access_log.csv |	Subpoenaed from TransEuro Logistics IT
transeuro_data/comms_export.txt |	Subpoenaed from TransEuro Logistics IT

You can download the files by clicking on the button in the room.

## Gather the Intel.

*** Start by reading the PDF brief!!!! You will save yourself headaches. ***

### Question 1: Which VPN service was used to send the anonymous email from the .eml file?

I use the Microsoft Message Header Analyzer, https://mha.azurewebsites.net/ because it cleans up the noise very well.

Feed it the headers from the .eml (exhibit a) and find the sender's IP (it was a very short connection if that helps). Then run this through your favorite IP Info site (I use https://ip.info/<IP address>). This should show you that the IP belongs to a VPN service. If you use my favorite site, you will have to log in to get the name (make a free account, it's worth it.) If you use another site...well...you'll figure it out. I believe in you!

### Question 2: What is the full street address of the petrol station where the missing vehicle was last seen?

This is where the PNG file (exhibit b) comes in (THE PICTURE IS ALMOST A RED HERRING. It gives you some info that helps tangentally, but not enough to do anything with. You can be successful wthout it.) And to be honest, this one kicked my butt for 4 hours! I tried everything from reverse image searches, exif data, the official ORLEN station map, heck I even put my friends Gemini, Grok, and Claude on it (they hallucinated BAD!)

Anyway, I didn't get this answer until I found the answer to Question 6, so skip it for now. 

### Quesiton 3: At what time did the suspicious action take place in the route planning system on March 25th, 2026?

Analyze the access_log.csv in the "transeuro_data" folder. Look for events on March 25th, the answer should stand out to you pretty well. Some may say it will "export" itself ;)

### Question 4: What is the employee ID of the person who sent the anonymous email?

Again, Look at the log file. It happens the same day, soon after the event from Q3

### Question 5: What is the employee ID of the employee responsible for leaking the shipment details?

You'll find that answer when you get the answer for Q3

### Question 6: What is the full name of the culprit?

This one had me mad, losing my mind. Looking at the comms log, I found an email address of an outside user trying to access files. I did the standard google searching for the email address, played with some OSINT tools, fed my friends again. We all came up short.

Then I remembereed ghunt. https://github.com/mxrch/ghunt?tab=readme-ov-file

Running `ghunt email <address>` quickly gave me the way forward. I found a google maps review (remmeber question 2?). Pulling up the link, the review had the full name of the theif AND the link for the correct ORLEN station with the address.
