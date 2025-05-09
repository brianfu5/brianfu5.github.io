---
title: Building a Bot to Help Me Buy F1 Tickets
categories:
- Technology
excerpt: |
    A guide on how I wrote a Python script to automate buying Formula 1 tickets.
---

<br/>

{% include figure.html image="https://brianfu.net/assets/img/f1_bot/f1_shanghai_poster.jpeg" width="500" height="800" %}

### Introduction

Ever since I became a Formula 1 fan over a year ago, I’ve dreamed about watching a race in person. I wanted to feel the live atmosphere, experiencing all the speed, noise, drama, and thrill that F1 delivers. Now that I am currently studying abroad in Korea, I saw F1 Shanghai as the golden opportunity; it’s a relatively cheaper race, it’s close to Korea, and I get to go to Shanghai! This was a great idea, but there was only one problem. 

Ticketing for this event started in December of last year on the Chinese booking platform [Juss Sports 久事体育](https://ztwen.jussyun.com/pc/list){:target="_blank"}, and they sold out instantly. The only other places I could find tickets were on American ticketing websites with such inflated prices that going was just not worth it.

I was just about to give up on my Shanghai F1 dream when a friend jokingly said to me, “What if you built a bot that can do this for you?” I initially laughed this idea off; it was just a random quip, and I hadn’t done anything like it before. But when I got home that night and gave it some more thought, the more confident I became that maybe it wouldn’t be that hard. I realized that with some effort and learning, this was totally doable, so the next afternoon I sat down and got to work.

### Understanding the Ticketing System

Juss Sports (and actually now most of China in general) uses a different approach for ticket booking than America. To buy a ticket, you need an ID for each seat you buy (for me, my American passport). Then, on the day of the event, you just need to provide that ID to gain entry, eliminating the need for any mobile or physical ticket. This “real-name verification service” prevents scalpers from buying all the tickets and reselling them at a higher price since all the tickets need to be tied to an ID, and you cannot transfer tickets from one ID to another. 

With this process, if someone decides to refund their ticket, it would automatically become available on the ticketing website again for someone else to buy. I remember I was refreshing this website manually every couple of minutes to see if I could get lucky and snipe a ticket, but it never happened. It became clear to me that a computer program would be so much more efficient. So I built a Python bot to continuously check for available tickets and complete most of the purchasing process. The following section will cover the technical aspects of how it works.

### How It Works

Here is a brief summary of the tools and libraries I used:
 - Programming Language: Python
 - Libraries: Selenium, BeautifulSoup, Requests, Playsound
 - Dependencies: ChromeDriver (for Selenium)

Writing the code for the bot happened in two distinct phases. The first was writing the script to constantly check for ticket availability, and the second was to have it automatically open up the ticketing page and navigate to the seating selection page and payment. 

**1. Checking for ticket availability (`main.py`)**

The majority of the logic is located in the `check_tickets()` function. To get started, I had to first check how the site gets the data for if there is an open ticket or not. If the website shows that there is no ticket available to the user, it must get that information in some way, so I wanted to figure it out to see how I can scrape that data later. With a little digging, I realized that Juss Sports uses AJAX, meaning the website is rendered client-side in the user’s browser, but it dynamically fetches updated data from the server using JavaScript. This meant that the website probably sent some API requests to a server to get this data, so I opened up the Inspect Network tab on Chrome and looked for the Fetch/XHR, where it showed the outgoing HTTP API requests and the responses.

{% include figure.html image="https://brianfu.net/assets/img/f1_bot/network_tab.png" width="300" height="800" %}

After looking through each one of the responses, I found one that contained data for when a session had an available ticket, meaning that if we can just query this API endpoint, we have our method to check for ticket data.

{% include figure.html image="https://brianfu.net/assets/img/f1_bot/network_response.png" width="400" height="400" %}

So I did some research and found the requests library, which can send requests and get responses from different endpoints. Using the requests library, I wrote code to send about 20 requests per second to the specified endpoint. To avoid setting off any simple bot detection algorithms, I swapped between a pool of 3 user agents in my requests, and all requests were sent uniformly randomly between a time of 0 and 0.1 seconds. This would work well, but there would be times where the request would have connection issues or be stuck on a read, so I also added a retry mechanism that allows up to 3 tries (if it fails 3 times in a row, they probably figured out I’m a bot, and we can just exit the program). 

Each time we query the endpoint and receive a response, we can parse that response and isolate the seat data for the specific session you want. In my case, I wanted the A High Grandstands for all 3 days, so I figured out the sessionId for that, again using the network page, and specifically checked for the seat data for it. This would ignore all other available tickets, such as the occasionally available Friday only passes, and only go for the specific session I wanted. When the bot finds an available ticket, an alarm sound is played to alert the user, and the function to buy the tickets is also called, which is written next.

**2. Automate filling out the forms for the ticket (`buyTicket.py`)**

This step was actually rather easy once I figured out how everything worked, but initially when I started trying to fill in the forms automatically, I had trouble because I had not used Selenium WebDriver or worked with XPath’s before. XPath’s are actually really simple; essentially they are just syntax for navigating nodes in an XML document, or in our case, an HTML document. Again, Chrome makes this really easy to find; when you right-click on an element in HTML using the Inspect tab, you have an option to directly copy that element’s XPath.

Then, the code is actually really simple. `clickButton()` takes an input of an XPath and clicks on that element using WebDriver. Similarly, `fillField()` fills in a field with characters given the XPath of the field element. The ‘login()’ function just presses buttons and fills in the username and password in the correct order, and we can log in pretty much instantly. The actual `buyTicket()` function just completes the whole code, first launching the Chrome browser and loading the event site, calling the `login()` function, and finally calling `clickButtons()` in its own specific order to quickly reach the seat selection page. 

While the logic behind this code is straightforward, I spent a lot of time experimenting to have everything working smoothly. The main challenge came from learning how Selenium works and how to use XPath in my code. For example, I was initially trying to use RegEx to click on buttons that matched element names, but this was super buggy, so I looked around some more and found XPath which I started using instead. There were also some other bugs that appeared throughout, which I had to fix, such as WebDriver opening an infinite number of webpages or buttons not getting clicked. The latter bug was actually an interesting problem. Because the bot executes actions instantly, it would try to click on buttons before AJAX had finished loading the dynamic data, so I had to add a one second sleep for the HTML to load and the buttons to become clickable. One other small complication was that because all the tickets were sold out for this event, I had to test it on another event on Juss Sports, which had open tickets, and then transfer the testing results to the F1 page. But overall, the bugs were not too difficult to figure out, and once I actually understood how everything worked, it I was able to finish this part quickly.

### Results

I finished the program on the evening of March 8th and left my computer running with the program overnight, with the intent of the alarm waking me up if there was a ticket found. However, nothing happened, and this was confirmed when I checked my computer the following morning. March 9th was a Sunday, so I was in my room on and off, but I would run the program whenever there was time. Each time, I would wait for the alarm to ring, but there was still nothing. At this point, I had started to lose hope; with the race being only 10 days away, it seemed that people were already committed to going to the event. Nevertheless, I still let the program run throughout the night, hoping for a miracle. 

Monday morning came, and once again, I was out of luck. I shut down the program for a bit while I was out, but when I returned around noon, I decided to give it another shot. This time, just within five minutes, I heard the alarm sound ringing. I immediately ran to my laptop, thinking that it was a false alarm or maybe the program had bugged, but there it was in the terminal output that a ticket might have been found. I checked the website, and to my surprise, there were multiple tickets available. I quickly selected a single seat, thinking it had the highest probability of still being available, and checked out. It wasn’t until I completed the payment and got to the final confirmation screen that it hit me–I was going to Shanghai. 

### Disclaimers and Ethics

Creating a ticket bot, admittedly, is kind of a scummy move; it is unfair to genuine ticket buyers and ruins the integrity of the booking websites (it’s also probably definitely against the terms of service as well, but I didn’t read them :/). However, in my desperation, I went along with it. I tried to keep things somewhat fair by only checking ticket availability around 20 times a second, and I rationalized with myself that I would only be taking a ticket from one person. Not to try and justify anything, but in my initial search for F1 tickets, I found Chinese people who you could pay around 300 RMB to buy you tickets as well with your ID. I’m not sure what systems they use, but given the advanced nature of ticket botting, I wouldn’t be surprised if they checked for tickets on the scale of milliseconds or even faster. Compared to these advanced bots, this felt more like an ethical compromise. Nonetheless, I don’t particularly encourage anyone to use my script or to write their own ticketing program, but if you do, please keep in mind the ethics of it :). 

### Future Improvements

There are some improvements that I hope I can work on in the future when I have the time.

**Captcha Handling**  
My original plan was to have the script do everything for booking, including payment. However, I quickly ran into an issue: captchas. Every time a user clicked the submit button to go to the payment screen, they had to solve a slider captcha. A potential solution is to use pyautogui to try to mimic a human sliding of the bar (with acceleration and movement and everything else).

**Automatic Seat Selection**  
From my initial analysis on the seating selection, it seems to be rendered through Mapbox GL JS on an HTML \<canvas>. That made it hard to just select seats based on the dynamically loaded HTML and JavaScript in the code. My idea is to look at the pixel colors on the screen and click into a seat that was open (open seats were blue dots, compared to gray dots for reserved seats). Libraries like ImageGrab and pyautogui are able to take a screenshot, analyze the pixels on the screen, and click on those pixels. I was actually starting to implement this, but I decided that I wanted to be the one that picked my own seats if multiple were available, so I didn’t go through with it. But in the future, I’d like to have a toggle or a boolean where you could have the bot automatically select seats for you if you don’t care and just want a ticket. 

**Improving Code Readability and Configurability**  
One last improvement I want to make is to make the code more readable and also more easily configurable. In its current state, you need some understanding of code to be able to read and modify it, such as knowing how to find the XPath of an element in HTML. However, this can be trickier for people who are not too familiar with code, so I hope to make this more accessible to them. Also, the bot can only buy one ticket at a time right now, so I think a configuration where you can choose how many tickets to buy would be pleasant.

### Learnings and Conclusion

Working on this project was genuinely so fun and probably the most practical that studying computer science has been for me. What started out as a frustrated whim about buying F1 tickets turned into a technical challenge, where solving it meant that I would get the F1 tickets I so desired.

This also reminded me why I loved being a CS major–I have the power to build solutions to real problems if I just put my mind to it. I also gained valuable experience in web scraping, browser automation, and debugging. 

Also, I know I stopped posting, but I will _actually_ try this now to post semi-regularly. My next blog post will be about the actual F1 race and my time in Shanghai, as well as an aspect of Chinese culture among young people that I find so interesting, so stay tuned for that!

### Code

You can find the full code for this project on my [GitHub](https://github.com/brianfu5/Jussyun-Ticket-Bot){:target="_blank"}, where if you want, you can fork it and configure it for yourself. There are also instructions there on how to set up and run the program. 
